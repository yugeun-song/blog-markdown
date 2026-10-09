# container_of의 정의와 용법

`container_of`는 구조체 멤버의 포인터로 그 멤버를 품은 구조체의 포인터를 구하는 매크로이다. 리눅스 커널 소스코드를 분석하다 보면, 디렉토리를 구분하지 않고 이 매크로를 많이 사용하는 것을 볼 수 있다. 그중에서도 커널 자료구조, RCU, 동기화, 커널 구조체를 조작하는 형태의 코드에서 주로 보인다. 이런 코드는 구조체 전체 대신 그 안에 넣은 멤버의 포인터를 주고받고, 바깥 구조체가 필요할 때 `container_of`로 되찾는다.

## `container_of`의 인자와 결과

`container_of(ptr, type, member)`의 목적은 `member`를 품은 `type` 객체의 시작 주소를 구하는 것이다. 다만 실제로 하는 일은 `ptr`의 값에서 `offsetof(type, member)`를 빼고, 그 결과를 `type *`로 바꾸는 것뿐이다. `offsetof(type, member)`는 `type`의 시작에서 `member`까지의 바이트 수이다.

세 인자는 다음과 같다.

- `ptr`: `type` 객체 안의 `member`를 가리켜야 하는 포인터.
- `type`: `member`를 품은 구조체의 타입. `offsetof`의 첫 인자이자 결과 포인터의 타입이다.
- `member`: `type` 안에서 그 멤버의 이름. 값이 아니라 이름이고, `offsetof`의 두 번째 인자이다.

**결과를 `type` 객체의 포인터로 쓰려면, `ptr`의 값이 어떤 `type` 객체 안 `member`의 주소와 정확히 같아야 한다.** 이것이 `container_of`의 전제이다. 뺄셈의 결과가 시작 주소와 같아지는 것은 이때뿐이기 때문이다.

전제는 호출하는 쪽이 지켜야 한다. `container_of`는 전제가 성립하는지 확인하지 않고, 컴파일 시점에 비교하는 것도 타입뿐이기 때문이다.

전제가 틀려도 매크로는 같은 뺄셈을 하고, 어떤 객체의 시작도 아닌 주소를 돌려준다. 그렇다고 반드시 크래시가 나지는 않는다. 어떤 결과로 이어지는지는 뒤에서 다룬다.

또 결과에서는 `const`가 빠진다. `ptr`이 `const` 객체를 가리켜도 결과는 `type *`이다. `const`를 지키려면 같은 헤더의 `container_of_const`를 쓴다.

## `container_of`의 용법: 리스트 순회 예제

리스트 순회는 `container_of`의 대표적인 용법이다. 커널의 리스트는 원소 구조체에 넣은 `struct list_head`끼리 이어지므로, 순회하면서 얻는 포인터는 `list_head`의 주소이다. 원소의 데이터를 쓰려면 이 주소에서 `container_of`로 원소 구조체의 주소를 구해야 한다.

다음 모듈은 `struct list_head`를 `list` 멤버로 품은 `struct user_info` 세 개를 리스트로 잇는다. 그리고 `list_for_each`로 리스트를 순회하면서, `container_of`로 각 원소를 구한다.

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

`container_of(pos, struct user_info, list)`는 `pos`에서 `list`의 오프셋인 `offsetof(struct user_info, list)`, 즉 0x100을 빼서 `list`를 품은 `user_info`의 시작 주소를 구한다. 이 호출에서는 전제가 성립한다. `list_for_each`가 `pos`에 넣어 주는 값이 각 `user_info` 안에 든 `list` 멤버, 즉 `struct list_head`의 주소이기 때문이다.

다이어그램 1은 `users[0]`에서 이 계산을 그린 것이다. `struct user_info`는 총 280바이트이고, 맨 뒤에 7바이트 패딩이 붙는다.

