# 커널 모듈 작성 입문 — Hello World부터 디바이스 드라이버까지

리눅스 커널 모듈은 실행 중인 커널에 동적으로 로드할 수 있는 코드 조각이다. 커널을 재컴파일하지 않고도 기능을 확장할 수 있어, 드라이버 개발이나 커널 실험에 필수적인 도구다.

이 글에서는 가장 기본적인 모듈 구조부터 시작해 캐릭터 디바이스 드라이버까지 점진적으로 복잡도를 높여간다.

## 개발 환경 준비

커널 모듈을 빌드하려면 현재 실행 중인 커널의 헤더가 필요하다. Arch Linux 기준으로 다음 패키지를 설치한다.

```bash
sudo pacman -S linux-headers base-devel
```

헤더가 정상적으로 설치되었는지 확인하려면 `/lib/modules/$(uname -r)/build` 경로가 존재하는지 체크하면 된다. 이 심볼릭 링크가 커널 소스 트리를 가리키고 있어야 `make`가 올바르게 동작한다.

### Makefile 기본 구조

커널 모듈용 Makefile은 일반적인 유저스페이스 빌드와 다르다. `obj-m` 변수를 통해 빌드할 모듈 오브젝트를 지정한다.

```makefile
obj-m += hello.o

KDIR := /lib/modules/$(shell uname -r)/build

all:
	make -C $(KDIR) M=$(PWD) modules

clean:
	make -C $(KDIR) M=$(PWD) clean
```

`make -C`는 커널 빌드 시스템 디렉토리로 이동한 뒤, `M=` 으로 지정된 외부 모듈 경로에서 빌드를 수행한다.

## Hello World 모듈

가장 단순한 커널 모듈은 로드 시 메시지를 출력하고, 언로드 시 정리 작업을 수행하는 두 개의 함수로 구성된다.

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("devlog");
MODULE_DESCRIPTION("A simple hello world kernel module");

static int __init hello_init(void)
{
    pr_info("hello: module loaded\n");
    return 0;
}

static void __exit hello_exit(void)
{
    pr_info("hello: module unloaded\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

`pr_info`는 `printk(KERN_INFO ...)`의 매크로 래퍼다. 커널 로그 레벨에 따라 `pr_err`, `pr_warn`, `pr_debug` 등을 사용할 수 있다.

### 모듈 로드와 확인

빌드 후 `insmod`로 로드하고 `dmesg`로 커널 로그를 확인한다.

```bash
make
sudo insmod hello.ko
dmesg | tail -1
# [12345.678] hello: module loaded

sudo rmmod hello
dmesg | tail -1
# [12345.789] hello: module unloaded
```

`lsmod` 명령으로 현재 로드된 모듈 목록을 확인할 수 있으며, `modinfo hello.ko`로 모듈 메타데이터를 조회할 수 있다.

## 모듈 파라미터

커널 모듈은 로드 시 파라미터를 받을 수 있다. `module_param` 매크로를 사용하면 `insmod` 또는 `modprobe` 시점에 값을 전달할 수 있다.

```c
static int count = 1;
module_param(count, int, 0644);
MODULE_PARM_DESC(count, "Number of greetings");

static int __init hello_init(void)
{
    for (int i = 0; i < count; i++)
        pr_info("hello: greeting %d\n", i);
    return 0;
}
```

세 번째 인자 `0644`는 `/sys/module/<name>/parameters/` 아래에 생성되는 파일의 퍼미션이다. 런타임에 `echo` 명령으로 파라미터 값을 변경할 수도 있다.

### sysfs 인터페이스

`0644` 퍼미션을 설정하면 `/sys/module/hello/parameters/count` 파일이 읽기/쓰기 가능하게 된다. 이를 통해 모듈을 재로드하지 않고도 동작을 변경할 수 있다.

단, 파라미터 변경 시 콜백이 자동으로 호출되지는 않으므로, 변경을 감지하려면 별도의 메커니즘이 필요하다.

## 캐릭터 디바이스 드라이버

실제로 유용한 모듈을 만들려면 유저스페이스와 통신할 방법이 필요하다. 가장 기본적인 방식이 캐릭터 디바이스(`/dev/mydevice`)를 만드는 것이다.

```c
#include <linux/fs.h>
#include <linux/cdev.h>
#include <linux/uaccess.h>

#define DEVICE_NAME "mydevice"
#define BUF_SIZE 256

static dev_t dev_num;
static struct cdev my_cdev;
static struct class *my_class;
static char kernel_buf[BUF_SIZE];

static ssize_t my_read(struct file *f, char __user *buf,
                       size_t len, loff_t *off)
{
    size_t to_copy = min(len, (size_t)(BUF_SIZE - *off));
    if (copy_to_user(buf, kernel_buf + *off, to_copy))
        return -EFAULT;
    *off += to_copy;
    return to_copy;
}

static ssize_t my_write(struct file *f, const char __user *buf,
                        size_t len, loff_t *off)
{
    size_t to_copy = min(len, (size_t)BUF_SIZE);
    if (copy_from_user(kernel_buf, buf, to_copy))
        return -EFAULT;
    return to_copy;
}
```

`copy_to_user`와 `copy_from_user`는 커널-유저 경계를 안전하게 넘나드는 핵심 함수다. 직접 포인터를 역참조하면 커널 패닉이 발생할 수 있으므로 반드시 이 함수들을 사용해야 한다.

### file_operations 등록

`struct file_operations`에 읽기/쓰기 핸들러를 등록하고, `cdev_add`로 커널에 디바이스를 등록한다. `class_create`와 `device_create`를 통해 udev가 자동으로 `/dev/mydevice` 노드를 생성하도록 할 수 있다.

```c
static struct file_operations fops = {
    .owner = THIS_MODULE,
    .read = my_read,
    .write = my_write,
};
```

이렇게 등록된 디바이스는 `echo "test" > /dev/mydevice`와 `cat /dev/mydevice`로 간단히 테스트할 수 있다.

## 디버깅과 주의사항

커널 모듈 개발에서 가장 위험한 점은 버그가 시스템 전체를 멈추게 할 수 있다는 것이다. NULL 포인터 역참조, 잘못된 메모리 접근, 무한 루프 등은 즉시 커널 패닉으로 이어진다.

`pr_debug`와 `dynamic_debug`를 활용한 로깅, `ftrace`를 통한 함수 호출 추적, 그리고 가상 머신 내에서의 테스트가 필수적이다. 실제 하드웨어에서 직접 테스트하기 전에 QEMU 등으로 충분히 검증하는 습관을 들이자.

### DKMS로 관리하기

배포용 모듈이라면 DKMS(Dynamic Kernel Module Support)를 사용해 커널 업데이트 시 자동으로 재빌드되도록 설정하는 것이 좋다. `dkms.conf` 파일을 작성하고 `dkms install`로 등록하면 된다.
