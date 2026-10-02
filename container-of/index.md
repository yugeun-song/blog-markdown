# container_of 의 정의와 용법

리눅스 커널 소스코드를 분석하다 보면, 디렉토리를 구분하지 않고 `container_of` 매크로를 많이 사용하는 것을 볼 수 있다. 그중에서도 커널 자료구조, RCU, 동기화, 커널 구조체를 조작하는 형태의 코드에서 주로 보인다.

## `container_of`의 정의와 `offsetof`

`container_of`는 `include/linux/container_of.h`에 정의되어 있다.

<!-- source-ref id="container_of_h" -->

```c
/* SPDX-License-Identifier: GPL-2.0 */
#ifndef _LINUX_CONTAINER_OF_H
#define _LINUX_CONTAINER_OF_H

#include <linux/build_bug.h>
#include <linux/stddef.h>

#define typeof_member(T, m)	typeof(((T*)0)->m)

/**
 * container_of - cast a member of a structure out to the containing structure
 * @ptr:	the pointer to the member.
 * @type:	the type of the container struct this is embedded in.
 * @member:	the name of the member within the struct.
 *
 * WARNING: any const qualifier of @ptr is lost.
 */
#define container_of(ptr, type, member) ({				\
	void *__mptr = (void *)(ptr);					\
	static_assert(__same_type(*(ptr), ((type *)0)->member) ||	\
		      __same_type(*(ptr), void),			\
		      "pointer type mismatch in container_of()");	\
	((type *)(__mptr - offsetof(type, member))); }) /* [!hl] */

/**
 * container_of_const - cast a member of a structure out to the containing
 *			structure and preserve the const-ness of the pointer
 * @ptr:		the pointer to the member
 * @type:		the type of the container struct this is embedded in.
 * @member:		the name of the member within the struct.
 */
#define container_of_const(ptr, type, member)				\
	_Generic(ptr,							\
		const typeof(*(ptr)) *: ((const type *)container_of(ptr, type, member)),\
		default: ((type *)container_of(ptr, type, member))	\
	)

#endif	/* _LINUX_CONTAINER_OF_H */
```

파일 전체의 소스 내용은 그렇게 많지 않다. 모든 비필수적인 기능들을 제거한 최소한의 코드만 놓고 보면 다음과 같다.

```c
#define container_of(ptr, type, member) ({				\
	(type *)((void *)(ptr) - offsetof(type, member)); })
```

`offsetof`의 정의는 다음과 같다.

<!-- source-ref id="offsetof" -->

```c
#define offsetof(TYPE, MEMBER)	__builtin_offsetof(TYPE, MEMBER)
```

결국 `container_of`는 `offsetof`를 사용한 **단순한 오프셋 계산식**이다. 원래 `offsetof`는 C 언어 표준이 `<stddef.h>`에 정의한 기능이다. 다만, 리눅스 커널은 `-nostdinc`로 빌드되어 표준 헤더를 쓰지 않으므로, `offsetof`처럼 C 언어 표준과 동일한 기능들을 자체적으로 정의한다. `offsetof`는 이와 같이 GCC/Clang이 지원하는 `__builtin_offsetof`를 이용한다.

다음 예제는 `__builtin_offsetof`로 각 멤버의 오프셋을 구하고, 실제 멤버 주소와 함께 출력한다.

```c
#include <stdio.h>

struct example {
    char    a;
    int     b;
    double  c;
    short   d;
};

int main(void)
{
    struct example e;

    printf("sizeof(struct example) = %zu\n", sizeof(struct example));

    printf("offsetof(a) = %zu\n", __builtin_offsetof(struct example, a));
    printf("offsetof(b) = %zu\n", __builtin_offsetof(struct example, b));
    printf("offsetof(c) = %zu\n", __builtin_offsetof(struct example, c));
    printf("offsetof(d) = %zu\n", __builtin_offsetof(struct example, d));

    printf("\n");

    printf("&e      = %p\n", (void *)&e);
    printf("&e.a    = %p\n", (void *)&e.a);
    printf("&e.b    = %p\n", (void *)&e.b);
    printf("&e.c    = %p\n", (void *)&e.c);
    printf("&e.d    = %p\n", (void *)&e.d);

    return 0;
}
```

실행 결과는 다음과 같다.

```text
sizeof(struct example) = 24
offsetof(a) = 0
offsetof(b) = 4
offsetof(c) = 8
offsetof(d) = 16

&e      = 0x7ffd329cce10
&e.a    = 0x7ffd329cce10
&e.b    = 0x7ffd329cce14
&e.c    = 0x7ffd329cce18
&e.d    = 0x7ffd329cce20
```