```memory-layout
{
  "id": "container-of-user-info",
  "label": "container_of(pos, struct user_info, list)의 계산: users[0]은 0xffffffffc0203020에서 시작하고, ptr인 pos는 list를 가리키며, offsetof는 0x100이고, 결과 user는 users[0]의 시작을 가리킨다. users[1](bob)은 그 다음 주소 0xffffffffc0203138에서 시작한다",
  "regions": [
    {"word": "...", "sub": "(users[1], bob)", "start": "0xffffffffc0203138", "marker": "users[1]", "tone": "orange"},
    {"id": "pad", "word": "00 00 00 00 00 00 00", "sub": "(padding, +0x111)", "start": "0xffffffffc0203131"},
    {"value": "0x14", "sub": "(age, +0x110)", "start": "0xffffffffc0203130"},
    {"id": "prev", "value": "0xffffffffc0203370", "sub": "(list.prev, +0x108)", "start": "0xffffffffc0203128"},
    {"id": "next", "value": "0xffffffffc0203238", "sub": "(list.next, +0x100)", "start": "0xffffffffc0203120", "marker": "ptr: pos", "tone": "red"},
    {"id": "name", "word": "\"alice\"", "sub": "(username, +0x000)", "start": "0xffffffffc0203020", "h": 120, "marker": "user == &users[0]", "tone": "blue"}
  ],
  "code": [
    [["user", "blue"], " = container_of(", ["pos", "red"], ", ", ["struct user_info", "orange"], ", ", ["list", "purple"], ");"],
    [["(struct user_info *)", "orange"], "(", ["0xffffffffc0203120", "red"], " - 0x100) = ", ["0xffffffffc0203020", "blue"]],
    [["user", "blue"], " == &users[0]"]
  ]
}
```

구조체 타입을 $T$, 그 타입의 객체를 $s$, $s$의 멤버를 $m$이라 하면, 이 계산은 다음 식과 같다.

$$
\def\arraystretch{1.3}
\begin{array}{rll}
 & \operatorname{addr}(s.m) & \small\text{절대 주소} \\
- & \operatorname{offsetof}(T, m) & \small\text{상대 주소} \\
= & \operatorname{addr}(s) & \small\text{시작 주소}
\end{array}
$$

$T$와 $m$은 각각 `container_of`의 `type`과 `member`이고, 절대 주소는 `ptr`의 값이다. 다이어그램 1에서는 $s$가 `users[0]`, $m$이 `list`이므로, `pos`의 값이 절대 주소이고 `user`의 값이 시작 주소이다.

`user`의 값은 첫 멤버 `username`의 주소와도 같다. C 표준에서 구조체 맨 앞에는 패딩이 올 수 없기 때문이다.

다이어그램 2는 예제 모듈의 리스트 전체를 그린 것이다. `list_for_each`는 `next`를 따라 `list_head`만 옮겨 다니고, 각 원소의 바깥 구조체는 `container_of`가 `offsetof`만큼 거슬러 올라가 구한다.

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

## `container_of`의 구현

`container_of`는 `offsetof`를 사용한 **단순한 오프셋 계산식**이다. 정의는 `include/linux/container_of.h`에 있다.

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

파일 전체의 소스 내용은 그렇게 많지 않다. 모든 비필수적인 기능을 제거한 최소한의 코드만 놓고 보면 다음과 같다.

```c
#define container_of(ptr, type, member) ({				\
	(type *)((void *)(ptr) - offsetof(type, member)); })
```

매크로는 `offsetof`가 돌려준 바이트 수를 그대로 뺀다. `ptr`을 먼저 `void *`로 바꾸고, GNU C에서 `void *`의 덧셈과 뺄셈은 1바이트 단위이기 때문이다.

전제가 지켜지면 결과 주소는 `ptr`과 같거나 그보다 앞에 있다. `offsetof(type, member)`는 0 이상이고, `container_of`가 빼는 값은 이 오프셋 하나이기 때문이다. 결과가 `ptr`과 같아지는 것은 `member`가 구조체 맨 앞에 있어 오프셋이 0일 때이다.

인용 1의 주석에 있는 `WARNING`은 앞에서 본 대로 결과에서 `const`가 빠진다는 경고이다.

### `offsetof`

`offsetof(TYPE, MEMBER)`는 구조체 시작에서 그 멤버까지의 거리를 `size_t` 타입의 바이트 수로 돌려준다. 첫 번째 인자는 구조체 타입이고, 두 번째 인자는 멤버 이름이다.

커널의 `offsetof`는 GCC와 Clang이 지원하는 `__builtin_offsetof`를 그대로 쓴다. 원래 `offsetof`는 C 언어 표준이 `<stddef.h>`에 정의한 기능이다. 그러나 리눅스 커널은 `-nostdinc`로 빌드되어 표준 헤더를 쓰지 않으므로, `offsetof`처럼 C 언어 표준과 같은 기능을 자체적으로 정의한다.

