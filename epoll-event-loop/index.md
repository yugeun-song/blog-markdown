# Building an Event Loop with epoll — Recreating libuv Internals

Event-driven I/O is the backbone of modern servers, from Node.js to Nginx. Under the hood, Linux provides `epoll` — a scalable I/O event notification mechanism that outperforms both `select()` and `poll()` for large numbers of file descriptors.

In this post, we build a minimal event loop from scratch using the epoll API, following the same principles that power libuv.

## Prerequisites

You need a Linux system with kernel 2.6+ and a C compiler. We use `epoll_create1`, `epoll_ctl`, and `epoll_wait` from `<sys/epoll.h>`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <errno.h>
#include <sys/epoll.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <fcntl.h>

#define MAX_EVENTS 64
#define LISTEN_BACKLOG 128
#define BUF_SIZE 4096
```

## Setting Up a Non-blocking Listener

The first step is creating a TCP socket in non-blocking mode. This is critical — blocking sockets would stall the entire event loop.

```c
static int make_nonblocking(int fd) {
    int flags = fcntl(fd, F_GETFL, 0);
    if (flags == -1) return -1;
    return fcntl(fd, F_SETFL, flags | O_NONBLOCK);
}

static int create_listener(uint16_t port) {
    int fd = socket(AF_INET, SOCK_STREAM, 0);
    if (fd < 0) {
        perror("socket");
        return -1;
    }

    int opt = 1;
    setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, &opt, sizeof(opt));

    struct sockaddr_in addr = {
        .sin_family = AF_INET,
        .sin_port = htons(port),
        .sin_addr.s_addr = INADDR_ANY,
    };

    if (bind(fd, (struct sockaddr *)&addr, sizeof(addr)) < 0) {
        perror("bind");
        close(fd);
        return -1;
    }

    if (listen(fd, LISTEN_BACKLOG) < 0) {
        perror("listen");
        close(fd);
        return -1;
    }

    make_nonblocking(fd);
    return fd;
}
```

## The epoll Instance

An epoll instance is a kernel object that watches a set of file descriptors. Think of it as a subscription list: you tell the kernel which fds and events you care about, and it notifies you when something happens.

```c
typedef struct {
    int epoll_fd;
    int listen_fd;
    struct epoll_event events[MAX_EVENTS];
} event_loop_t;

static int loop_init(event_loop_t *loop, uint16_t port) {
    loop->epoll_fd = epoll_create1(0);
    if (loop->epoll_fd < 0) {
        perror("epoll_create1");
        return -1;
    }

    loop->listen_fd = create_listener(port);
    if (loop->listen_fd < 0) return -1;

    struct epoll_event ev = {
        .events = EPOLLIN,
        .data.fd = loop->listen_fd,
    };

    if (epoll_ctl(loop->epoll_fd, EPOLL_CTL_ADD, loop->listen_fd, &ev) < 0) {
        perror("epoll_ctl");
        return -1;
    }

    return 0;
}
```

Key details:

- `epoll_create1(0)` creates the epoll instance. The argument is flags (0 for default).
- `EPOLLIN` means we want to be notified when data is available for reading.
- `data.fd` stores the file descriptor so we can identify it later in the event loop.

## Accepting Connections

When the listener socket becomes readable, a new client is waiting. We accept it, set it to non-blocking, and register it with epoll.

```c
static void handle_accept(event_loop_t *loop) {
    for (;;) {
        int client_fd = accept(loop->listen_fd, NULL, NULL);
        if (client_fd < 0) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                break;  /* all pending connections handled */
            }
            perror("accept");
            break;
        }

        make_nonblocking(client_fd);

        struct epoll_event ev = {
            .events = EPOLLIN | EPOLLET,  /* edge-triggered */
            .data.fd = client_fd,
        };

        if (epoll_ctl(loop->epoll_fd, EPOLL_CTL_ADD, client_fd, &ev) < 0) {
            perror("epoll_ctl: client");
            close(client_fd);
        }
    }
}
```

Notice `EPOLLET` — this enables **edge-triggered** mode. Unlike level-triggered (the default), edge-triggered only fires when the state *changes*. This means you must drain all available data in a single callback, or you will miss events.

## Reading Data (Edge-Triggered)

With edge-triggered epoll, we must read in a loop until `EAGAIN`:

```c
static void handle_read(int fd) {
    char buf[BUF_SIZE];

    for (;;) {
        ssize_t n = read(fd, buf, sizeof(buf));

        if (n < 0) {
            if (errno == EAGAIN) break;  /* no more data */
            perror("read");
            close(fd);
            return;
        }

        if (n == 0) {
            /* client disconnected */
            close(fd);
            return;
        }

        /* echo back — a real server would buffer this */
        write(fd, buf, n);
    }
}
```

## The Main Loop

This is the heart of the event loop, structurally identical to what libuv does internally:

```c
static void loop_run(event_loop_t *loop) {
    printf("Event loop running on :%d\n", 8080);

    for (;;) {
        int n = epoll_wait(loop->epoll_fd, loop->events, MAX_EVENTS, -1);
        if (n < 0) {
            if (errno == EINTR) continue;
            perror("epoll_wait");
            break;
        }

        for (int i = 0; i < n; i++) {
            int fd = loop->events[i].data.fd;

            if (fd == loop->listen_fd) {
                handle_accept(loop);
            } else {
                handle_read(fd);
            }
        }
    }
}
```

The `-1` timeout means "block forever until an event arrives". In a real event loop (like libuv), this timeout is calculated from pending timers.

## Putting It Together

```c
int main(void) {
    event_loop_t loop;

    if (loop_init(&loop, 8080) < 0) {
        return EXIT_FAILURE;
    }

    loop_run(&loop);

    close(loop.listen_fd);
    close(loop.epoll_fd);
    return EXIT_SUCCESS;
}
```

Compile and test:

```bash
gcc -O2 -Wall -o evloop evloop.c
./evloop &
echo "hello" | nc localhost 8080
```

## Level-Triggered vs Edge-Triggered

| Mode | Fires when | Must drain? | Use case |
|------|-----------|-------------|----------|
| Level-triggered | fd is ready | No | Simpler logic, default |
| Edge-triggered | state changes | Yes | Higher performance, less syscalls |

libuv uses level-triggered by default for simplicity. Nginx uses edge-triggered for performance. Both are valid — the choice depends on your throughput requirements and tolerance for complexity.

## What libuv Adds on Top

Our loop handles I/O, but libuv's event loop also manages:

- **Timers** — a min-heap of deadlines, used to calculate `epoll_wait` timeout
- **Idle/prepare/check handles** — hooks that run on each loop iteration
- **Thread pool** — for blocking operations like DNS and filesystem I/O
- **Cross-platform abstraction** — epoll on Linux, kqueue on BSD/macOS, IOCP on Windows

The core pattern, however, is the same: wait for events, dispatch callbacks, repeat.

## Summary

Building an event loop from the epoll API gives you a concrete understanding of how Node.js, Nginx, and other event-driven systems work under the hood. The key takeaways:

1. **Non-blocking sockets** are the foundation — without them, the loop stalls.
2. **epoll** is a subscription-based notification system, not a polling mechanism.
3. **Edge-triggered mode** requires draining all data, but reduces syscall overhead.
4. **The loop itself is simple** — the complexity lives in managing timers, thread pools, and cross-platform abstractions.