`__builtin_offsetof`는 표준 `offsetof`와 인자 순서도 결과도 같다. 둘 다 첫 번째 인자로 구조체 타입을, 두 번째 인자로 멤버 이름을 받는다. 그리고 구조체 시작에서 그 멤버까지의 거리를 `size_t` 타입의 바이트 수로 돌려준다. 실제로 GCC와 Clang의 `<stddef.h>`는 `offsetof`를 `__builtin_offsetof`로 정의한다. 이 거리에는 **멤버 앞에 들어간 패딩도 포함된다**. 위 결과에서 `b`의 오프셋이 1이 아니라 4인 것은 `char a` 뒤에 3바이트 패딩이 들어갔기 때문이다. 반면 마지막 멤버 `d`(오프셋 16, 2바이트) 뒤의 6바이트 패딩은 어느 멤버의 오프셋에도 들어가지 않고 `sizeof`에만 드러난다. 그래서 구조체 크기는 18이 아니라 24이다.

## `container_of`에 담긴 커널과 C의 설계 철학

`container_of`를 통해 커널의 설계 철학, 더 나아가서는 C 언어의 설계 철학을 알 수 있다.

**[LEMMA]** 타입이 다르면 같은 시작 주소에서도 연산의 대상이 되는 메모리 영역의 범위가 달라진다(예: `int`, `char`, `void *` 타입 등).

**[THEOREM]** `container_of` 매크로 자체는 아는 정보가 하나도 없다. `container_of`를 활용하는 외부 자료구조에 데이터(구조체)의 구조 및 사용 방법의 책임을 넘긴다.

**[COROLLARY]** 그래서 `container_of`를 다양한 커널 자료구조에서 사용할 수 있다.

이는 여타 객체지향 언어에서 자주 쓰이는 iterable하고 상속 가능한 것을, C 언어가 허용하는 범위 안에서 최대한 strict하고 type-safe하게 구현하고자 했던 각종 노력의 결과물이다. 그래서 후술하겠지만, `static_assert` 같은 기능을 이용하여 타입 안전성을 검사한다.

## `container_of`의 용법: 리스트 순회 예제

```c
// SPDX-License-Identifier: 0BSD
#include <linux/list.h>
#include <linux/module.h>

struct user_info {
	char username[256];
	struct list_head list;
	u8 age;
};

static LIST_HEAD(user_info_list);

static struct user_info users[] = {
	{ .username = "alice", .age = 20 },
	{ .username = "bob", .age = 30 },
	{ .username = "carol", .age = 40 },
};

static int __init user_info_init(void)
{
	struct list_head *pos;
	struct user_info *user;
	size_t i;

	pr_info("user_info: init\n");

	for (i = 0; i < ARRAY_SIZE(users); ++i) {
		user = &users[i];
		list_add_tail(&user->list, &user_info_list);
	}

	list_for_each(pos, &user_info_list) {
		user = container_of(pos, struct user_info, list); /* [!hl] */
		pr_info("username=%s age=%u\n", user->username, user->age);
	}

	return 0;
}

static void __exit user_info_exit(void)
{
	pr_info("user_info: exit\n");
}

module_init(user_info_init);
module_exit(user_info_exit);

MODULE_LICENSE("Dual BSD/GPL");
MODULE_DESCRIPTION("container_of over a list_head list");
```

위 코드는 `container_of`를 활용하는 예제 코드이다. 이와 같이 `container_of`는 그 자체로서는 별 기능이 없다. 그리고 `container_of`에 넘기는 타입과 멤버 이름은 **`ptr`이 실제로 속한 상위 구조체의 타입, 멤버 이름과 일치해야** 올바르게 작동한다.