<!-- source-ref id="offsetof" -->

```c
#define offsetof(TYPE, MEMBER)	__builtin_offsetof(TYPE, MEMBER)
```

`__builtin_offsetof`는 표준 `offsetof`와 인자 순서도 결과도 같다. GCC와 Clang의 `<stddef.h>`도 `offsetof`를 `__builtin_offsetof`로 정의하기 때문이다.

다음 예제는 `__builtin_offsetof`로 각 멤버의 오프셋을 구하고, 실제 멤버 주소와 함께 출력한다.

```c
#include <stdio.h>

struct example {
    char a;
    int b;
    double c;
    short d;
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

    printf("&e   = %p\n", (void *)&e);
    printf("&e.a = %p\n", (void *)&e.a);
    printf("&e.b = %p\n", (void *)&e.b);
    printf("&e.c = %p\n", (void *)&e.c);
    printf("&e.d = %p\n", (void *)&e.d);

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

&e   = 0x7ffd329cce10
&e.a = 0x7ffd329cce10
&e.b = 0x7ffd329cce14
&e.c = 0x7ffd329cce18
&e.d = 0x7ffd329cce20
```

출력된 주소를 보면, 각 멤버의 주소는 `&e`에 그 멤버의 오프셋을 더한 값이다. 컴파일러가 `e.b` 같은 멤버의 주소를 구하는 방식도 이와 같다. `container_of`의 뺄셈은 이 덧셈의 역연산이다.

오프셋에는 **멤버 앞에 들어간 패딩도 포함된다**. 위 결과에서 `b`의 오프셋이 1이 아니라 4인 것은 `char a` 뒤에 3바이트 패딩이 들어갔기 때문이다.

반면 마지막 멤버 뒤의 패딩은 어느 멤버의 오프셋에도 들어가지 않고 `sizeof`에만 드러난다. 위 결과에서는 `d`(오프셋 16, 2바이트) 뒤에 6바이트 패딩이 붙어, 구조체 크기가 18이 아니라 24이다.

### `typeof`

`typeof(식)`은 그 식의 타입을 나타낸다. 결과는 타입이므로 변수 선언이나 캐스트처럼 타입 이름이 들어갈 자리에 쓴다. 예를 들어 `int x`가 있으면 `typeof(x)`는 `int`이고, `typeof(&x)`는 `int *`이다.

커널은 GNU 확장인 `typeof`를 쓴다. `-std=gnu11`로 빌드하기 때문이다. `typeof`는 오랫동안 GNU C의 확장이었다가 C23에서 표준이 되었다.

`typeof`는 피연산자의 타입만 쓰고, 피연산자를 평가하지 않는다. 예외는 가변 길이 배열이 들어간 타입뿐이다. 다음 예제는 이 성질을 보여 준다.

```c
#include <stdio.h>

int main(void)
{
    int x = 1;
    typeof(x) y = 2;
    typeof(&x) p = &x;
    typeof(++x) z = 3;

    printf("x = %d, y = %d, *p = %d, z = %d\n", x, y, *p, z);

    return 0;
}
```

실행 결과는 다음과 같다.

```text
x = 1, y = 2, *p = 1, z = 3
```

출력에서 `x`는 1 그대로이다. `typeof(++x)`는 `int`가 되지만, `++x`는 실행되지 않기 때문이다. `++x`가 실행되었다면 `x`와 `*p`는 2로 출력되었을 것이다.

인용 1의 `container_of_const`는 `typeof`를 써서, `ptr`이 `const` 객체를 가리킬 때만 결과에도 `const`를 남긴다. `const typeof(*(ptr)) *`는 `ptr`이 가리키는 타입에 `const`를 붙인 포인터 타입이다.

C11의 `_Generic`은 `ptr`의 타입에 맞는 분기를 고른다. `ptr`이 이 타입이면 결과를 `const type *`로, 아니면 `type *`로 바꾼다.

### `((type *)0)->member`가 컴파일되는 이유

`((type *)0)->member`는 널 포인터를 역참조하지 않는다. 이 식은 타입을 정하는 데에만 쓰이고, **메모리를 읽는 코드가 되지 않기 때문이다**.

인용 1에는 이 꼴의 식이 두 번 나온다. `typeof_member`의 정의와 `static_assert` 안이다. 두 곳 모두 `0`을 `type *`로 바꾼 널 포인터로 멤버에 접근하므로, 겉보기에는 `NULL` 역참조이다.

그래도 이 식은 문법과 타입 모두 올바르다. 정수 상수 `0`은 널 포인터 상수이고, `(type *)0`은 `type *` 타입의 널 포인터이다. `->`는 포인터가 가리키는 타입에서 멤버를 찾는다. 컴파일러는 `type`의 구조를 알고 있으므로, 포인터의 값과 관계없이 이 식의 타입을 `member`의 타입으로 정한다.

문제는 이 식을 평가할 때만 생긴다. 앞에서 본 것처럼 `typeof`는 피연산자를 평가하지 않고, `sizeof`도 마찬가지이다. 커널의 `typeof_member()`와 `sizeof_field()`는 이 성질로 멤버의 타입과 크기를 구한다.

`static_assert`의 두 식 `*(ptr)`과 `((type *)0)->member`도 메모리를 읽는 코드가 되지 않는다. 그 안의 `__same_type(a, b)`는 `__builtin_types_compatible_p(typeof(a), typeof(b))`로 정의되어 있다. 이 builtin은 두 타입이 같은지를 컴파일 시점의 상수 1이나 0으로 바꾼다.

다음 예제는 같은 식을 네 가지로 쓰고, 결과를 각각 변수에 대입한다. 그중 메모리를 읽는 것은 식을 그대로 평가하는 `value` 하나뿐이다. `struct example`는 예제 3과 같다.

```c
#include <stddef.h>

struct example {
    char a;
    int b;
    double c;
    short d;
};

int main(void)
{
    int same = __builtin_types_compatible_p(typeof(((struct example *)0)->c),
                                            double);
    size_t size = sizeof(((struct example *)0)->c);
    size_t offset = (size_t)&((struct example *)0)->c;
    double value = ((struct example *)0)->c;

    return 0;
}
```

x86-64 GCC 16.2로 최적화 없이(`-O0`) 컴파일한 AT&T 문법 어셈블리는 다음과 같다. 읽기 쉽도록 `-fomit-frame-pointer`로 프레임 포인터 코드를 빼고, 어셈블러 지시어는 생략했다.

<!-- caption kind="none" -->
```asm
main:
	movl	$1, -28(%rsp)	/* same = 1 */ /* [!hl] */
	movq	$8, -24(%rsp)	/* size = 8 */ /* [!hl] */
	movq	$8, -16(%rsp)	/* offset = 8 */ /* [!hl] */
	movl	$0, %eax	/* rax = (struct example *)0 */ /* [!hl] */
	movsd	8(%rax), %xmm0	/* load ->c from address 8 */ /* [!hl] */
	movsd	%xmm0, -8(%rsp)	/* value = the loaded double */ /* [!hl] */
	movl	$0, %eax
	ret
```

`same`, `size`, `offset`에는 상수 1, 8, 8이 그대로 저장되고, 최적화를 끈 상태에서도 메모리를 읽는 명령은 없다. 세 값은 컴파일 시점에 정해지기 때문이다. 네 대입문은 위에서부터 같은 순서로 명령이 되고, `-28(%rsp)`, `-24(%rsp)`, `-16(%rsp)`는 `same`, `size`, `offset`의 스택 자리이다.

`value`만 메모리를 읽는다. `movsd 8(%rax), %xmm0`이 주소 0 + 8에서 8바이트를 읽으므로, 이 프로그램을 실행하면 이 대입에서 segmentation fault로 끝난다. aarch64에서도 세 변수에는 상수가 저장되고, `value`만 주소 8을 읽는다.

`offset`에 대입하는 식은 평가되지만, 멤버의 값이 아니라 주소만 구한다. 주소 0에 `struct example`가 있다고 치면 `c`의 주소는 곧 `c`의 오프셋 8이다. 커널의 `include/linux/stddef.h`도 v5.18 전까지는 이 식을 `offsetof`의 대체 정의로 두었다. 지금은 이 대체 정의가 없고, 커널은 `__builtin_offsetof`만 쓴다.