```memory-layout
{
  "id": "container-of-user-info",
  "label": "container_of(pos, struct user_info, list)의 계산: users[0]은 0xffffffffc0203020에서 시작하고, ptr인 pos는 list를 가리키며, offsetof는 0x100이고, 결과 user는 users[0]의 시작을 가리킨다. users[1](bob)은 그 다음 주소 0xffffffffc0203138에서 시작한다",
  "regions": [
    {"word": "...", "sub": "(users[1], bob)", "start": "0xffffffffc0203138", "h": 76, "marker": "users[1]", "tone": "orange"},
    {"id": "pad", "word": "00 00 00 00 00 00 00", "sub": "(padding, +0x111)", "start": "0xffffffffc0203131"},
    {"value": "0x14", "sub": "(age, +0x110)", "start": "0xffffffffc0203130"},
    {"id": "prev", "value": "0xffffffffc0203370", "sub": "(list.prev, +0x108)", "start": "0xffffffffc0203128"},
    {"id": "next", "value": "0xffffffffc0203238", "sub": "(list.next, +0x100)", "start": "0xffffffffc0203120", "marker": "ptr: pos", "tone": "red"},
    {"id": "name", "word": "\"alice\"", "sub": "(username, +0x000)", "start": "0xffffffffc0203020", "marker": "user == &users[0]", "tone": "blue"}
  ],
  "spans": [
    {"from": "pad", "to": "name", "label": "type", "sub": ["struct user_info", "(users[0])"], "tone": "orange"},
    {"from": "prev", "to": "next", "label": "member", "sub": "list (+0x100 .. +0x10f)", "tone": "purple"},
    {"from": "name", "to": "name", "label": "offsetof", "sub": "(type, member) = 0x100"}
  ],
  "code": [
    [["user", "blue"], " = container_of(", ["pos", "red"], ", ", ["struct user_info", "orange"], ", ", ["list", "purple"], ");"],
    [["(struct user_info *)", "orange"], "(", ["0xffffffffc0203120", "red"], " - 0x100) = ", ["0xffffffffc0203020", "blue"]],
    [["user", "blue"], " == &users[0]"]
  ]
}
```

다이어그램 1과 같이 `struct user_info`는 총 280바이트이고, 맨 뒤에 7바이트 패딩이 붙는다. `user = container_of(pos, struct user_info, list);`는 `pos`에서 바깥 구조체의 주소를 역산한다. `pos`는 `user_info` 안에 든 `list` 멤버의 주소이고, `list`의 타입은 `struct list_head`이다. 이 주소에서 `list`의 오프셋인 `offsetof(struct user_info, list)`, 즉 0x100을 빼면 `list`를 품은 `user_info`의 시작 주소가 나온다.

`list_head`를 쓰는 구조체는 `user_info`만이 아니다. `struct task_struct`는 `tasks` 멤버로 모든 프로세스를, `struct module`은 `list` 멤버로 적재된 모듈을, `struct net_device`는 `dev_list` 멤버로 같은 네트워크 네임스페이스의 장치를 잇는다. 세 멤버 `tasks`, `list`, `dev_list`의 타입은 모두 `list_head`이다. 리스트를 잇고, 끊고, 순회하는 연산(`list_add_tail`, `list_del`, `list_for_each`)은 `list_head`만 다룬다. 원소의 데이터가 필요할 때만 `container_of`로 바깥 구조체를 구한다. 그래서 구조체마다 `next()` 같은 순회 함수나 getter 같은 접근 함수를 따로 만들 필요가 없다. 어떤 구조체든 `list_head`를 멤버로 품기만 하면 같은 인터페이스를 그대로 쓴다. **따라서 구조체는 자기 정보를 온전하게 담는 데에만 집중하고, 탐색과 값 추출은 이미 있는 인터페이스에 맡기면 된다.**

다이어그램 2는 예제 모듈의 리스트를 이 관점에서 그린 것이다. `list_for_each`는 `next`를 따라 `list_head`만 옮겨 다니고, 각 원소의 바깥 구조체는 `container_of`가 `offsetof`만큼 거슬러 올라가 구한다.

```struct-chain
{
  "id": "user-info-list",
  "label": "user_info_list에서 시작한 list_for_each가 각 list 멤버의 next만 따라 users[0], users[1], users[2]를 거쳐 user_info_list로 돌아온다. 각 노드의 구조체 시작은 pos에서 offsetof 0x100을 뺀 container_of 결과이다",
  "fields": [["username", 256], ["list", 16], ["age", 1], {"pad": 7}],
  "link": {"field": "list", "cells": ["next", "prev"]},
  "head": {"name": "user_info_list", "sub": "(head)"},
  "nodes": [
    {"name": "users[0]", "values": {"username": "\"alice\"", "age": "20"}},
    {"name": "users[1]", "values": {"username": "\"bob\"", "age": "30"}},
    {"name": "users[2]", "values": {"username": "\"carol\"", "age": "40"}}
  ],
  "tones": {"node": "orange", "link": "purple", "walk": "red", "offset": "blue"},
  "code": [
    ["list_for_each(", ["pos", "red"], ", ", ["&user_info_list", "purple"], ")"],
    ["    ", ["user", "blue"], " = container_of(", ["pos", "red"], ", ", ["struct user_info", "orange"], ", ", ["list", "purple"], ");"]
  ]
}
```