널 포인터 꼴에는 흠이 두 가지 있다. 널 포인터로 멤버에 접근하는 것은 C 표준에서 정의되지 않은 동작이다. 또 표준 C에서는 이 식이 정수 상수식도 아니어서, `_Static_assert`처럼 정수 상수식이 필요한 자리에 쓰면 GCC와 Clang은 `-pedantic`에서 경고를 낸다.

`__builtin_offsetof`도 GCC 안에서는 같은 꼴로 구현되어 있다. GCC의 C 프런트엔드는 `*(T *)0`에 멤버 참조를 이어 붙인 식을 만들고, `fold_offsetof()`로 각 멤버의 오프셋을 더해 상수로 바꾼다. 이 계산은 컴파일러 안에서 끝나므로 앞의 두 흠이 없다. 반면 Clang은 이 꼴을 쓰지 않고, 구조체 레이아웃에 기록된 멤버 오프셋을 바로 더한다.

## `container_of`와 타입 안전성

`container_of`가 보장하는 타입 안전성은 컴파일 시점의 타입 비교 하나이다. 타입만 맞으면 전제가 깨져도 컴파일되고, 실행 중에도 크래시 없이 지나가기도 한다.

### `container_of`가 검사하는 것

인용 1의 `static_assert`는 `*ptr`의 타입이 `member`의 타입과 같은지, 또는 `ptr`이 `void *`인지만 컴파일 시점에 검사한다. 실행 시점의 검사는 없다. `container_of`는 메모리를 읽지 않고, 주소에서 상수를 빼고 타입을 바꿀 뿐이기 때문이다.

따라서 타입이 맞는 실수는 그대로 컴파일된다. 예를 들어 `struct task_struct`의 자식 리스트에서 `member` 자리에 `sibling` 대신 `children`을 써도 컴파일은 된다. 두 멤버가 모두 `struct list_head`이기 때문이다. 이때는 엉뚱한 오프셋을 빼게 된다.

이 리스트에서 `member`는 `sibling`이어야 한다. 두 멤버는 함께 자식 프로세스의 리스트를 이루고, 부모의 `children`에 이어지는 노드는 각 자식의 `sibling`이기 때문이다.

`list_for_each_entry()`는 `list_for_each`의 순회와 `container_of`를 한 번에 하는 매크로이고, 세 번째 인자가 `member`이다. 커널은 자식 리스트를 `list_for_each_entry(p, &father->children, sibling)`처럼 순회한다. 이 매크로는 `container_of`의 `type` 자리에 `typeof(*pos)`를 넘긴다. 첫 원소를 구하는 시점의 `pos`는 아직 초기화 전일 수 있지만, `typeof`는 `*pos`를 평가하지 않으므로 문제가 없다.

어느 `type` 객체에도 들어 있지 않은 주소도, 타입만 맞으면 컴파일된다. 예제의 리스트 머리 `user_info_list`는 `struct list_head`이지만, 어느 `user_info`에도 들어 있지 않다. 그래서 `list_for_each`는 머리로 돌아오면 순회를 멈추고, 머리에는 `container_of`를 적용하지 않는다.

### 전제가 깨졌을 때

크래시가 없다는 사실만으로는 전제가 지켜졌는지 알 수 없다. 전제가 깨져도 `container_of` 자체는 메모리를 읽지 않으므로 크래시를 내지 않는다. 결과 주소로 무슨 일이 생기는지는 그 주소에 무엇이 있는지, 결과를 어떻게 쓰는지에 따라 다르다.

- 결과가 매핑되지 않은 메모리를 가리키면, 역참조하는 순간 page fault가 나고 커널은 oops를 낸다.
- 결과가 접근할 수 있는 메모리를 가리키면, 크래시 없이 엉뚱한 값을 읽고 쓴다.
- 결과를 역참조하지 않으면 메모리 접근 자체가 없다.

첫 번째 경우의 흔한 예는 `ptr`이 `NULL`일 때이다. 0에서 양수를 빼면 실제 결과는 주소 공간의 끝으로 넘어가므로, 예제의 `pos`가 `NULL`이라면 결과는 `0xffffffffffffff00`이다(64비트 기준).

이 값은 `NULL`이 아니므로 `NULL` 검사에도 걸리지 않는다. 그래서 마지막 노드의 `next`가 `NULL`인 `hlist`에서는 `hlist_for_each_entry()`가 `hlist_entry_safe()`로 노드를 변환한다. 이 매크로는 `ptr`이 `NULL`이면 `container_of`를 부르지 않고 `NULL`을 돌려준다.

두 번째 경우는 크래시보다 원인을 찾기 어렵다. 값이 조용히 틀리거나 다른 객체가 망가지고, 문제는 나중에 다른 곳에서 드러나기 때문이다. 앞에서 `sibling` 자리에 `children`을 쓴 실수가 이 경우이다.

세 번째 경우는 `list_for_each_entry()`가 이용한다. 이 매크로는 `list_for_each`와 달리, 순회가 끝날 때 리스트 머리에도 `container_of`를 적용한다. 결과 `pos`는 어느 원소의 시작도 아니다. 매크로는 이 `pos`를 역참조하지 않고, `&pos->member`로 머리의 주소를 되돌려 비교한 뒤 순회를 끝낸다. 순회가 끝난 뒤 이 `pos`를 원소처럼 쓰면 두 번째 경우가 된다.

## `member`와 `type`에 따른 주소 계산

전제만 지키면 `member`는 `ptr`이 가리키는 어느 멤버든 되고, `type`은 어느 단계의 바깥 구조체든 된다. 다만 멤버 사이를 오갈 때는 오프셋의 부호에 주의해야 한다.

### `member`의 역할

`member`는 `ptr`이 `type`의 어느 멤버를 가리키는지 알려 준다. `ptr`의 값만으로는 그 주소가 구조체의 어느 멤버인지 알 수 없기 때문이다. 따라서 `ptr`이 가리키는 멤버에 맞추면 `member`는 `list`가 아니어도 된다. 예를 들어 `container_of(&users[0].age, struct user_info, age)`는 `age`의 오프셋 0x110을 빼서 같은 `users[0]`을 구한다.

### 앞쪽 멤버와 오프셋의 부호

멤버 사이를 오갈 때는 `container_of`로 시작 주소를 구한 뒤, `->list`처럼 멤버에 접근하는 편이 안전하다. 두 오프셋의 차이를 직접 빼면 부호 문제가 생길 수 있기 때문이다.

뒤쪽 멤버의 주소로 앞쪽 멤버를 구할 때는 차이를 바로 빼도 된다. `age`를 가리키는 포인터에서 `age`와 `list`의 오프셋 차이인 0x10을 빼면 `list`가 나온다.

순서를 바꾸면 문제가 된다. `list`의 오프셋 0x100에서 `age`의 오프셋 0x110을 빼면 수학적으로 -0x10이지만, `offsetof`가 돌려주는 값은 부호 없는 `size_t`이다. 그래서 C에서 이 차이는 음수가 되지 않고 $2^{64} - \texttt{0x10}$으로 넘어간다(64비트 기준).

이 값을 포인터에서 빼는 것은 C 표준에서 정의되지 않은 동작이다. 실제 주소 계산에서는 0x10을 더한 결과가 나온다. 다이어그램 1에서 `age`보다 0x10바이트 뒤는 이미 `users[1]`의 영역이다.

### `type`과 구조체의 깊이

`container_of`가 올라가는 단계는 호출하는 쪽이 `type`으로 정한다. `type`으로 지정한 구조체가 더 큰 구조체 안에 들어 있어도, 더 바깥의 구조체는 계산에 들어가지 않는다. 예제의 결과가 가장 바깥 구조체와 같았던 것은 `user_info`가 다른 구조체에 들어 있지 않기 때문이다.

예를 들어 `struct platform_device`는 `dev` 멤버로 `struct device`를 품는다. `struct device *`만 받는 코드는 `to_platform_device()`로 한 단계 위의 `struct platform_device`를 구한다. 이 매크로의 정의는 `container_of((x), struct platform_device, dev)`이다.

<!-- source-ref id="platform_device" -->
```c
struct platform_device {
	const char	*name;
	int		id;
	bool		id_auto;
	struct device	dev; /* [!hl] */
	u64		platform_dma_mask;
	struct device_dma_parameters dma_parms;
	u32		num_resources;
	struct resource	*resource;

	const struct platform_device_id	*id_entry;

	/* MFD cell pointer */
	struct mfd_cell *mfd_cell;

	/* arch specific additions */
	struct pdev_archdata	archdata;
};

/* ... */

#define to_platform_device(x) container_of((x), struct platform_device, dev) /* [!hl] */
```

`struct device`를 품는 구조체는 장치마다 다르다. PCI 장치에서는 `struct pci_dev`가 `dev` 멤버로 품고, `to_pci_dev()`가 `struct pci_dev`를 구한다. 두 매크로는 `member`가 모두 `dev`이고 `type`만 다르다. 어느 쪽을 쓸지는 장치의 종류를 아는 호출하는 쪽이 정한다.

`struct pci_dev`는 200줄이 넘으므로, 인용 4에는 `dev` 멤버와 매크로만 남기고 나머지 줄을 생략했다.

<!-- source-ref id="pci_dev" -->
```c
struct pci_dev {
	struct list_head bus_list;	/* Node in per-bus list */
	struct pci_bus	*bus;		/* Bus this device is on */
	struct pci_bus	*subordinate;	/* Bus this device bridges to */

/* ... */

	struct device	dev;			/* Generic device interface */ /* [!hl] */

/* ... */

};

/* ... */

#define	to_pci_dev(n) container_of(n, struct pci_dev, dev) /* [!hl] */
```

`member` 자리에는 `a.b` 같은 멤버 경로도 들어간다. `offsetof`가 멤버 경로를 받기 때문이다. TCP 소켓은 `struct sock`을 세 겹으로 감싼다. 같은 `struct sock *`에서 출발해도, `type`과 `member`에 따라 도착하는 구조체가 다르다.

- `inet_sk()`는 `sk` 경로로 `struct inet_sock`을 구한다.
- `inet_csk()`는 `icsk_inet.sk` 경로로 `struct inet_connection_sock`을 구한다.
- `tcp_sk()`는 `inet_conn.icsk_inet.sk` 경로로 `struct tcp_sock`을 구한다.

세 결과는 `ptr`과 주소가 같고 타입만 다르다. 각 구조체가 바로 안쪽 구조체를 첫 멤버로 품어, 세 경로의 오프셋이 모두 0이기 때문이다. 바깥 구조체일수록 경로가 길고, 한 번에 건너뛰는 단계도 많다. 세 매크로는 모두 `container_of_const`로 정의되어 있다.

## `container_of`에 담긴 커널과 C의 설계 철학

`container_of`를 통해 커널의 설계 철학, 더 나아가서는 C 언어의 설계 철학을 알 수 있다. **구조체는 자기 정보를 온전하게 담는 데에만 집중하고, 탐색과 값 추출은 이미 있는 인터페이스에 맡기면 된다.**

**[THEOREM]** `container_of` 매크로 자체는 아는 정보가 하나도 없다. `container_of`를 활용하는 외부 자료구조에 데이터(구조체)의 구조 및 사용 방법의 책임을 넘긴다.

**[LEMMA]** 타입이 다르면 같은 시작 주소에서도 연산의 대상이 되는 메모리 영역의 범위가 달라진다(예: `int`, `char`, `void *` 타입 등).

**[COROLLARY]** 그래서 `container_of`를 다양한 커널 자료구조에서 사용할 수 있다.

이런 설계는 여타 객체지향 언어에서 자주 쓰이는 iterable하고 상속 가능한 것을, C 언어가 허용하는 범위 안에서 최대한 strict하고 type-safe하게 구현하고자 했던 각종 노력의 결과물이다. 그래서 앞에서 본 것처럼, `static_assert` 같은 기능을 이용하여 타입 안전성을 검사한다.

리스트가 이 설계의 대표적인 예이다. 리스트에 넣을 구조체마다 `next()` 같은 순회 함수나 getter 같은 접근 함수를 따로 만들 필요가 없다. 어떤 구조체든 `list_head`를 멤버로 품기만 하면 같은 인터페이스를 그대로 쓴다.

다이어그램 2의 순회처럼, 리스트를 잇고, 끊고, 순회하는 연산(`list_add_tail`, `list_del`, `list_for_each`)은 `list_head`만 다룬다. `container_of`는 원소의 데이터가 필요할 때만 쓴다.

`list_head`를 쓰는 구조체는 `user_info`만이 아니다. `struct task_struct`는 `tasks` 멤버로 모든 프로세스를, `struct module`은 `list` 멤버로 적재된 모듈을, `struct net_device`는 `dev_list` 멤버로 같은 네트워크 네임스페이스의 장치를 잇는다. 세 멤버 `tasks`, `list`, `dev_list`의 타입은 모두 `list_head`이다.
