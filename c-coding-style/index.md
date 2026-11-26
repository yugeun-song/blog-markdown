# C 코딩 스타일

한글 원문: [coding-style.ko.md](https://github.com/yugeun-song/c-programming/blob/main/docs/coding-style.ko.md)\
영문 원문: [coding-style.en.md](https://github.com/yugeun-song/c-programming/blob/main/docs/coding-style.en.md)

목차

- **[1. 적용 범위](#적용-범위)**
  - [1.1. 적용 대상](#적용-대상)
  - [1.2. 프로젝트와 종속성의 코딩 스타일 차이](#프로젝트와-종속성의-코딩-스타일-차이)
- **[2. 대원칙](#대원칙)**
  - [2.1. 단순하고 직관적인, 일반적인 코드](#단순하고-직관적인-일반적인-코드)
  - [2.2. 이름 충돌 피하기](#이름-충돌-피하기)
- **[3. 언어 표준과 플랫폼](#언어-표준과-플랫폼)**
  - [3.1. 언어 표준](#언어-표준)
  - [3.2. 플랫폼 API](#플랫폼-api)
  - [3.3. 확장 기능의 표시](#확장-기능의-표시)
  - [3.4. implementation-defined behavior](#implementation-defined-behavior)
  - [3.5. 표준 함수의 선택](#표준-함수의-선택)
  - [3.6. 쓰지 않는 표준 함수](#쓰지-않는-표준-함수)
- **[4. 파일](#파일)**
  - [4.1. 파일 이름](#파일-이름)
  - [4.2. 인코딩과 줄 끝](#인코딩과-줄-끝)
  - [4.3. 문자 집합](#문자-집합)
  - [4.4. 마지막 개행](#마지막-개행)
  - [4.5. 공백 문자](#공백-문자)
  - [4.6. 파일 안의 배치 순서](#파일-안의-배치-순서)
  - [4.7. feature test macro](#feature-test-macro)
  - [4.8. include 순서](#include-순서)
  - [4.9. include guard](#include-guard)
  - [4.10. 헤더의 내용](#헤더의-내용)
- **[5. 서식](#서식)**
  - [5.1. 들여쓰기](#들여쓰기)
  - [5.2. 줄 길이](#줄-길이)
  - [5.3. 긴 식 나누기](#긴-식-나누기)
  - [5.4. 사용자에게 보이는 문자열](#사용자에게-보이는-문자열)
  - [5.5. 함수 선언부](#함수-선언부)
  - [5.6. 함수 중괄호](#함수-중괄호)
  - [5.7. 제어문과 반복문 중괄호](#제어문과-반복문-중괄호)
  - [5.8. 중괄호 생략 금지](#중괄호-생략-금지)
  - [5.9. struct, union, enum 중괄호](#struct-union-enum-중괄호)
  - [5.10. 범위를 좁히는 블록](#범위를-좁히는-블록)
  - [5.11. 중괄호를 이용한 초기화](#중괄호를-이용한-초기화)
  - [5.12. switch와 case](#switch와-case)
  - [5.13. goto 레이블](#goto-레이블)
  - [5.14. 공백](#공백)
  - [5.15. 포인터 표기](#포인터-표기)
  - [5.16. 한 줄에는 가능한 한 문장만](#한-줄에는-가능한-한-문장만)
  - [5.17. 빈 줄](#빈-줄)
  - [5.18. 정렬](#정렬)
  - [5.19. preprocessing directive](#preprocessing-directive)
  - [5.20. GCC/Clang 인라인 어셈블리](#gccclang-인라인-어셈블리)
- **[6. 이름](#이름)**
  - [6.1. 함수와 변수의 이름](#함수와-변수의-이름)
  - [6.2. 매크로와 enumeration constant의 이름](#매크로와-enumeration-constant의-이름)
  - [6.3. function-like macro](#function-like-macro)
  - [6.4. const 객체의 이름](#const-객체의-이름)
  - [6.5. 이름의 길이와 내용](#이름의-길이와-내용)
  - [6.6. 함수 이름의 첫 단어](#함수-이름의-첫-단어)
  - [6.7. 참 거짓을 나타내는 이름](#참-거짓을-나타내는-이름)
  - [6.8. 전역 변수와 static 변수](#전역-변수와-static-변수)
  - [6.9. typedef](#typedef)
  - [6.10. 예약된 식별자](#예약된-식별자)
  - [6.11. Win32 API 심볼과 타입](#win32-api-심볼과-타입)
- **[7. 선언과 타입](#선언과-타입)**
  - [7.1. 한 줄에 선언 하나](#한-줄에-선언-하나)
  - [7.2. 선언 위치](#선언-위치)
  - [7.3. 범위와 shadowing](#범위와-shadowing)
  - [7.4. 수명이 끝난 대상을 가리키는 포인터](#수명이-끝난-대상을-가리키는-포인터)
  - [7.5. 정수 타입](#정수-타입)
  - [7.6. 참 거짓 값](#참-거짓-값)
  - [7.7. 상수](#상수)
  - [7.8. const](#const)
  - [7.9. register와 volatile](#register와-volatile)
  - [7.10. VLA](#vla)
  - [7.11. 초기화](#초기화)
- **[8. 식](#식)**
  - [8.1. 리터럴 표기](#리터럴-표기)
  - [8.2. 증감 연산자](#증감-연산자)
  - [8.3. sizeof](#sizeof)
  - [8.4. 캐스트](#캐스트)
  - [8.5. 우선순위 괄호](#우선순위-괄호)
  - [8.6. 비교](#비교)
  - [8.7. 널 포인터와 참 거짓 판정](#널-포인터와-참-거짓-판정)
  - [8.8. 삼항 연산자](#삼항-연산자)
  - [8.9. 할당](#할당)
  - [8.10. 쉼표 연산자](#쉼표-연산자)
- **[9. 문장](#문장)**
  - [9.1. switch](#switch)
  - [9.2. 빈 반복문 본문](#빈-반복문-본문)
  - [9.3. goto](#goto)
- **[10. 함수](#함수)**
  - [10.1. 매개변수 목록](#매개변수-목록)
  - [10.2. 매개변수 순서](#매개변수-순서)
  - [10.3. 포인터 매개변수](#포인터-매개변수)
  - [10.4. 반환값 규약](#반환값-규약)
  - [10.5. 함수의 크기](#함수의-크기)
  - [10.6. inline](#inline)
  - [10.7. 함수 배치](#함수-배치)
  - [10.8. main](#main)
- **[11. 전처리기와 매크로](#전처리기와-매크로)**
  - [11.1. 조건부 컴파일](#조건부-컴파일)
  - [11.2. 매크로를 대신할 수단](#매크로를-대신할-수단)
  - [11.3. 괄호](#괄호)
  - [11.4. 여러 문장 매크로](#여러-문장-매크로)
  - [11.5. 피해야 할 매크로](#피해야-할-매크로)
  - [11.6. 매크로 안의 지역 이름](#매크로-안의-지역-이름)
- **[12. 주석](#주석)**
  - [12.1. 주석의 양과 내용](#주석의-양과-내용)
  - [12.2. 주석 형식](#주석-형식)
  - [12.3. 함수 주석과 문서 주석](#함수-주석과-문서-주석)
- **[13. 오류 처리와 자원](#오류-처리와-자원)**
  - [13.1. 반환값 검사](#반환값-검사)
  - [13.2. unused 진단](#unused-진단)
  - [13.3. errno](#errno)
  - [13.4. 자원 정리](#자원-정리)
  - [13.5. 동적 메모리](#동적-메모리)
  - [13.6. assert](#assert)
  - [13.7. static assertion](#static-assertion)
  - [13.8. 종료](#종료)
- **[14. 도구](#도구)**
  - [14.1. clang-format](#clang-format)
  - [14.2. clang-tidy](#clang-tidy)
  - [14.3. 컴파일러 경고](#컴파일러-경고)
  - [14.4. 에디터 설정](#에디터-설정)
- **[15. 참고 자료](#참고-자료)**

## 적용 범위

### 적용 대상

이 코딩 스타일은 Linux를 플랫폼으로 하는 코드와, Linux도 대상에 포함하는 크로스플랫폼 코드에 쓰는 규약이다. 적용 대상은 C 소스(`.c`, `.h`)와 그 안의 인라인 어셈블리이다.

직접 작성하는 Linux 커널 모듈과 드라이버에는 커널 원본 소스와 별개로 이 코딩 스타일을 적용할 수 있다. 다만 Linux 커널 소스 자체를 커스터마이징할 때는 커널 코딩 스타일을 그대로 따른다. 커널 소스 안에 서로 다른 스타일이 섞이지 않게 하기 위해서이다.

Windows 전용, FreeBSD 전용, macOS 전용 프로젝트처럼 특정 OS나 아키텍처만 대상으로 하는 프로젝트는 그 OS나 아키텍처의 코딩 스타일을 따른다. 반면 크로스플랫폼 프로젝트에서는 특정 OS에 종속된 코드에도 이 코딩 스타일을 적용한다. 플랫폼마다 독립된 트리로 나누어 구조와 바이너리를 따로 두는 프로젝트는 크로스플랫폼 프로젝트가 아니다. 각 트리는 자기 플랫폼의 스타일을 따른다.

또한, 이 코딩 스타일은 AI 에이전트가 C 코드를 생성할 때 지켜야 할 규약의 모음이기도 하다. AI 에이전트는 이 규약을 기준으로 코드를 생성하고 검토할 수 있다. `.clang-format` 같은 포맷 설정으로는 검사할 수 없는 영역도 AI 에이전트가 이 규약에 따라 검사하고 수정안을 제안할 수 있다.

### 프로젝트와 종속성의 코딩 스타일 차이

프로젝트에서 직접 작성한 코드는 기존 코드까지 모두 이 규칙에 맞춰 고친다. 반면 외부에서 들여온 종속성 코드는 원래 스타일을 유지한다.

종속성 코드와 프로젝트 코드의 경계는 분명히 긋는다. 종속성 코드는 되도록 프로젝트 코드와 다른 디렉터리에 둔다. 같은 디렉터리에 두어야 하면 파일 단위로 경계를 긋는다. 어느 경우든 종속성 코드 파일에는 프로젝트의 코딩 스타일을 쓰지 않는다. 종속성 코드를 고쳐야 하면 그 파일의 원래 스타일대로 고친다. 이렇게 하면 프로젝트 안에 두 스타일이 함께 있어도 한 파일 안에는 한 스타일만 남는다. 한 파일에 서로 다른 스타일이 섞이면 읽는 사람이 어느 규칙을 따를지 헷갈린다. 종속성 코드를 새 버전으로 바꿀 때 바뀐 부분을 가려내기도 어렵다.

틀린 예(종속성 코드인 `third_party/tinyini/ini.c`에 프로젝트 함수를 더한 경우):

```c
static int
ini_skip_space (const char *s)
{
  int n = 0;
  while (s[n] == ' ' || s[n] == '\t')
    n++;
  return n;
}

static size_t count_sections(const char *text)
{
    size_t count = 0;

    for (size_t i = 0; text[i] != '\0'; ++i) {
        if (text[i] == '[') {
            ++count;
        }
    }

    return count;
}
```

올바른 예(종속성 코드를 다른 디렉터리에 둔 경우):

```text
src/
    main.c
    settings.c
    settings.h
third_party/
    tinyini/
        ini.c
        ini.h
```

올바른 예(종속성 코드인 `ini.c`, `ini.h`를 같은 디렉터리에 둔 경우):

```text
src/
    ini.c
    ini.h
    main.c
    settings.c
    settings.h
```

올바른 예(`src/settings.c`):

```c
#include <stddef.h>

static size_t count_sections(const char *text)
{
    size_t count = 0;

    for (size_t i = 0; text[i] != '\0'; ++i) {
        if (text[i] == '[') {
            ++count;
        }
    }

    return count;
}
```

## 대원칙

이 코딩 스타일의 토대는 Linux 커널 코딩 스타일과 MISRA C 규칙의 일부 아이디어이다. 중괄호, 이름, 선언, 주석의 모양은 커널을 따르되, 들여쓰기는 탭 대신 공백 4칸을 쓴다. MISRA C에서는 중괄호 생략 금지, 쉼표 연산자와 조건 안 할당의 제한, 정수와 포인터의 명시적 비교처럼 실수할 여지를 줄이는 규칙을 가져온다.

### 단순하고 직관적인, 일반적인 코드

코드는 짧게 작성하되 현학적으로 작성하지 않는다. 중의적으로 해석될 수 있는 부분은 최대한 지양하고, 지극히 단순하고 명료한 줄을 하나씩 쌓아 구현한다. 각 줄은 어셈블리 명령과 최대한 1:1에 가깝게 대응하도록 쓴다.

일부러 복잡하게 만들거나, 어려워 보이게 만들거나, 읽는 사람이 한 번 더 생각하게 만드는 코드는 쓰지 않는다. 생각할 여지와 실수할 여지가 많은 코드도 쓰지 않는다.

널리 쓰이는 형태로 쓴다. 실수를 줄여 주는 기법이라도 일반적인 코딩 스타일에서 크게 벗어난 방법이라면 쓰지 않는다. 예를 들어 상수를 왼쪽에 두는 비교(`5 == value`)에서는 `==`를 `=`로 잘못 쓰면 컴파일 오류가 난다. 그러나 널리 쓰이는 형태가 아니어서 읽는 사람의 익숙함과 어긋나고, 그 어긋남이 주는 부담이 얻는 이익보다 클 수도 있다. 이처럼 부작용이 실제 이익보다 클 수 있는 기법은 이 코딩 스타일에 넣지 않는다. 따라서, 제어문과 반복문에 최대한 할당문 형식의 코드를 지양하는 이유이기도 하다.

틀린 예:

```c
static long long sum_clamped(size_t *clipped, const int *v, int n, int lo, int hi)
{
    long long sum;

    for (sum = *clipped = 0; n-- > 0; ++v)
        sum += *v < lo ? (++*clipped, lo) : *v > hi ? (++*clipped, hi) : *v;
    return sum;
}
```

이 함수는 `v`가 가리키는 원소 `n`개를 `lo` 이상 `hi` 이하로 잘라 더하고, 잘린 원소의 수를 `*clipped`에 남긴다. 이 코드는 C99부터 지금의 표준까지 문법상 올바르다. undefined behavior도 없다. 결과도 아래 올바른 예와 같다. 그러나 한 줄 안에서 원소 읽기, 비교 두 번, 증가, 더하기가 쉼표 연산자와 중첩된 삼항 연산자로 얽혀 있다. 읽는 사람은 삼항 연산자가 어느 쪽으로 묶이는지, `n--`와 `++v`가 언제 실행되는지, `sum = *clipped = 0`이 어떤 순서로 할당되는지 하나씩 따져 봐야 한다. 이처럼 문법상 문제가 없더라도 생각할 거리를 늘려 읽고 고치는 시간을 낭비하게 만드는 코드는 쓰지 않는다.

올바른 예:

```c
static long long sum_clamped(size_t *mut_clipped, const int *samples, size_t count,
                             int low, int high)
{
    long long sum = 0;
    size_t clipped = 0;

    for (size_t i = 0; i < count; ++i) {
        int value = samples[i];

        if (value < low) {
            value = low;
            ++clipped;
        } else if (value > high) {
            value = high;
            ++clipped;
        }

        sum += value;
    }

    *mut_clipped = clipped;
    return sum;
}
```

각 줄은 읽기, 비교, 할당, 증가 가운데 한 가지만 하므로 어셈블리 명령 몇 개에 바로 대응한다. 틀린 예에서 바꾼 점은 다음과 같다.

- 포인터 매개변수를 옮기며 읽지 않고, 첨자 `samples[i]`로 원소를 읽는다. 포인터 매개변수 자체는 바꾸지 않는다.
- 원소 수는 `size_t`로 받는다. 반복 변수 `i`는 `for` 문의 초기식에서 선언하고 증감식에서만 바꾼다. 조건식에서는 `n--`처럼 값을 바꾸지 않는다.
- 쉼표 연산자와 중첩된 삼항 연산자를 `if`와 `else if`로 풀어, 한 줄에서 한 가지 일만 한다.
- 연쇄 할당 `sum = *clipped = 0`을 변수마다 선언과 초기화로 나눈다.
- 잘린 원소의 수는 지역 변수 `clipped`에 세고, 끝에서 한 번만 `*mut_clipped`에 쓴다. 매개변수를 이용해서 값을 수정해야 하는 경우에는 이름을 `mut_`으로 시작한다.
- `for` 문의 본문을 중괄호로 감싼다.
- 줄인 이름(`v`, `lo`, `hi`) 대신 뜻이 드러나는 이름(`samples`, `low`, `high`)을 쓴다.

### 이름 충돌 피하기

C 표준과 POSIX 같은 널리 쓰이는 표준의 이름, 그리고 C 키워드와 겹칠 여지가 큰 이름은 쓰지 않는다. 지금은 겹치지 않아도 새 표준이나 라이브러리가 같은 이름을 더하면 빌드가 깨진다. 예를 들어 C23 표준에서 `bool`, `true`, `false`, `nullptr`, `typeof`가 키워드가 되었다. 또 POSIX.1-2024에서는 `strlcpy`와 `strlcat`이 표준 함수가 되었다.

틀린 예:

```c
#define nullptr ((void *)0)

static const char *gs_last_error = nullptr;
```

```c
#include <string.h>

static size_t strlcpy(char *mut_dst, const char *src, size_t dst_size) // [!hl]
{
    size_t length = strlen(src);

    if (dst_size == 0) {
        return length;
    }

    size_t copy_length = (length < dst_size) ? length : dst_size - 1;

    memcpy(mut_dst, src, copy_length);
    mut_dst[copy_length] = '\0';
    return length;
}
```

올바른 예:

```c
static const char *gs_last_error = NULL;
```

```c
#include <string.h>

static size_t copy_string(char *mut_dst, const char *src, size_t dst_size) // [!hl]
{
    size_t length = strlen(src);

    if (dst_size == 0) {
        return length;
    }

    size_t copy_length = (length < dst_size) ? length : dst_size - 1;

    memcpy(mut_dst, src, copy_length);
    mut_dst[copy_length] = '\0';
    return length;
}
```

## 언어 표준과 플랫폼

### 언어 표준

기본 표준은 리눅스 커널과 동일한 GNU11(`-std=gnu11`, C11 표준과 GNU 확장)이다. 프로젝트는 필요하면 다른 표준이나 순수 ISO 표준 모드를 골라도 무방하다. 고른 표준은 빌드 설정에 명시한다. 컴파일러 기본값에 의존하지 않고 명시적인 설정을 한다.

- GCC와 Clang에서는 `-std`로 고른다(`-std=gnu11`, `-std=c11`, `-std=c99`, `-std=c17`, `-std=c23`, `-std=gnu99`).
- MSVC에는 C99 모드가 없다. `/std`를 주지 않은 기본 모드는 C89에 Microsoft 확장을 더한 것이다. 그 확장 가운데 일부가 C99 기능이다. C11 이상은 `/std:c11`, `/std:c17`, `/std:clatest`(`clang-cl`에서는 `/clang:-std=gnu11`)로 고른다. 기본 모드에서는 traditional preprocessor가 동작한다. 표준에 맞는 preprocessor는 `/Zc:preprocessor`로 고른다. `/std:c11`이나 `/std:c17`을 주면 이 preprocessor가 함께 켜진다.
- Visual Studio 프로젝트에서는 `LanguageStandard_C`(`stdc11`, `stdc17`, `stdclatest`)와 `UseStandardPreprocessor` 속성으로 고른다.
- CMake에서는 `CMAKE_C_STANDARD`와 `CMAKE_C_EXTENSIONS`로 고른다.

표준 문서와 WG14 공개 초안은 다음과 같다. 대응 관계는 [WG14 프로젝트 페이지](https://www.open-std.org/jtc1/sc22/wg14/www/projects)를 따른다.

- C99: [ISO/IEC 9899:1999](https://www.iso.org/standard/29237.html), 초안 [N1256](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf)
- C11: [ISO/IEC 9899:2011](https://www.iso.org/standard/57853.html), 초안 [N1570](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf)
- C17: [ISO/IEC 9899:2018](https://www.iso.org/standard/74528.html), 초안 [N2310](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n2310.pdf)
- C23: [ISO/IEC 9899:2024](https://www.iso.org/standard/82075.html), 초안 [N3220](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf)

틀린 예:

```make
CFLAGS += -Wall -Wextra -pedantic -Wshadow
```

올바른 예(GCC/Clang):

```make
CFLAGS += -std=gnu11 -Wall -Wextra -pedantic -Wshadow
```

올바른 예(MSVC에서 C11을 고르는 경우):

<!-- caption kind="none" -->
```bat
cl /std:c11 /W4 /c main.c
```

올바른 예(CMake):

```cmake
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)
```

### 플랫폼 API

스레드, 동기화, atomic 연산처럼 플랫폼마다 구현이 다른 기능에는 그 플랫폼과 컴파일러의 고유 API를 우선해서 쓴다. Linux에서는 POSIX threads(pthread)를, Windows에서는 Win32 API를 쓴다. atomic 연산에는 GCC/Clang에서 `__atomic` builtin을, Windows에서 Win32 API의 Interlocked 함수를 쓴다. C11의 `<threads.h>`와 `<stdatomic.h>`처럼 플랫폼 API 위에 한 겹을 더 씌운 표준 라이브러리는 지양한다.

C++나 Rust 같은 언어가 있는데도 C를 쓰는 이유는 어셈블리와 플랫폼에 거의 1:1로 대응하는 투명성 위에 최소한의 추상화만 얹은 구성을 원하기 때문이다. 플랫폼 API를 직접 쓰면 쓸데없는 추상화 계층을 거치지 않고, 동작을 정밀하고 투명하게 제어할 수 있다.

지양하는 예:

```c
#include <threads.h>

static mtx_t gs_hit_lock;
static size_t gs_hit_count;

static void record_hit(void)
{
    mtx_lock(&gs_hit_lock);
    ++gs_hit_count;
    mtx_unlock(&gs_hit_lock);
}
```

```c
#include <stdatomic.h>

static atomic_size_t gs_hit_count;

static void record_hit(void)
{
    atomic_fetch_add_explicit(&gs_hit_count, 1, memory_order_relaxed);
}
```

올바른 예(Linux):

```c
#include <pthread.h>

static pthread_mutex_t gs_hit_lock;
static size_t gs_hit_count;

static void record_hit(void)
{
    pthread_mutex_lock(&gs_hit_lock);
    ++gs_hit_count;
    pthread_mutex_unlock(&gs_hit_lock);
}
```

올바른 예(Windows):

```c
#include <windows.h>

static SRWLOCK gs_hit_lock;
static size_t gs_hit_count;

static void record_hit(void)
{
    AcquireSRWLockExclusive(&gs_hit_lock);
    ++gs_hit_count;
    ReleaseSRWLockExclusive(&gs_hit_lock);
}
```

올바른 예(GCC/Clang의 atomic builtin):

```c
static size_t gs_hit_count;

static void record_hit(void)
{
    __atomic_add_fetch(&gs_hit_count, 1, __ATOMIC_RELAXED);
}
```

올바른 예(Windows의 Interlocked 함수):

```c
#include <windows.h>

static LONG gs_hit_count;

static void record_hit(void)
{
    InterlockedIncrement(&gs_hit_count);
}
```

### 확장 기능의 표시

선택한 언어 표준 밖의 기능은 드러나게 쓴다.

- GNU 확장 문법(statement expression 등)은 `__extension__`으로 감싼다.
- 기본 표준인 GNU11(`-std=gnu11`)에서는 인라인 어셈블리에 `asm`을 쓴다(리눅스 커널 관례와 같다). `-std=c11` 같은 순수 ISO 표준 모드를 고르거나 그런 모드에서 포함될 공개 헤더에서만 `__asm__`, `__typeof__`처럼 밑줄 철자를 쓴다.

틀린 예:

```c
#define ABS_DIFF(a, b)                     \
    ({                                     \ // [!hl]
        __typeof__(a) a_ = (a);            \
        __typeof__(b) b_ = (b);            \
        (a_ > b_) ? (a_ - b_) : (b_ - a_); \
    })
```

올바른 예:

```c
#define ABS_DIFF(a, b)                     \
    __extension__({                        \ // [!hl]
        __typeof__(a) a_ = (a);            \
        __typeof__(b) b_ = (b);            \
        (a_ > b_) ? (a_ - b_) : (b_ - a_); \
    })
```

```c
static inline void compiler_barrier(void)
{
    asm volatile("" ::: "memory");
}
```

올바른 예(순수 ISO 표준 모드에서 포함될 공개 헤더):

```c
static inline void compiler_barrier(void)
{
    __asm__ volatile("" ::: "memory");
}

#define SWAP_VALUES(a, b)         \
    do {                          \
        __typeof__(a) tmp_ = (a); \
        (a) = (b);                \
        (b) = tmp_;               \
    } while (0)
```

### implementation-defined behavior

정수 폭, `char`의 부호, 엔디언, 정렬, 패딩 같은 implementation-defined behavior와 플랫폼별 내부 동작에 의존하지 않는다. 의존해야 한다면 주석으로 밝힌다. 다만 대상 플랫폼을 분명히 정한 코드는 그 플랫폼의 동작을 확실하게 가정해도 된다.

틀린 예:

```c
struct dns_header {
    uint16_t id;
    uint16_t flags;
    uint16_t question_count;
    uint16_t answer_count;
    uint16_t authority_count;
    uint16_t additional_count;
};

static void read_dns_header(struct dns_header *mut_header, const unsigned char *packet)
{
    memcpy(mut_header, packet, sizeof(*mut_header));
}
```

DNS 헤더의 필드는 big-endian으로 전송된다. 이 코드는 little-endian인 x86-64와 AArch64 Linux에서 필드마다 바이트 순서가 뒤집힌 값을 읽는다. 또 구조체에 패딩이 없다고 가정한다.

올바른 예:

```c
struct dns_header {
    uint16_t id;
    uint16_t flags;
    uint16_t question_count;
    uint16_t answer_count;
    uint16_t authority_count;
    uint16_t additional_count;
};

static uint16_t decode_be16(const unsigned char *bytes)
{
    return (uint16_t)(((unsigned int)bytes[0] << 8) | bytes[1]);
}

static void read_dns_header(struct dns_header *mut_header, const unsigned char *packet)
{
    mut_header->id = decode_be16(packet);
    mut_header->flags = decode_be16(packet + 2);
    mut_header->question_count = decode_be16(packet + 4);
    mut_header->answer_count = decode_be16(packet + 6);
    mut_header->authority_count = decode_be16(packet + 8);
    mut_header->additional_count = decode_be16(packet + 10);
}
```

올바른 예(의존하는 동작을 주석으로 밝힌 경우):

```c
static int save_counter_cache(FILE *mut_file, const uint64_t *counters, size_t count)
{
    /* written in host byte order; only the host that wrote the cache reads it back */
    if (fwrite(counters, sizeof(*counters), count, mut_file) != count) {
        return -1;
    }
    return 0;
}
```

올바른 예(데이터를 little-endian으로 보내는 센서의 값을 읽는, little-endian 아키텍처 기반 펌웨어 코드):

```c
struct sensor_sample {
    uint16_t temperature;
    uint16_t humidity;
};

static void read_sensor_sample(struct sensor_sample *mut_sample,
                               const unsigned char *bytes)
{
    memcpy(mut_sample, bytes, sizeof(*mut_sample));
}
```

### 표준 함수의 선택

C 표준 함수는 구현체, 문자열과 메모리 안전성, 성능의 trade-off를 판단해서 쓴다. 특히 문자열 함수가 그렇다. 배치 처리, 병렬 처리, 스레드 안전성 등을 확실하게 보장해야 하면 그 기능을 직접 작성한다.

예를 들어 `strtok`은 다음 탐색을 시작할 위치를 함수 안에 저장해 두고 다음 호출에서 이어 쓴다. C11 표준에 따르면 `strtok`은 다른 `strtok` 호출과의 data race를 피하지 않아도 된다(C11 §7.24.5.8 ¶6). 또 `strncat`은 호출할 때마다 대상 문자열의 끝을 처음부터 다시 찾는다. 따라서, 많은 문자열을 차례로 이어 붙이면 이어 붙인 길이의 제곱에 비례하는 시간이 든다.

틀린 예:

```c
struct field_job {
    char line[256];
    size_t field_count;
};

static void *count_fields(void *mut_arg)
{
    struct field_job *job = mut_arg;

    for (const char *field = strtok(job->line, ","); field != NULL; field = strtok(NULL, ",")) {
        ++job->field_count;
    }

    return NULL;
}
```

지양하는 예:

```c
static void join_words(char *mut_dst, size_t dst_size, const char *const *words,
                       size_t count)
{
    if (dst_size == 0) {
        return;
    }

    mut_dst[0] = '\0';

    for (size_t i = 0; i < count; ++i) {
        strncat(mut_dst, words[i], dst_size - strlen(mut_dst) - 1);
    }
}
```

올바른 예:

```c
struct field_job {
    char line[256];
    size_t field_count;
};

static char *take_field(char **mut_cursor)
{
    char *field = *mut_cursor + strspn(*mut_cursor, ",");

    if (*field == '\0') {
        return NULL;
    }

    char *end = field + strcspn(field, ",");
    if (*end != '\0') {
        *end = '\0';
        *mut_cursor = end + 1;
    } else {
        *mut_cursor = end;
    }

    return field;
}

static void *count_fields(void *mut_arg)
{
    struct field_job *job = mut_arg;
    char *cursor = job->line;

    for (const char *field = take_field(&cursor); field != NULL; field = take_field(&cursor)) {
        ++job->field_count;
    }

    return NULL;
}
```

```c
static void join_words(char *mut_dst, size_t dst_size, const char *const *words,
                       size_t count)
{
    if (dst_size == 0) {
        return;
    }

    size_t used = 0;

    for (size_t i = 0; i < count; ++i) {
        size_t length = strlen(words[i]);

        if (length > dst_size - used - 1) {
            length = dst_size - used - 1;
        }

        memcpy(mut_dst + used, words[i], length);
        used += length;
    }

    mut_dst[used] = '\0';
}
```

### 쓰지 않는 표준 함수

`gets`는 쓰지 않는다. 버퍼 크기를 받지 않아 버퍼 넘침을 막을 수 없다. 또 C11 표준에서 빠졌다.

틀린 예:

```c
static int read_command(char *mut_line)
{
    if (gets(mut_line) == NULL) {
        return -1;
    }

    return 0;
}
```

올바른 예:

```c
static int read_command(char *mut_line, size_t line_size)
{
    if (line_size < 2 || line_size > INT_MAX) {
        return -1;
    }

    if (fgets(mut_line, (int)line_size, stdin) == NULL) {
        return -1;
    }

    char *newline = strchr(mut_line, '\n');
    if (newline != NULL) {
        *newline = '\0';
        return 0;
    }

    int result = 0;

    for (int next = getchar(); next != '\n' && next != EOF; next = getchar()) {
        result = -1;
    }

    return result;
}
```

`fgets`는 개행 문자를 버퍼에 남긴다. 또 버퍼에 들어가지 않는 줄의 나머지는 스트림에 남긴다. 따라서, 올바른 예는 `gets`처럼 개행 문자를 지운다. 버퍼에 들어가지 않는 줄은 나머지를 건너뛰고 실패로 처리한다.

## 파일

### 파일 이름

C 소스 파일의 이름은 `.c`로, 헤더 파일의 이름은 `.h`로 끝난다. 이름은 소문자 snake_case로 짓는다. Windows의 예약 이름(`CON`, `PRN`, `AUX`, `NUL`, `COM1`, `LPT1` 등)은 확장자를 붙여도 쓰지 않는다.

틀린 예:

```text
text_buffer.C
word_count.hpp
```

```text
TextBuffer.c
wordCount.c
wordCount.h
```

```text
prn.c
```

올바른 예:

```text
text_buffer.c
word_count.h
```

```text
text_buffer.c
word_count.c
word_count.h
```

```text
printer.c
```

### 인코딩과 줄 끝

파일은 BOM 없는 UTF-8로 인코딩하고, 줄 끝에는 LF를 쓴다. Windows 전용 스크립트(`.bat`, `.ps1`)와 Visual Studio 프로젝트 파일(`.sln`, `.slnx`, `.vcxproj`, `.vcxproj.filters`, `.props`, `.targets`)에만 CRLF를 쓴다. `.gitattributes`와 `.editorconfig`로 이 규칙을 강제할 수 있다.

Git의 `core.autocrlf`는 쓰지 않는다. `core.autocrlf`는 사람마다 다르게 둘 수 있는 로컬 설정이다. 따라서, 이 설정으로는 프로젝트의 줄 끝을 고정할 수 없다. 줄 끝은 프로젝트에 함께 들어가는 `.gitattributes`로 정한다. `.gitattributes`에서 `eol`을 정한 파일은 `core.autocrlf` 값과 상관없이 그 줄 끝으로 checkout된다.

`text=auto`처럼 Git이 텍스트 여부를 스스로 판정하게 하는 설정도 쓰지 않는다. 자동 판정을 쓰면 Git이 파일 내용을 보고 텍스트 여부를 정한다. 따라서, 경로만 보고는 줄 끝을 알 수 없다. 경로마다 `text`와 `eol`을 직접 정하고, 바이너리 파일은 `binary`로 표시한다.

틀린 예:

```text
$ file word_count.c
word_count.c: C source, Unicode text, UTF-8 (with BOM) text, with CRLF line terminators
```

```sh
git config --global core.autocrlf true
```

```gitattributes
* text=auto eol=lf
*.sln text eol=crlf
*.slnx text eol=crlf
*.vcxproj text eol=crlf
*.vcxproj.filters text eol=crlf
*.props text eol=crlf
*.targets text eol=crlf
```

올바른 예:

```text
$ file word_count.c
word_count.c: C source, ASCII text
```

```gitattributes
* text eol=lf
*.bat text eol=crlf
*.ps1 text eol=crlf
*.sln text eol=crlf
*.slnx text eol=crlf
*.vcxproj text eol=crlf
*.vcxproj.filters text eol=crlf
*.props text eol=crlf
*.targets text eol=crlf
*.png binary
```

### 문자 집합

코드와 주석은 ASCII로 쓴다. 출력 문자열에는 뜻이 있는 기호가 필요할 때만 ASCII 밖의 문자를 쓴다. 메시지의 따옴표에는 ASCII `'`와 `"`를 쓴다.

틀린 예:

```c
/* returns the power of 1024 for unit (B → 0, KiB → 1, MiB → 2), or -1 if unknown */
int parse_unit(const char *unit);
```

```c
static void report_unknown_unit(const char *program_name, const char *unit)
{
    fprintf(stderr, "%s: unknown unit ‘%s’\n", program_name, unit); // [!hl]
}
```

올바른 예:

```c
/* returns the power of 1024 for unit (B -> 0, KiB -> 1, MiB -> 2), or -1 if unknown */
int parse_unit(const char *unit);
```

```c
static void report_unknown_unit(const char *program_name, const char *unit)
{
    fprintf(stderr, "%s: unknown unit '%s'\n", program_name, unit); // [!hl]
}
```

올바른 예(뜻이 있는 기호를 출력하는 경우):

```c
static void print_temperature(double celsius)
{
    printf("%.1f °C\n", celsius);
}
```

### 마지막 개행

모든 파일은 개행 문자로 끝난다.

C 소스 파일에는 C 표준이 이를 요구한다. 기본 표준인 C99의 §5.1.1.2 ¶1, translation phase 2에 다음 문장이 있다.

> A source file that is not empty shall end in a new-line character, which shall not be immediately preceded by a backslash character before any such splicing takes place.

이 요구를 어기면 C99 §4 ¶2에 따라 undefined behavior가 된다. C99 Annex J.2의 undefined behavior 목록에도 이 경우가 있다. C11과 C23에도 같은 문장이 있다.

틀린 예:

```text
$ tail -c 2 word_count.c | od -c
0000000  \n   }
0000002
```

올바른 예:

```text
$ tail -c 2 word_count.c | od -c
0000000   }  \n
0000002
```

### 공백 문자

줄 끝에 공백을 두지 않는다. 빈 줄에는 공백이나 탭을 남기지 않는다. 탭 문자는 Makefile 레시피에만 쓴다. form feed(`^L`)도 쓰지 않는다.

틀린 예:

```text
$ cat -A word_count.c
static int is_all_spaces(const char *text)  $
{$
^Ifor (size_t i = 0; text[i] != '\0'; ++i) {$
^I^Iif (text[i] != ' ') {$
^I^I^Ireturn 0;$
^I^I}$
^I}$
    $
^Ireturn 1;$
}$
^L$
```

올바른 예:

```text
$ cat -A word_count.c
static int is_all_spaces(const char *text)$
{$
    for (size_t i = 0; text[i] != '\0'; ++i) {$
        if (text[i] != ' ') {$
            return 0;$
        }$
    }$
$
    return 1;$
}$
```

```text
$ cat -A Makefile
word_count: word_count.o$
^I$(CC) $(LDFLAGS) -o $@ word_count.o$
```

### 파일 안의 배치 순서

소스 파일은 다음 순서로 쓴다. 묶음 사이에는 빈 줄을 하나 둔다.

1. feature test macro
2. `#include`
3. 매크로
4. 타입(struct, union, enum)
5. file scope 변수
6. static 함수
7. external linkage 함수
8. `main`

올바른 예(`scoreboard.h`):

```c
#ifndef SCOREBOARD_H
#define SCOREBOARD_H

void reset_scores(void);

#endif /* SCOREBOARD_H */
```

올바른 예(`scoreboard.c`):

```c
#define _POSIX_C_SOURCE 200809L

#include "scoreboard.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_PLAYERS 8
#define MAX_NAME_LEN 16

struct player {
    char name[MAX_NAME_LEN];
    int score;
};

static struct player gs_players[MAX_PLAYERS];
static size_t gs_player_count;

static int add_player(const char *name, int score)
{
    if (gs_player_count == MAX_PLAYERS) {
        return -1;
    }

    struct player *player = &gs_players[gs_player_count];
    size_t length = strnlen(name, MAX_NAME_LEN - 1);

    memcpy(player->name, name, length);
    player->name[length] = '\0';
    player->score = score;
    ++gs_player_count;
    return 0;
}

void reset_scores(void)
{
    for (size_t i = 0; i < gs_player_count; ++i) {
        gs_players[i].score = 0;
    }
}

int main(void)
{
    if (add_player("ada", 7) != 0 || add_player("grace", 9) != 0) {
        return EXIT_FAILURE;
    }

    reset_scores();

    for (size_t i = 0; i < gs_player_count; ++i) {
        printf("%s: %d\n", gs_players[i].name, gs_players[i].score);
    }

    return 0;
}
```

헤더와 소스를 나눈 프로젝트에서는 구조상 이 순서를 지킬 수 없을 때만 예외를 허용한다.

### feature test macro

POSIX나 glibc 기능을 쓰는 파일에서는 feature test macro로 그 기능을 켠다. 시스템 헤더가 보여 주는 선언은 feature test macro에 따라 달라진다. 이 매크로는 소스 파일에서 어떤 `#include`보다도 앞에 정의하고, 그 아래에 빈 줄을 하나 둔다. 헤더 안에서는 정의하지 않는다. 그 헤더보다 먼저 include된 헤더에는 효과가 없기 때문이다.

필요한 기능에 맞는 가장 좁은 매크로를 고른다. 표준 C 기능만 쓰는 파일에는 정의하지 않는다.

- `_POSIX_C_SOURCE`: POSIX 인터페이스를 쓸 때 정의한다. 기본값으로 POSIX.1-2008의 `200809L`을 쓴다. POSIX.1-2024의 값은 `202405L`이다.
- `_XOPEN_SOURCE`: POSIX의 XSI 확장까지 쓸 때 정의한다. POSIX.1-2008의 값은 `700`, POSIX.1-2024의 값은 `800`이다.
- `_DEFAULT_SOURCE`: glibc의 기본 기능 집합을 쓸 때 정의한다. 이 집합에는 그 glibc가 지원하는 가장 높은 수준의 POSIX 기능과 BSD, SVID 기능 일부가 들어 있다. 다른 feature test macro를 정의하거나 `-std=c11` 같은 ISO 표준 모드로 컴파일하면 glibc는 이 집합을 끈다.
- `_GNU_SOURCE`: 정의하면 glibc의 모든 기능과 GNU 확장이 켜진다. `_POSIX_C_SOURCE`와 `_XOPEN_SOURCE`도 그 glibc가 지원하는 가장 높은 값으로 함께 정의된다. glibc 2.44에서는 `202405L`과 `800`이다. Linux 전용 파일에서만 쓴다.
- `WIN32_LEAN_AND_MEAN`, `_WIN32_WINNT`: feature test macro는 아니지만 같은 자리에 둔다. `WIN32_LEAN_AND_MEAN`을 정의하면 `<windows.h>`에서 Cryptography, DDE, RPC, Shell, Windows Sockets 같은 API가 빠진다. `_WIN32_WINNT`는 지원할 가장 낮은 Windows 버전으로 정의한다. Windows 10의 값은 `0x0A00`이다.

틀린 예:

```c
#include <stdio.h>
#include <stdlib.h>

static int count_lines(size_t *mut_count, FILE *mut_input)
{
    char *line = NULL;
    size_t capacity = 0;
    size_t count = 0;

    while (getline(&line, &capacity, mut_input) != -1) {
        ++count;
    }

    free(line);
    line = NULL;
    if (ferror(mut_input) != 0) {
        return -1;
    }

    *mut_count = count;
    return 0;
}
```

```c
#include <stdio.h>
#define _POSIX_C_SOURCE 200809L // [!hl]
#include <stdlib.h>

static int count_lines(size_t *mut_count, FILE *mut_input)
{
    char *line = NULL;
    size_t capacity = 0;
    size_t count = 0;

    while (getline(&line, &capacity, mut_input) != -1) {
        ++count;
    }

    free(line);
    line = NULL;
    if (ferror(mut_input) != 0) {
        return -1;
    }

    *mut_count = count;
    return 0;
}
```

```c
#ifndef TEXT_LINES_H
#define TEXT_LINES_H

#define _POSIX_C_SOURCE 200809L

#include <stdio.h>

int count_lines(size_t *mut_count, FILE *mut_input);

#endif /* TEXT_LINES_H */
```

```c
#define _GNU_SOURCE // [!hl]

#include <stdio.h>
#include <stdlib.h>

static int count_lines(size_t *mut_count, FILE *mut_input)
{
    char *line = NULL;
    size_t capacity = 0;
    size_t count = 0;

    while (getline(&line, &capacity, mut_input) != -1) {
        ++count;
    }

    free(line);
    line = NULL;
    if (ferror(mut_input) != 0) {
        return -1;
    }

    *mut_count = count;
    return 0;
}
```

```c
#define _POSIX_C_SOURCE 200809L

#include <stdio.h>

static void print_total(long total)
{
    printf("total: %ld\n", total);
}
```

올바른 예(POSIX 인터페이스를 쓰는 경우):

```c
#define _POSIX_C_SOURCE 200809L // [!hl]

#include <stdio.h>
#include <stdlib.h>

static int count_lines(size_t *mut_count, FILE *mut_input)
{
    char *line = NULL;
    size_t capacity = 0;
    size_t count = 0;

    while (getline(&line, &capacity, mut_input) != -1) {
        ++count;
    }

    free(line);
    line = NULL;
    if (ferror(mut_input) != 0) {
        return -1;
    }

    *mut_count = count;
    return 0;
}
```

올바른 예(`text_lines.h`):

```c
#ifndef TEXT_LINES_H
#define TEXT_LINES_H

#include <stdio.h>

int count_lines(size_t *mut_count, FILE *mut_input);

#endif /* TEXT_LINES_H */
```

올바른 예(XSI 확장을 쓰는 경우):

```c
#define _XOPEN_SOURCE 700

#include <time.h>

static int parse_date(struct tm *mut_date, const char *text)
{
    struct tm date = { 0 };
    const char *end = strptime(text, "%Y-%m-%d", &date);

    if (end == NULL || *end != '\0') {
        return -1;
    }

    *mut_date = date;
    return 0;
}
```

올바른 예(glibc의 기본 기능을 쓰는 경우):

```c
#define _DEFAULT_SOURCE

#include <stdlib.h>

static int grow_scores(int **mut_scores, size_t count)
{
    if (count == 0) {
        return -1;
    }

    int *scores = reallocarray(*mut_scores, count, sizeof(**mut_scores));
    if (scores == NULL) {
        return -1;
    }

    *mut_scores = scores;
    return 0;
}
```

올바른 예(GNU 확장을 쓰는 Linux 전용 파일):

```c
#define _GNU_SOURCE

#include <string.h>

static size_t measure_key(const char *entry)
{
    return (size_t)(strchrnul(entry, '=') - entry);
}
```

올바른 예(표준 C 기능만 쓰는 경우):

```c
#include <stdio.h>

static void print_total(long total)
{
    printf("total: %ld\n", total);
}
```

올바른 예(Windows):

```c
#define WIN32_LEAN_AND_MEAN
#define _WIN32_WINNT 0x0A00

#include <windows.h>
```

### include 순서

`#include`는 세 가지 유형으로 나눈다. 각 유형 안에서는 알파벳 순서로 정렬한다. 다만 include 순서에 따른 종속성이 있는 헤더끼리는 그 순서를 따른다. 유형 사이에는 빈 줄을 하나 둔다.

1. main header: 그 소스 파일이 구현하는 헤더(`foo.c`의 `"foo.h"`)
2. `<...>` 형식: 표준 헤더, 시스템 헤더, 외부 라이브러리 헤더
3. `"..."` 형식: 프로젝트 안의 헤더

종속성이 있는 헤더는 같은 묶음의 맨 뒤에 두어, 그 헤더에 필요한 헤더보다 뒤에 오게 한다. 예를 들어 GNU Readline의 `<readline/readline.h>`는 `FILE`을 쓰면서 `<stdio.h>`를 스스로 include하지 않는다. 알파벳 순서로는 `<stdio.h>`보다 앞에 오므로 그대로 두면 컴파일 오류가 난다.

틀린 예:

```c
#include <string.h>
#include "util/text.h"
#include <stdio.h>
#include "report/summary.h"
#include <stdlib.h>
#include "store/inventory.h"
```

```c
#include <readline/readline.h>
#include <stdio.h>
#include <stdlib.h>
```

올바른 예(`report/summary.c`의 include):

```c
#include "report/summary.h"

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include "store/inventory.h"
#include "util/text.h"
```

올바른 예:

```c
#include <stdio.h>
#include <stdlib.h>
#include <readline/readline.h>
```

### include guard

모든 헤더는 include guard로 감싼다. 가드 이름은 프로젝트 루트에서 본 경로를 대문자로 바꾸고, 영문자와 숫자가 아닌 문자를 `_`로 바꿔 만든다. `#pragma once`는 쓰지 않는다.

`#endif` 뒤에는 반드시 가드 이름을 주석으로 단다. 헤더가 길면 `#endif`가 어느 `#ifndef`의 짝인지 바로 보이지 않기 때문이다.

가드 이름 맨 앞에 `_`를 붙이지 않는다. `_`로 시작하고 바로 뒤에 대문자가 오는 이름은 C 표준이 implementation에 예약한 이름이다. 시스템 헤더도 이런 이름을 자주 쓴다. 실제로 glibc의 `<net/if.h>`는 `_NET_IF_H`를 가드로 쓴다. 프로젝트에 `net/if.h`라는 헤더가 있어서 같은 이름을 가드로 쓰면, 두 헤더 가운데 나중에 include한 쪽의 내용이 빠진다. 컴파일러는 이 문제를 오류나 경고로 알리지 않는다.

틀린 예(`util/text.h`):

```c
#ifndef TEXT_H
#define TEXT_H

#include <stddef.h>

size_t count_words(const char *text);

#endif /* TEXT_H */
```

```c
#pragma once

#include <stddef.h>

size_t count_words(const char *text);
```

```c
#ifndef UTIL_TEXT_H
#define UTIL_TEXT_H

#include <stddef.h>

size_t count_words(const char *text);

#endif // [!hl]
```

```c
#ifndef _UTIL_TEXT_H
#define _UTIL_TEXT_H

#include <stddef.h>

size_t count_words(const char *text);

#endif /* _UTIL_TEXT_H */
```

올바른 예(`util/text.h`):

```c
#ifndef UTIL_TEXT_H
#define UTIL_TEXT_H

#include <stddef.h>

size_t count_words(const char *text);

#endif /* UTIL_TEXT_H */ // [!hl]
```

### 헤더의 내용

헤더에는 선언, 타입 정의, 매크로, inline 함수만 둔다. 변수와 그 밖의 함수는 `.c` 파일 하나에 정의한다. 여러 파일에서 쓰는 변수는 헤더에 `extern`으로 선언하고, `.c` 파일 하나에서 정의하며 초기화한다. 함수 선언에는 `extern`을 붙이지 않는다.

다음 정의는 헤더에 두지 않는다.

- 변수의 정의: 헤더를 include하는 translation unit마다 정의가 생겨 링크할 때 `multiple definition` 오류가 난다.
- `static` 변수: 헤더를 include하는 translation unit마다 별개의 변수가 생긴다. 한 파일에서 바꾼 값이 다른 파일에는 보이지 않는다.
- `static` 함수: translation unit마다 사본이 생기는 점은 `static inline` 함수와 같다. 다만 그 함수를 쓰지 않는 파일마다 GCC와 Clang이 `-Wunused-function` 경고를 낸다. `static inline` 함수에는 이 경고를 내지 않는다.

틀린 예(`store/inventory.h`):

```c
#ifndef STORE_INVENTORY_H
#define STORE_INVENTORY_H

#include <stddef.h>

#define MAX_ITEM_NAME_LEN 32

struct item {
    char name[MAX_ITEM_NAME_LEN];
    size_t quantity;
};

size_t g_low_stock_limit = 5;
static size_t gs_item_count;

static int is_low_stock(const struct item *item)
{
    return item->quantity < g_low_stock_limit;
}

extern int add_item(const char *name, size_t quantity);

#endif /* STORE_INVENTORY_H */
```

올바른 예(`store/inventory.h`):

```c
#ifndef STORE_INVENTORY_H
#define STORE_INVENTORY_H

#include <stddef.h>

#define MAX_ITEM_NAME_LEN 32

struct item {
    char name[MAX_ITEM_NAME_LEN];
    size_t quantity;
};

extern size_t g_low_stock_limit;

static inline int is_low_stock(const struct item *item)
{
    return item->quantity < g_low_stock_limit;
}

int add_item(const char *name, size_t quantity);

#endif /* STORE_INVENTORY_H */
```

올바른 예(`store/inventory.c`):

```c
#include "store/inventory.h"

#include <string.h>

#define MAX_ITEMS 64

size_t g_low_stock_limit = 5;

static struct item gs_items[MAX_ITEMS];
static size_t gs_item_count;

int add_item(const char *name, size_t quantity)
{
    size_t length = strlen(name);
    if (gs_item_count == MAX_ITEMS || length >= MAX_ITEM_NAME_LEN) {
        return -1;
    }

    struct item *item = &gs_items[gs_item_count];

    memcpy(item->name, name, length + 1);
    item->quantity = quantity;
    ++gs_item_count;
    return 0;
}
```

## 서식

### 들여쓰기

들여쓰기는 공백 4칸이다. 탭으로 들여쓰지 않는다.

틀린 예:

```c
static const char *describe_temperature(int celsius)
{
  if (celsius < 0) {
    return "freezing";
  }
  if (celsius < 25) {
    return "mild";
  }
  return "hot";
}
```

올바른 예:

```c
static const char *describe_temperature(int celsius)
{
    if (celsius < 0) {
        return "freezing";
    }
    if (celsius < 25) {
        return "mild";
    }
    return "hot";
}
```

### 줄 길이

한 줄은 100열을 넘기지 않는 것이 원칙이다. 다만 넘치는 부분이 짧으면 넘겨도 된다.

틀린 예:

```c
static void print_order_summary(const char *customer, size_t item_count, unsigned long total_cents, unsigned int delivery_days)
{
    printf("order for %s: %zu items, total %lu.%02lu, delivery in %u business days\n", customer, item_count, total_cents / 100, total_cents % 100, delivery_days);
}
```

올바른 예:

```c
static void print_order_summary(const char *customer, size_t item_count,
                                unsigned long total_cents, unsigned int delivery_days)
{
    printf("order for %s: %zu items, total %lu.%02lu, delivery in %u business days\n", customer,
           item_count, total_cents / 100, total_cents % 100, delivery_days);
}
```

### 긴 식 나누기

줄을 나눌 때는 다음 모양을 따른다.

- 소괄호 안에서 나누면, 다음 줄은 아직 닫히지 않은 소괄호 가운데 가장 안쪽 소괄호의 바로 다음 열에서 시작한다. 즉, 다음 줄의 첫 글자는 그 소괄호 바로 뒤의 글자와 같은 열에 놓인다. 함수 호출의 인자, 함수 선언의 매개변수, `if`와 `while`의 조건식이 모두 그렇다.
- 소괄호 밖의 식을 연산자 뒤에서 나누면, 다음 줄은 그 식의 첫 피연산자와 같은 열에서 시작한다.
- 할당문은 할당 연산자 뒤에서 나누지 않고, 오른쪽 식의 소괄호 안에서 나누는 것을 기본으로 한다. 오른쪽 식이 매우 길어 소괄호 안에서 나누면 다음 줄이 들어갈 자리가 남지 않을 때만 할당 연산자 바로 뒤에서 나눈다. 이때 다음 줄은 문장의 첫 줄보다 4칸 더 들여쓴다.
- 이항 연산자와 쉼표는 줄 끝에 남긴다.

틀린 예:

```c
static int format_location(char *mut_buffer, size_t buffer_size, const char *file_name,
                           int line_number, int column_number)
{
    int written = snprintf(mut_buffer, buffer_size, "%s:%d:%d: unexpected token\n", file_name,
        line_number, column_number); // [!hl]

    return written;
}
```

```c
static int format_location(char *mut_buffer, size_t buffer_size, const char *file_name,
                           int line_number, int column_number)
{
    int written =
        snprintf(mut_buffer, buffer_size, "%s:%d:%d: unexpected token\n", file_name, line_number,
                 column_number);

    return written;
}
```

```c
static size_t sum_section_bytes(size_t metadata_bytes, size_t thumbnail_bytes, size_t pixel_bytes,
                                size_t color_profile_bytes, size_t annotation_bytes,
                                size_t checksum_bytes)
{
    size_t total = metadata_bytes + thumbnail_bytes + pixel_bytes + color_profile_bytes
                   + annotation_bytes + checksum_bytes;

    return total;
}
```

올바른 예:

```c
static int format_location(char *mut_buffer, size_t buffer_size, const char *file_name,
                           int line_number, int column_number)
{
    int written = snprintf(mut_buffer, buffer_size, "%s:%d:%d: unexpected token\n", file_name,
                           line_number, column_number); // [!hl]

    return written;
}
```

```c
static size_t sum_section_bytes(size_t metadata_bytes, size_t thumbnail_bytes, size_t pixel_bytes,
                                size_t color_profile_bytes, size_t annotation_bytes,
                                size_t checksum_bytes)
{
    size_t total = metadata_bytes + thumbnail_bytes + pixel_bytes + color_profile_bytes +
                   annotation_bytes + checksum_bytes;

    return total;
}
```

```c
int is_supported_format(int format, int bits_per_pixel);

static int can_display(int width, int height, int format, int bits_per_pixel, int max_width,
                       int max_height)
{
    if (width <= max_width && height <= max_height &&
        is_supported_format(format, bits_per_pixel) != 0) {
        return 1;
    }

    return 0;
}
```

오른쪽 식이 매우 길 때의 올바른 예:

```c
unsigned long estimate_remaining_conversion_time_ms(unsigned long first_job, unsigned long last_job,
                                                    size_t worker_count);

static unsigned long report_remaining_time(unsigned long first_job, unsigned long last_job,
                                           size_t worker_count)
{
    unsigned long remaining_conversion_time_ms =
        estimate_remaining_conversion_time_ms(first_job, last_job, worker_count);

    return remaining_conversion_time_ms;
}
```

### 사용자에게 보이는 문자열

출력 메시지 문자열은 줄 길이 제한을 넘어도 나누지 않는다. grep 같은 문자열 탐색 도구로 메시지를 찾을 수 있어야 하기 때문이다. 다만 여러 줄을 출력하는 문자열은 출력 줄 단위로 나눠 이어 붙여도 된다.

틀린 예:

```c
static void report_missing_value(const char *key, int line_number)
{
    fprintf(stderr, "settings: key '%s' on line %d has no value, "
                    "expected a line of the form key=value\n",
            key, line_number);
}
```

올바른 예:

```c
static void report_missing_value(const char *key, int line_number)
{
    fprintf(stderr, "settings: key '%s' on line %d has no value, expected a line of the form key=value\n",
            key, line_number);
}
```

```c
static void show_help(const char *program_name)
{
    printf("usage: %s [options] <file>\n"
           "  -c  count characters\n"
           "  -w  count words\n",
           program_name);
}
```

### 함수 선언부

반환형, 함수 이름, 매개변수 목록을 한 줄에 쓴다. 그 줄이 길면 매개변수 목록 안에서 나눈다.

틀린 예:

```c
static int
clamp_channel(int value)
{
    if (value < 0) {
        return 0;
    }
    if (value > 255) {
        return 255;
    }
    return value;
}
```

```c
static void blend_image_rows(uint8_t *mut_dst, const uint8_t *src, size_t pixel_count, uint8_t alpha)
{
    for (size_t i = 0; i < pixel_count; ++i) {
        mut_dst[i] = (uint8_t)((src[i] * alpha + mut_dst[i] * (255U - alpha)) / 255U);
    }
}
```

올바른 예:

```c
static int clamp_channel(int value)
{
    if (value < 0) {
        return 0;
    }
    if (value > 255) {
        return 255;
    }
    return value;
}
```

```c
static void blend_image_rows(uint8_t *mut_dst, const uint8_t *src,
                             size_t pixel_count, uint8_t alpha)
{
    for (size_t i = 0; i < pixel_count; ++i) {
        mut_dst[i] = (uint8_t)((src[i] * alpha + mut_dst[i] * (255U - alpha)) / 255U);
    }
}
```

### 함수 중괄호

함수 본문의 여는 중괄호와 닫는 중괄호는 각각 새 줄에 둔다. 들여쓰기는 함수 선언부와 같다.

틀린 예:

```c
static int is_leap_year(int year) {
    return (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
}
```

올바른 예:

```c
static int is_leap_year(int year)
{
    return (year % 4 == 0 && year % 100 != 0) || year % 400 == 0;
}
```

### 제어문과 반복문 중괄호

`if`, `else`, `for`, `while`, `do`, `switch`의 여는 중괄호는 같은 줄 끝에 둔다. 닫는 중괄호는 새 줄에 둔다. 같은 문장이 이어지면 닫는 중괄호 뒤에 붙여 쓴다(`} else {`, `} while (...);`).

다만 Windows 전용 프로젝트에서는 Windows 관례대로 여는 중괄호를 새 줄에 두는 Allman 스타일을 쓴다.

틀린 예:

```c
static int adjust_level(int level, const unsigned char *history, size_t count)
{
    if (level < 0)
    {
        level = 0;
    }
    else if (level > 100)
    {
        level = 100;
    }
    else
    {
        level += 1;
    }

    for (size_t i = 0; i < count && level <= 100; ++i)
    {
        level += history[i];
    }

    while (level > 100)
    {
        level -= 10;
    }

    do
    {
        level += 2;
    }
    while (level % 5 != 0);

    switch (level / 25)
    {
    case 0:
        level = 25;
        break;
    default:
        break;
    }

    return level;
}
```

올바른 예:

```c
static int adjust_level(int level, const unsigned char *history, size_t count)
{
    if (level < 0) {
        level = 0;
    } else if (level > 100) {
        level = 100;
    } else {
        level += 1;
    }

    for (size_t i = 0; i < count && level <= 100; ++i) {
        level += history[i];
    }

    while (level > 100) {
        level -= 10;
    }

    do {
        level += 2;
    } while (level % 5 != 0);

    switch (level / 25) {
    case 0:
        level = 25;
        break;
    default:
        break;
    }

    return level;
}
```

### 중괄호 생략 금지

모든 제어문의 본문은 반드시 중괄호로 감싼다. 대상은 `if`, `else`, `for`, `while`, `do`이다. 본문이 한 문장뿐이어도 감싼다.

중괄호를 생략하면 본문에 문장을 더할 때 오동작하기 쉽다. 예를 들어 `if (cond) do_something();`의 본문에 `do_after();`를 더하면서 같은 들여쓰기로 두면, 두 문장이 모두 `if`에 걸린 것처럼 보인다. 그러나 `if`에 걸리는 것은 `do_something();` 하나뿐이고, `do_after();`는 조건과 상관없이 늘 실행된다. 중괄호를 늘 쓰는 것은 이런 오동작을 막기 위해서이다.

틀린 예:

```c
void stop_motor(void);
void log_event(const char *message);

static void check_motor(int temperature, int limit)
{
    if (temperature > limit)
        stop_motor();
        log_event("motor stopped");
}
```

```c
static long long compute_absolute_value(int value)
{
    if (value < 0)
        return -(long long)value;
    else
        return value;
}

static long long sum_values(const int *values, size_t count)
{
    long long sum = 0;

    for (size_t i = 0; i < count; ++i)
        sum += values[i];

    return sum;
}

static size_t skip_spaces(const char *text)
{
    size_t i = 0;

    while (text[i] == ' ')
        ++i;

    return i;
}

static unsigned long long find_next_power_of_ten(unsigned int value)
{
    unsigned long long scale = 1;

    do
        scale *= 10;
    while (scale <= value);

    return scale;
}
```

올바른 예:

```c
void stop_motor(void);
void log_event(const char *message);

static void check_motor(int temperature, int limit)
{
    if (temperature > limit) {
        stop_motor();
        log_event("motor stopped");
    }
}
```

```c
static long long compute_absolute_value(int value)
{
    if (value < 0) {
        return -(long long)value;
    } else {
        return value;
    }
}

static long long sum_values(const int *values, size_t count)
{
    long long sum = 0;

    for (size_t i = 0; i < count; ++i) {
        sum += values[i];
    }

    return sum;
}

static size_t skip_spaces(const char *text)
{
    size_t i = 0;

    while (text[i] == ' ') {
        ++i;
    }

    return i;
}

static unsigned long long find_next_power_of_ten(unsigned int value)
{
    unsigned long long scale = 1;

    do {
        scale *= 10;
    } while (scale <= value);

    return scale;
}
```

### struct, union, enum 중괄호

struct, union, enum의 여는 중괄호는 같은 줄에 둔다.

틀린 예:

```c
enum traffic_light
{
    TRAFFIC_LIGHT_RED = 0,
    TRAFFIC_LIGHT_YELLOW,
    TRAFFIC_LIGHT_GREEN,
};

struct rgb_color
{
    uint8_t red;
    uint8_t green;
    uint8_t blue;
};

union float_bits
{
    float value;
    uint32_t bits;
};
```

올바른 예:

```c
enum traffic_light {
    TRAFFIC_LIGHT_RED = 0,
    TRAFFIC_LIGHT_YELLOW,
    TRAFFIC_LIGHT_GREEN,
};

struct rgb_color {
    uint8_t red;
    uint8_t green;
    uint8_t blue;
};

union float_bits {
    float value;
    uint32_t bits;
};
```

### 범위를 좁히는 블록

범위를 좁히려고 만든 블록은 최대한 지양한다. 설령 쓰더라도 여는 중괄호와 닫는 중괄호를 각각 다른 코드 없이 한 줄에 둔다.

틀린 예:

```c
static void print_totals(long online_sales, long store_sales, long returns)
{
    { long sold = online_sales + store_sales;
        printf("sold: %ld\n", sold); }

    printf("returned: %ld\n", returns);
}
```

올바른 예:

```c
static void print_totals(long online_sales, long store_sales, long returns)
{
    {
        long sold = online_sales + store_sales;

        printf("sold: %ld\n", sold);
    }

    printf("returned: %ld\n", returns);
}
```

### 중괄호를 이용한 초기화

중괄호 초기화 목록은 중괄호 안쪽에 공백을 둔다. 마지막 원소 뒤의 쉼표는 원소가 더 늘어날 여지가 있는 목록에만 둔다. enum이나 표처럼 원소가 늘어날 목록은 원소마다 줄을 바꾸고 마지막 원소 뒤에도 쉼표를 둔다. 그러면 원소를 더하거나 뺄 때 diff가 한 줄로 끝난다.

구조체를 designated initializer로 초기화할 때는 멤버마다 줄을 바꾸는 것을 기본으로 한다.

틀린 예:

```c
static const int gs_first_primes[] = {2, 3, 5, 7};
```

```c
enum image_format { IMAGE_FORMAT_PNG = 0, IMAGE_FORMAT_JPEG, IMAGE_FORMAT_GIF };

static const char *const gs_image_extensions[] = { ".png", ".jpg", ".gif" };
```

```c
struct window_size {
    int width;
    int height;
};

static const struct window_size gs_default_size = { .width = 800, .height = 600 };
```

올바른 예:

```c
static const int gs_first_primes[] = { 2, 3, 5, 7 };
```

```c
enum image_format {
    IMAGE_FORMAT_PNG = 0,
    IMAGE_FORMAT_JPEG,
    IMAGE_FORMAT_GIF,
};

static const char *const gs_image_extensions[] = {
    ".png",
    ".jpg",
    ".gif",
};
```

```c
struct window_size {
    int width;
    int height;
};

static const struct window_size gs_default_size = {
    .width = 800,
    .height = 600
};
```

### switch와 case

`case`와 `default` 레이블은 `switch`와 같은 열에 둔다. `case` 본문은 한 줄에 몰아 쓰지 않는다.

틀린 예:

```c
static int convert_grade(char grade)
{
    int points = 0;

    switch (grade) {
        case 'A':
            points = 4;
            break;
        case 'B':
            points = 3;
            break;
        default:
            points = 0;
            break;
    }

    return points;
}
```

```c
static int convert_grade(char grade)
{
    int points = 0;

    switch (grade) {
    case 'A': points = 4; break;
    case 'B': points = 3; break;
    default: points = 0; break;
    }

    return points;
}
```

올바른 예:

```c
static int convert_grade(char grade)
{
    int points = 0;

    switch (grade) {
    case 'A':
        points = 4;
        break;
    case 'B':
        points = 3;
        break;
    default:
        points = 0;
        break;
    }

    return points;
}
```

### goto 레이블

goto 레이블은 감싸는 블록의 중괄호와 같은 열에 둔다. 따라서, 함수 본문 바로 안에 있는 레이블은 줄의 맨 앞에 온다.

틀린 예:

```c
struct session {
    FILE *log;
    char *buffer;
    int *scores;
};

static int open_session(struct session *mut_session, const char *log_path)
{
    mut_session->log = fopen(log_path, "a");
    if (mut_session->log == NULL) {
        return -1;
    }

    mut_session->buffer = malloc(4096);
    if (mut_session->buffer == NULL) {
        goto handle_buffer_alloc;
    }

    mut_session->scores = calloc(64, sizeof(*mut_session->scores));
    if (mut_session->scores == NULL) {
        goto handle_scores_alloc;
    }

    return 0;

    handle_scores_alloc:
    free(mut_session->buffer);
    mut_session->buffer = NULL;
    handle_buffer_alloc:
    fclose(mut_session->log);
    mut_session->log = NULL;
    return -1;
}
```

올바른 예:

```c
struct session {
    FILE *log;
    char *buffer;
    int *scores;
};

static int open_session(struct session *mut_session, const char *log_path)
{
    mut_session->log = fopen(log_path, "a");
    if (mut_session->log == NULL) {
        return -1;
    }

    mut_session->buffer = malloc(4096);
    if (mut_session->buffer == NULL) {
        goto handle_buffer_alloc;
    }

    mut_session->scores = calloc(64, sizeof(*mut_session->scores));
    if (mut_session->scores == NULL) {
        goto handle_scores_alloc;
    }

    return 0;

handle_scores_alloc:
    free(mut_session->buffer);
    mut_session->buffer = NULL;
handle_buffer_alloc:
    fclose(mut_session->log);
    mut_session->log = NULL;
    return -1;
}
```

```c
static size_t count_rows_without_zero(const int *cells, size_t row_count, size_t column_count)
{
    size_t count = 0;

    for (size_t row = 0; row < row_count; ++row) {
        for (size_t column = 0; column < column_count; ++column) {
            if (cells[row * column_count + column] == 0) {
                goto next_row;
            }
        }

        ++count;
    next_row:
        continue;
    }

    return count;
}
```

### 공백

- `if`, `for`, `while`, `switch`, `return` 같은 키워드 뒤에는 공백을 둔다.
- 함수 이름과 `(` 사이, `sizeof`와 `(` 사이에는 공백을 두지 않는다.
- 이항 연산자와 삼항 연산자의 양옆에는 공백을 둔다.
- 단항 연산자와 증감 연산자는 피연산자에 붙여 쓴다. `.`, `->`의 양옆에도 공백을 두지 않는다.
- 쉼표 뒤에는 공백을 두고, 앞에는 두지 않는다.
- 괄호 안쪽과 캐스트 뒤에는 공백을 두지 않는다.

틀린 예:

```c
struct point {
    short x;
    short y;
};

static int sum_distances(const struct point *points, size_t count, bool is_closed)
{
    long long total = 0;

    for(size_t i = 1; i<count; ++ i) {
        total+=abs (points[i] . x-points[i - 1] . x)+abs( points[i].y - points[i - 1].y );
    }

    if(! is_closed && count>0) {
        const struct point *last = & points[count - 1];

        total -= abs(last -> x - points[0].x);
    }

    while(total > INT_MAX / 2) {
        total /= 2;
    }

    switch(count) {
    case 0:
        -- total;
        break;
    default:
        break;
    }

    int unit = (sizeof (*points)*count>4096)?2:1;

    return(int) (total / unit);
}
```

올바른 예:

```c
struct point {
    short x;
    short y;
};

static int sum_distances(const struct point *points, size_t count, bool is_closed)
{
    long long total = 0;

    for (size_t i = 1; i < count; ++i) {
        total += abs(points[i].x - points[i - 1].x) + abs(points[i].y - points[i - 1].y);
    }

    if (!is_closed && count > 0) {
        const struct point *last = &points[count - 1];

        total -= abs(last->x - points[0].x);
    }

    while (total > INT_MAX / 2) {
        total /= 2;
    }

    switch (count) {
    case 0:
        --total;
        break;
    default:
        break;
    }

    int unit = (sizeof(*points) * count > 4096) ? 2 : 1;

    return (int)(total / unit);
}
```

### 포인터 표기

포인터를 선언할 때 `*`는 이름 쪽에 붙인다. 포인터를 돌려주는 함수도 같다. 다만 Windows 전용 프로젝트에서는 Windows 관례대로 `*`를 타입 쪽에 붙인다.

틀린 예:

```c
struct menu;

const char* find_label(const struct menu* menu, int id);
char* copy_label(const char* label);
```

올바른 예:

```c
struct menu;

const char *find_label(const struct menu *menu, int id);
char *copy_label(const char *label);
```

### 한 줄에는 가능한 한 문장만

특별한 이유가 없는 한, 한 줄에는 문장 하나만 쓴다.

틀린 예:

```c
struct size {
    int width;
    int height;
};

static void reset_size(struct size *mut_size)
{
    mut_size->width = 0; mut_size->height = 0;
}
```

올바른 예:

```c
struct size {
    int width;
    int height;
};

static void reset_size(struct size *mut_size)
{
    mut_size->width = 0;
    mut_size->height = 0;
}
```

### 빈 줄

작업이나 맥락이 바뀌는 덩어리 사이에는 빈 줄을 적절히 둔다.

올바른 예:

```c
static long count_long_lines(const char *path, size_t limit)
{
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        return -1;
    }

    long count = 0;
    size_t length = 0;

    for (int c = fgetc(file); c != EOF; c = fgetc(file)) {
        if (c != '\n') {
            ++length;
            continue;
        }

        if (length > limit) {
            ++count;
        }
        length = 0;
    }

    if (ferror(file) != 0) {
        count = -1;
    } else if (length > limit) {
        ++count;
    }

    fclose(file);
    file = NULL;
    return count;
}
```

### 정렬

연속된 할당, 선언, 매크로, 줄 끝 주석을 열에 맞춰 정렬하지 않는다. 한 줄만 고쳐도 이웃 줄까지 다시 맞춰야 하기 때문이다.

다만 여러 줄에 걸친 매크로의 줄 끝 `\`는 같은 열에 맞춘다. 가장 긴 줄에서는 마지막 글자 뒤에 공백 하나를 두고 `\`를 쓴다. 다른 줄의 `\`도 그 열에 둔다. `\`가 줄마다 코드 바로 뒤에 붙어 열이 제각각이면, 코드를 읽기 어렵고 그 블록이 매크로라는 것도 한눈에 알아보기 어렵다. 반면 `\`가 한 열에 모여 있으면 매크로의 범위가 바로 보이고, 빠진 `\`도 쉽게 눈에 띈다.

틀린 예:

```c
#define MAX_ACCOUNTS  64
#define NAME_LEN      32

struct account {
    char   name[NAME_LEN]; /* login name */
    long   balance;        /* in cents */
    double interest_rate;  /* per year */
};

static void open_account(struct account *mut_account)
{
    mut_account->balance       = 0;
    mut_account->interest_rate = 0.015;
}
```

```c
#define RESET_RANGE(range) \
    do { \
        (range)->first = 0; \
        (range)->last = -1; \
    } while (0)
```

올바른 예:

```c
#define MAX_ACCOUNTS 64
#define NAME_LEN 32

struct account {
    char name[NAME_LEN]; /* login name */
    long balance; /* in cents */
    double interest_rate; /* per year */
};

static void open_account(struct account *mut_account)
{
    mut_account->balance = 0;
    mut_account->interest_rate = 0.015;
}
```

```c
#define RESET_RANGE(range)  \
    do {                    \
        (range)->first = 0; \
        (range)->last = -1; \
    } while (0)
```

### preprocessing directive

- preprocessing directive의 `#`은 늘 줄의 맨 앞에 둔다. 함수나 다른 중괄호 블록 안에 있는 directive도 같다.
- 중첩된 조건부 컴파일 블록 안에서도 directive를 들여쓰지 않는다. `#`과 directive 이름 사이에 공백을 두지 않고 모든 directive를 줄의 맨 앞에 붙인다.
- 조건부 컴파일 블록 안의 C 코드는 directive의 중첩과 상관없이, directive가 없을 때와 같은 들여쓰기를 따른다.

모든 directive를 1열에 붙이면 전처리기 구문이 일반 C 코드의 들여쓰기와 분명히 구별된다. 또한 바깥에 `#if`를 덧붙이거나 뺄 때 안쪽 directive 줄의 공백이 바뀌지 않아 diff가 깔끔하게 유지된다.

틀린 예:

```c
static char get_path_separator(void)
{
    #if defined(_WIN32)
    return '\\';
    #else
    return '/';
    #endif
}
```

```c
#if defined(__linux__)
    #if defined(__GLIBC__)
        #define LIBC_NAME "glibc"
    #else
        #define LIBC_NAME "other libc"
    #endif
#else
    #define LIBC_NAME "unknown"
#endif
```

```c
#if defined(__linux__)
# if defined(__GLIBC__)
#  define LIBC_NAME "glibc"
# else
#  define LIBC_NAME "other libc"
# endif
#else
# define LIBC_NAME "unknown"
#endif
```

올바른 예:

```c
static char get_path_separator(void)
{
#if defined(_WIN32)
    return '\\';
#else
    return '/';
#endif
}
```

```c
#if defined(__linux__)
#if defined(__GLIBC__)
#define LIBC_NAME "glibc"
#else
#define LIBC_NAME "other libc"
#endif
#else
#define LIBC_NAME "unknown"
#endif
```

올바른 예(`platform/paths.h`):

```c
#ifndef PLATFORM_PATHS_H
#define PLATFORM_PATHS_H

#if defined(_WIN32)
#define PATH_SEPARATOR '\\'
#else
#define PATH_SEPARATOR '/'
#endif

#endif /* PLATFORM_PATHS_H */
```

### GCC/Clang 인라인 어셈블리

- 기본 표준인 GNU11(`-std=gnu11`)으로 GCC와 Clang을 주로 쓰는 코드에서는 `asm`을 우선한다. `-std=c11`처럼 순수 ISO C 표준을 엄격하게 지켜야 하는 코드나 그런 모드에서 포함될 공개 헤더에서는 `__asm__`을 쓴다([3.3절](#확장-기능의-표시)).
- 인라인 어셈블리는 명령 하나를 문자열 하나에 쓴다. 마지막 문자열을 뺀 나머지 문자열의 끝에는 `\n\t`를 붙인다.

`\n\t`는 GCC 매뉴얼의 Extended Asm 절에 나오는 권장 구분자이다. `\n`은 명령 사이의 줄바꿈이다. `\t`는 컴파일러가 출력하는 어셈블리에서 명령을 명령 필드까지 들여쓰는 탭이다. 컴파일러는 asm 문의 마지막 명령 뒤에서 스스로 줄을 바꾼다. 따라서, 마지막 문자열에 `\n\t`를 붙이면 탭만 있는 줄이 하나 더 생긴다.

틀린 예:

```c
static inline uint32_t read_max_cpuid_leaf(void)
{
    uint32_t leaf;

    asm volatile("xorl %%eax, %%eax; cpuid"
                 : "=a"(leaf)
                 :
                 : "ebx", "ecx", "edx", "cc");
    return leaf;
}
```

올바른 예:

```c
static inline uint32_t read_max_cpuid_leaf(void)
{
    uint32_t leaf;

    asm volatile("xorl %%eax, %%eax\n\t"
                 "cpuid"
                 : "=a"(leaf)
                 :
                 : "ebx", "ecx", "edx", "cc");
    return leaf;
}
```

## 이름

### 함수와 변수의 이름

함수, 변수, 매개변수, struct·union·enum 태그, goto 레이블은 소문자 snake_case로 짓는다. 크로스플랫폼 프로젝트에서는 Windows에 종속된 코드도 이 규칙을 따른다.

틀린 예:

```c
struct GroceryList {
    pthread_mutex_t lock;
    size_t itemCount;
    long totalCents;
};

union PriceValue {
    long Cents;
    double Ratio;
};

enum CartState {
    CART_STATE_OPEN = 0,
    CART_STATE_PAID,
};

static int AddItem(struct GroceryList *pList, long PriceCents)
{
    int Result = -1;

    pthread_mutex_lock(&pList->lock);
    if (PriceCents < 0 || PriceCents > LONG_MAX - pList->totalCents) {
        goto OutUnlock;
    }

    ++pList->itemCount;
    pList->totalCents += PriceCents;
    Result = 0;

OutUnlock:
    pthread_mutex_unlock(&pList->lock);
    return Result;
}
```

올바른 예:

```c
struct grocery_list {
    pthread_mutex_t lock;
    size_t item_count;
    long total_cents;
};

union price_value {
    long cents;
    double ratio;
};

enum cart_state {
    CART_STATE_OPEN = 0,
    CART_STATE_PAID,
};

static int add_item(struct grocery_list *mut_list, long price_cents)
{
    int result = -1;

    pthread_mutex_lock(&mut_list->lock);
    if (price_cents < 0 || price_cents > LONG_MAX - mut_list->total_cents) {
        goto out_unlock;
    }

    ++mut_list->item_count;
    mut_list->total_cents += price_cents;
    result = 0;

out_unlock:
    pthread_mutex_unlock(&mut_list->lock);
    return result;
}
```

### 매크로와 enumeration constant의 이름

매크로와 enumeration constant는 대문자 snake_case로 짓는다. enumeration constant에 열거형 이름 접두어를 강제하지 않는다.

틀린 예:

```c
#define maxRetries 3

enum shape {
    Circle = 0,
    Square,
};
```

올바른 예:

```c
#define MAX_RETRIES 3

enum shape {
    SHAPE_CIRCLE = 0,
    SHAPE_SQUARE,
};
```

올바른 예(열거형 이름을 접두어로 붙이지 않은 경우):

```c
enum direction {
    NORTH = 0,
    EAST,
    SOUTH,
    WEST,
};
```

### function-like macro

function-like macro도 대문자로 짓는다. 다만 `likely`, `unlikely`처럼 컴파일러 builtin을 감싸는 관용 이름에는 예외로 소문자를 허용한다.

틀린 예:

```c
#define to_string(x) #x
```

올바른 예:

```c
#define TO_STRING(x) #x
```

올바른 예(컴파일러 builtin을 감싸는 관용 이름):

```c
#define likely(x) __builtin_expect(!!(x), 1)
#define unlikely(x) __builtin_expect(!!(x), 0)
```

### const 객체의 이름

const 객체의 이름도 변수 이름 규칙을 따른다. 대문자 이름은 매크로와 enumeration constant에만 쓴다.

틀린 예:

```c
static const int DAYS_IN_MONTH[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
```

올바른 예:

```c
static const int gs_days_in_month[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };
```

### 이름의 길이와 내용

- external linkage가 있는 이름과 file scope 이름은 뜻이 드러나게 짓는다.
- 사용 범위가 좁은 지역 변수는 쓰임이 관례로 굳은 짧은 이름으로 지어도 된다(`i`, `n`, `tmp`, `iter` 등).
- Hungarian notation처럼 타입을 나타내는 접두어나 접미어를 이름에 붙이지 않는다. 이름은 값의 뜻을 나타낸다.
- 널리 쓰이는 약어만 쓴다.
- `l`, `O`처럼 숫자와 혼동되는 한 글자 이름은 쓰지 않는다. 대소문자만 다른 이름도 쓰지 않는다.

틀린 예:

```c
struct reading {
    int celsius;
};

static int avgtmp(const struct reading *pReadings, size_t nCount)
{
    if (nCount == 0) {
        return 0;
    }

    long l = 0;
    size_t L = 0;

    for (size_t i = 0; i < nCount; ++i) {
        l += pReadings[i].celsius;
        ++L;
    }

    return (int)(l / (long)L);
}
```

올바른 예:

```c
struct reading {
    int celsius;
};

static int compute_average_celsius(const struct reading *readings, size_t count)
{
    if (count == 0) {
        return 0;
    }

    long sum = 0;

    for (size_t i = 0; i < count; ++i) {
        sum += readings[i].celsius;
    }

    return (int)(sum / (long)count);
}
```

```c
static void fill_bytes(unsigned char *mut_buffer, size_t size, unsigned char value)
{
    for (size_t i = 0; i < size; ++i) {
        mut_buffer[i] = value;
    }
}
```

### 함수 이름의 첫 단어

함수 이름은 동사로 시작한다. 값을 돌려주는 함수도 같다. 모듈 접두어가 있으면 접두어 다음 단어가 동사이다.

틀린 예:

```c
struct rect {
    double width;
    double height;
};

double perimeter(const struct rect *rect);
unsigned int checksum(const unsigned char *bytes, size_t count);
int cart_item_add(const char *name);
```

올바른 예:

```c
struct rect {
    double width;
    double height;
};

double compute_perimeter(const struct rect *rect);
unsigned int calculate_checksum(const unsigned char *bytes, size_t count);
int cart_add_item(const char *name);
```

### 참 거짓을 나타내는 이름

참 거짓을 돌려주는 함수와 function-like macro, 참 거짓 값을 담는 변수, 매개변수, struct 멤버의 이름은 다음 두 형태 가운데 하나로 짓는다.

- be 동사나 의문문을 만드는 동사로 시작한다. `is_` 뒤에는 보어인 명사나 형용사를, `has_` 뒤에는 명사나 과거분사를, `can_`과 `should_` 뒤에는 동사 원형을 붙인다(`is_empty`, `is_directory`, `has_children`, `has_found`, `can_retry`, `should_flush`).
- 상태를 나타내는 3인칭 단수 동사로 시작한다(`exists`, `contains_key`, `matches_pattern`).

틀린 예:

```c
#define EVEN(n) (((n) % 2) == 0)

struct download {
    bool finished;
    int retry;
};

bool palindrome(const char *text);
bool check_key(const char *key);
bool find_name(const char *const *names, size_t count, const char *name);
```

```c
static size_t count_code_lines(const char *const *lines, size_t count)
{
    size_t code_lines = 0;
    int inside_comment = 0;

    for (size_t i = 0; i < count; ++i) {
        const char *text = lines[i] + strspn(lines[i], " \t");

        if (strncmp(text, "/*", 2) == 0) {
            inside_comment = 1;
        }

        if (inside_comment == 0 && text[0] != '\0') {
            ++code_lines;
        }

        if (strstr(text, "*/") != NULL) {
            inside_comment = 0;
        }
    }

    return code_lines;
}
```

```c
static void report_negative(const int *values, size_t count)
{
    bool found = false;

    for (size_t i = 0; i < count; ++i) {
        if (values[i] < 0) {
            found = true;
            break;
        }
    }

    puts((found) ? "negative value found" : "all values are non-negative");
}
```

올바른 예:

```c
#define IS_EVEN(n) (((n) % 2) == 0)

struct download {
    bool is_finished;
    int can_retry;
};

bool is_palindrome(const char *text);
bool is_valid_key(const char *key);
bool contains_name(const char *const *names, size_t count, const char *name);
```

```c
static size_t count_code_lines(const char *const *lines, size_t count)
{
    size_t code_lines = 0;
    int is_inside_comment = 0;

    for (size_t i = 0; i < count; ++i) {
        const char *text = lines[i] + strspn(lines[i], " \t");

        if (strncmp(text, "/*", 2) == 0) {
            is_inside_comment = 1;
        }

        if (is_inside_comment == 0 && text[0] != '\0') {
            ++code_lines;
        }

        if (strstr(text, "*/") != NULL) {
            is_inside_comment = 0;
        }
    }

    return code_lines;
}
```

```c
static void report_negative(const int *values, size_t count)
{
    bool has_found = false;

    for (size_t i = 0; i < count; ++i) {
        if (values[i] < 0) {
            has_found = true;
            break;
        }
    }

    puts((has_found) ? "negative value found" : "all values are non-negative");
}
```

### 전역 변수와 static 변수

변수 이름의 접두어는 file scope 여부와 `static` 여부로 정한다. `g`는 file scope를, `s`는 `static`을 뜻한다.

- `static`이 없는 file scope 변수는 `g_`로 시작한다.
- `static`이 있는 file scope 변수는 `gs_`로 시작한다.
- 함수 안의 `static` 변수는 `s_`로 시작한다.
- `static`이 없는 지역 변수에는 접두어를 붙이지 않는다.

`g_`, `gs_`, `s_`가 붙는 변수는 모두 static storage duration 객체이다. 따라서, 프로그램이 끝날 때까지 남고, 프로그램 시작 전에 한 번만 초기화된다. 차이는 scope와 linkage이다. `g_` 변수는 다른 파일에서도 보이고, `gs_` 변수는 정의한 파일 안에서만 보인다. `s_` 변수는 정의한 함수 안에서만 보이지만, 값이 호출과 호출 사이에 남는다. 접두어가 있으면 변수를 쓰는 자리마다 이 수명과 범위가 드러난다.

틀린 예:

```c
int font_size = 12;
static size_t g_retry_limit = 3;

static unsigned int issue_ticket(void)
{
    static unsigned int last_ticket;

    ++last_ticket;
    unsigned int s_ticket = last_ticket;

    return s_ticket;
}
```

올바른 예:

```c
int g_font_size = 12;
static size_t gs_retry_limit = 3;

static unsigned int issue_ticket(void)
{
    static unsigned int s_last_ticket;

    ++s_last_ticket;
    unsigned int ticket = s_last_ticket;

    return ticket;
}
```

### typedef

struct, union, enum에 typedef를 붙이는 것은 가능한 한 지양한다. `struct point`처럼 태그를 그대로 쓴다. 특히 enum 타입의 변수는 특별한 이유가 없으면 typedef로 enum임을 감추지 않고 `enum` 태그로 선언한다. typedef는 내부를 감추는 opaque type과 함수 포인터 타입에 어울린다.

지양하는 예:

```c
typedef struct point {
    int x;
    int y;
} point;

typedef enum {
    FAN_SPEED_LOW = 0,
    FAN_SPEED_HIGH,
} fan_speed;

static fan_speed gs_fan_speed = FAN_SPEED_HIGH;
```

올바른 예:

```c
struct point {
    int x;
    int y;
};

enum fan_speed {
    FAN_SPEED_LOW = 0,
    FAN_SPEED_HIGH,
};

static enum fan_speed gs_fan_speed = FAN_SPEED_HIGH;
```

```c
typedef struct audio_device audio_device;
typedef void (*sample_callback)(const float *samples, size_t count);

audio_device *open_audio_device(const char *name, sample_callback on_samples);
```

### 예약된 식별자

C 표준은 일부 이름을 implementation이 쓰도록 예약한다. implementation은 컴파일러와 표준 라이브러리를 함께 이르는 C 표준의 용어이다. 표준 헤더와 컴파일러는 내부 매크로와 함수에 이런 이름을 쓴다. 프로그램이 예약된 식별자를 선언하거나 정의하면 undefined behavior이다(C99 §7.1.3 ¶2).

다음 이름은 쓰지 않는다.

- `_`와 대문자로 시작하는 이름(`_STORE_CART_H`)과 `__`로 시작하는 이름(`__retry_count`): 어떤 용도로도 예약되어 있다(C99 §7.1.3 ¶1).
- file scope에서 `_`로 시작하는 이름(`_buffer_size`): file scope 식별자로 예약되어 있다(C99 §7.1.3 ¶1).
- `_t`로 끝나는 이름(`item_id_t`): POSIX에서는 implementation이 어느 헤더에든 이런 이름을 새로 넣을 수 있다(POSIX.1-2024 XSH 2.2.2). glibc도 이런 이름을 타입 이름용으로 예약한다(glibc 매뉴얼 Reserved Names).

`_POSIX_C_SOURCE`처럼 표준이 사용자에게 정의하라고 정한 feature test macro는 예외이다.

틀린 예(`store/cart.h`):

```c
#ifndef _STORE_CART_H
#define _STORE_CART_H

#include <stddef.h>

typedef unsigned long item_id_t;

extern size_t __retry_count;

#endif /* _STORE_CART_H */
```

틀린 예(`store/cart.c`):

```c
#include "store/cart.h"

static size_t _buffer_size;
```

올바른 예(`store/cart.h`):

```c
#ifndef STORE_CART_H
#define STORE_CART_H

#include <stddef.h>

typedef unsigned long item_id;

extern size_t g_retry_count;

#endif /* STORE_CART_H */
```

올바른 예(`store/cart.c`):

```c
#include "store/cart.h"

static size_t gs_buffer_size;
```

### Win32 API 심볼과 타입

크로스플랫폼 코드를 개발할 때 Win32 API의 심볼과 타입 이름(`CreateFileW`, `HANDLE`, `SOCKET`)은 원래 철자로 쓴다. Win32 API의 이름을 다른 이름으로 감싸지 않는다. 직접 짓는 이름에는 Win32 API의 표기법을 따라 하지 않는다. 다만 Windows 단독 프로젝트에서는 직접 짓는 이름에도 Windows 관례를 따라도 무방하다([1.1절](#적용-대상)).

틀린 예:

```c
#include <windows.h>

#define close_log CloseHandle

typedef HANDLE log_handle;

static log_handle OpenLogFileW(const wchar_t *pszPath)
{
    return CreateFileW(pszPath, FILE_APPEND_DATA, FILE_SHARE_READ, NULL, OPEN_ALWAYS,
                       FILE_ATTRIBUTE_NORMAL, NULL);
}

static void CloseLogFile(log_handle hFile)
{
    close_log(hFile);
}
```

올바른 예(Windows):

```c
#include <windows.h>

static HANDLE open_log_file(const wchar_t *path)
{
    return CreateFileW(path, FILE_APPEND_DATA, FILE_SHARE_READ, NULL, OPEN_ALWAYS,
                       FILE_ATTRIBUTE_NORMAL, NULL);
}

static void close_log_file(HANDLE file)
{
    CloseHandle(file);
}
```

## 선언과 타입

### 한 줄에 선언 하나

선언 한 줄에는 변수 하나만 둔다. 다만 `for` 문의 초기식에서는 함께 다루는 반복 변수 여러 개를 한 선언에 둘 수 있다.

틀린 예:

```c
static void print_space_positions(const char *text)
{
    const char *first_space = strchr(text, ' '), *last_space = strrchr(text, ' ');

    if (first_space == NULL) {
        puts("no spaces");
        return;
    }

    size_t first_index = (size_t)(first_space - text), last_index = (size_t)(last_space - text);

    printf("spaces at %zu and %zu\n", first_index, last_index);
}
```

올바른 예:

```c
static void print_space_positions(const char *text)
{
    const char *first_space = strchr(text, ' ');
    const char *last_space = strrchr(text, ' ');

    if (first_space == NULL) {
        puts("no spaces");
        return;
    }

    size_t first_index = (size_t)(first_space - text);
    size_t last_index = (size_t)(last_space - text);

    printf("spaces at %zu and %zu\n", first_index, last_index);
}
```

### 선언 위치

변수는 처음 쓰는 곳에서, 가장 좁은 범위에 선언한다. 반복 변수는 `for` 초기식에서 선언한다.

틀린 예:

```c
struct order {
    long amount_cents;
    int is_paid;
};

static long sum_unpaid(const struct order *orders, size_t count)
{
    size_t i;
    long total;
    const struct order *order;

    total = 0;
    for (i = 0; i < count; ++i) {
        order = &orders[i];
        if (order->is_paid == 0) {
            total += order->amount_cents;
        }
    }

    return total;
}
```

올바른 예:

```c
struct order {
    long amount_cents;
    int is_paid;
};

static long sum_unpaid(const struct order *orders, size_t count)
{
    long total = 0;

    for (size_t i = 0; i < count; ++i) {
        const struct order *order = &orders[i];

        if (order->is_paid == 0) {
            total += order->amount_cents;
        }
    }

    return total;
}
```

다만 C89를 고른 프로젝트는 예외로 한다. C89는 문장 뒤의 선언과 `for` 초기식 안의 선언을 허용하지 않는다. 따라서, 변수는 쓰이는 가장 좁은 블록의 맨 앞에 선언하고, 반복 변수는 반복문을 감싼 블록의 맨 앞에 선언한다.

C89 프로젝트의 올바른 예:

```c
struct order {
    long amount_cents;
    int is_paid;
};

static long sum_unpaid(const struct order *orders, size_t count)
{
    long total = 0;
    size_t i;

    for (i = 0; i < count; ++i) {
        const struct order *order = &orders[i];

        if (order->is_paid == 0) {
            total += order->amount_cents;
        }
    }

    return total;
}
```

### 범위와 shadowing

한 함수에서만 쓰는 변수는 그 함수 안에 둔다.

함수와 file scope 변수는 기본으로 `static`으로 선언한다. 다른 파일에서 써야 하는 등 따로 사유가 있을 때만 `static`을 붙이지 않는다. `static`으로 선언하면 이름이 다른 파일에 드러나지 않는다. 따라서, 다른 파일에 같은 이름이 있어도 링크가 깨지지 않고, 쓰지 않는 함수는 컴파일러가 경고한다.

안쪽 범위에서 바깥 범위와 같은 이름을 선언하지 않는다. 이런 shadowing은 `-Wshadow`가 경고한다.

틀린 예:

```c
size_t total_errors;
char error_text[128];

size_t count_errors(const int *codes, size_t count)
{
    size_t total_errors = 0;

    for (size_t i = 0; i < count; ++i) {
        if (codes[i] != 0) {
            ++total_errors;
        }
    }

    snprintf(error_text, sizeof(error_text), "%zu errors", total_errors);
    puts(error_text);
    return total_errors;
}
```

올바른 예:

```c
static size_t gs_total_errors;

static size_t count_errors(const int *codes, size_t count)
{
    size_t errors = 0;

    for (size_t i = 0; i < count; ++i) {
        if (codes[i] != 0) {
            ++errors;
        }
    }

    gs_total_errors += errors;

    char error_text[128];

    snprintf(error_text, sizeof(error_text), "%zu errors", errors);
    puts(error_text);
    return errors;
}
```

### 수명이 끝난 대상을 가리키는 포인터

포인터가 가리키는 대상이 범위를 벗어나 수명이 끝났는데 포인터 변수는 남아 있으면 `NULL`을 할당한다. 더는 그 포인터로 접근할 일이 없어도 할당한다. 그러면 실수로 그 포인터를 다시 써도 사라진 대상 대신 `NULL`을 다룬다. 따라서, `NULL` 검사에 걸리거나 대개 그 자리에서 바로 실패한다.

틀린 예:

```c
struct word_scanner {
    const char *cursor;
    size_t word_count;
};

void scan_words(struct word_scanner *mut_scanner);

static int scan_next_line(struct word_scanner *mut_scanner, FILE *mut_input)
{
    char line[128];

    if (fgets(line, sizeof(line), mut_input) == NULL) {
        return -1;
    }

    if (strchr(line, '\n') == NULL && feof(mut_input) == 0) {
        return -1;
    }

    mut_scanner->cursor = line;
    scan_words(mut_scanner);
    return 0;
}
```

```c
static void print_greeting(const char *name, bool is_formal)
{
    const char *greeting = "hi";

    if (is_formal) {
        char formal[64];

        snprintf(formal, sizeof(formal), "good evening, %s", name);
        greeting = formal;
        puts(greeting);
    } else {
        puts(greeting);
    }

    printf("greeted %s\n", name);
}
```

올바른 예:

```c
struct word_scanner {
    const char *cursor;
    size_t word_count;
};

void scan_words(struct word_scanner *mut_scanner);

static int scan_next_line(struct word_scanner *mut_scanner, FILE *mut_input)
{
    char line[128];

    if (fgets(line, sizeof(line), mut_input) == NULL) {
        return -1;
    }

    if (strchr(line, '\n') == NULL && feof(mut_input) == 0) {
        return -1;
    }

    mut_scanner->cursor = line;
    scan_words(mut_scanner);
    mut_scanner->cursor = NULL;
    return 0;
}
```

```c
static void print_greeting(const char *name, bool is_formal)
{
    const char *greeting = "hi";

    if (is_formal) {
        char formal[64];

        snprintf(formal, sizeof(formal), "good evening, %s", name);
        greeting = formal;
        puts(greeting);
        greeting = NULL;
    } else {
        puts(greeting);
    }

    printf("greeted %s\n", name);
}
```

### 정수 타입

- 크기, 개수, 인덱스에는 `size_t`를 쓴다.
- 타입의 크기를 정확히 맞춰야 하면 `<stdint.h>`의 exact-width integer type을 쓴다. 다만 Linux의 POSIX API처럼 반환형과 매개변수가 대개 기본 타입인 API를 쓰는 코드에서는 `<stdint.h>` 대신 그 API의 타입을 따라도 된다.
- 바이트 데이터는 `unsigned char` 또는 `uint8_t`로 다룬다.
- 부호 있는 값과 없는 값을 섞어 연산할 때는 명시적으로 캐스트한다.

틀린 예:

```c
static unsigned int sum_bytes(const char *data, int length)
{
    unsigned int sum = 0;

    for (int i = 0; i < length; ++i) {
        sum += data[i];
    }

    return sum;
}

static int is_inside(int offset, size_t length)
{
    return offset < length;
}
```

올바른 예:

```c
static uint32_t sum_bytes(const unsigned char *data, size_t length)
{
    uint32_t sum = 0;

    for (size_t i = 0; i < length; ++i) {
        sum += data[i];
    }

    return sum;
}

static int is_inside(int offset, size_t length)
{
    return offset >= 0 && (size_t)offset < length;
}
```

올바른 예(POSIX API의 타입을 따르는 경우):

```c
static int read_exactly(int fd, unsigned char *mut_buffer, size_t size)
{
    size_t done = 0;

    while (done < size) {
        ssize_t count = read(fd, mut_buffer + done, size - done);

        if (count < 0 && errno == EINTR) {
            continue;
        }

        if (count <= 0) {
            return -1;
        }

        done += (size_t)count;
    }

    return 0;
}
```

### 참 거짓 값

참 거짓 값은 `<stdbool.h>`의 `bool`, `true`, `false`로 다뤄도 되고, `int` 같은 정수 타입과 `1`, `0`으로 다뤄도 된다. 다만 Linux의 POSIX API처럼 반환형과 매개변수가 대개 기본 타입인 API를 쓰는 코드에서는 `int`를 우선한다. 그 밖의 코드에서는 `bool`을 써도 된다.

정수로 다룰 때 참은 `1`, 거짓은 `0`이다. `0`이 아닌 값은 모두 참으로 해석되지만, 참 상태를 직접 정해 넣는 코드에서는 `1`을 우선한다.

틀린 예:

```c
#define FLAG_DIRTY 0x4U

static int is_dirty(unsigned int flags)
{
    return flags & FLAG_DIRTY; // [!hl]
}
```

올바른 예:

```c
#define FLAG_DIRTY 0x4U

static int is_dirty(unsigned int flags)
{
    return (flags & FLAG_DIRTY) != 0; // [!hl]
}
```

```c
#include <stdbool.h>

#define FLAG_DIRTY 0x4U

static bool gs_is_running = true;

static bool is_dirty(unsigned int flags)
{
    return (flags & FLAG_DIRTY) != 0;
}
```

```c
#define FLAG_DIRTY 0x4U

static int gs_is_running = 1;

static int is_dirty(unsigned int flags)
{
    return (flags & FLAG_DIRTY) != 0;
}
```

C89에는 `<stdbool.h>`가 없다. 따라서, C89를 고른 프로젝트는 참 거짓 값을 정수 타입으로 다룬다. 이때 `bool`, `true`, `false`를 직접 정의하지 않는다. C23 표준에서 세 이름이 모두 키워드가 되어 충돌하기 때문이다.

C89 프로젝트의 틀린 예:

```c
#define true 1
#define false 0

typedef int bool;

static bool gs_is_running = true;
```

C89 프로젝트의 올바른 예:

```c
static int gs_is_running = 1;
```

### 상수

- 서로 관련된 정수 상수의 묶음은 enum으로 정의한다.
- 단독 상수는 `#define`으로 정의한다.
- 타입이 중요한 상수는 `static const` 객체로 정의한다.

enum의 첫 enumeration constant에는 반드시 값을 명시한다. 첫 값은 생략해도 0이 되지만(C99 §6.7.2.2 ¶3), 값을 명시하면 0에서 시작한다는 의도가 선언에 드러난다.

C에서 `static const` 객체는 정수 상수식이 아니다(C99 §6.6 ¶6). 따라서, 배열 크기, `case` 레이블, enumeration constant의 값처럼 정수 상수식이 필요한 자리에는 `#define`이나 enum으로 정의한 상수를 쓴다.

틀린 예:

```c
#define COLOR_RED 0
#define COLOR_GREEN 1
#define COLOR_BLUE 2

#define MILLISECONDS_PER_DAY 86400000

static long long convert_days_to_ms(int days)
{
    return days * MILLISECONDS_PER_DAY;
}
```

`MILLISECONDS_PER_DAY`는 `int` 상수이므로 `days * MILLISECONDS_PER_DAY`도 `int`로 계산된다. 따라서, `days`가 25 이상이면 곱셈 결과가 `long long`으로 바뀌기 전에 `int`의 범위를 넘는다.

틀린 예(enum의 첫 값을 생략한 경우):

```c
enum color {
    COLOR_RED,
    COLOR_GREEN,
    COLOR_BLUE,
};
```

올바른 예:

```c
#define MAX_RETRIES 3

enum color {
    COLOR_RED = 0,
    COLOR_GREEN,
    COLOR_BLUE,
};

static const long long gs_milliseconds_per_day = 86400000;

static long long convert_days_to_ms(int days)
{
    return days * gs_milliseconds_per_day;
}
```

### const

- 지역 변수나 포인터 변수 자체에는 `const`를 강제하지 않는다(`*const`와 typedef로 정의한 포인터 타입 앞의 `const` 포함). 매개변수 자체에는 `const`를 붙이지 않는다([10.3절](#포인터-매개변수)).
- 포인터가 가리키는 대상을 바꾸지 않으면 `const T *name`으로 선언한다. 문자열 리터럴을 가리키는 포인터는 대상을 바꿀 수 없으므로 반드시 `const char *`로 선언한다.
- 읽기 전용으로 쓰는 정적 포인터 배열(`static const char *const table[]`)처럼 테이블 자체가 바뀌지 않는 경우에는 배열 원소에도 `const`를 붙여 읽기 전용 영역에 배치한다.
- 값으로 돌려주는 반환형에는 `const`를 붙이지 않는다.

틀린 예:

```c
static char *gs_default_unit = "cm";

const double compute_ratio(double part, double whole);
```

올바른 예:

```c
static const char *gs_default_unit = "cm";
static const char *const gs_units[] = { "mm", "cm", "m" };

double compute_ratio(double part, double whole);
```

```c
static double convert_to_inches(double centimeters)
{
    double inches = centimeters / 2.54;

    return inches;
}
```

틀린 예(대상을 바꾸지 않는 포인터 변수):

```c
static void print_extension(const char *file_name)
{
    char *dot = strrchr(file_name, '.'); // [!hl]

    if (dot != NULL) {
        printf("extension: %s\n", dot + 1);
    }
}
```

올바른 예(대상을 바꾸지 않는 포인터 변수):

```c
static void print_extension(const char *file_name)
{
    const char *dot = strrchr(file_name, '.'); // [!hl]

    if (dot != NULL) {
        printf("extension: %s\n", dot + 1);
    }
}
```

### register와 volatile

`register`는 써도 된다. `volatile`은 다음 경우에만 쓴다.

- 시그널 핸들러와 공유하는 변수(`volatile sig_atomic_t`)
- memory-mapped I/O
- `setjmp` 이후에 읽는 지역 변수
- 최적화를 막아야 하는 벤치마크 sink
- 결과를 쓰지 않아도 실행되어야 하거나, 호출할 때마다 다시 실행되어야 하는 인라인 어셈블리(`asm volatile`)

스레드 사이의 동기화에는 `volatile`을 쓰지 않고, mutex나 atomic 연산을 쓴다.

틀린 예:

```c
static int gs_result;
static volatile int gs_has_result;

static void publish_result(int value)
{
    gs_result = value;
    gs_has_result = 1;
}

static int read_result(int *mut_value)
{
    if (gs_has_result == 0) {
        return -1;
    }

    *mut_value = gs_result;
    return 0;
}
```

올바른 예:

```c
static pthread_mutex_t gs_result_lock;
static int gs_result;
static int gs_has_result;

static void publish_result(int value)
{
    pthread_mutex_lock(&gs_result_lock);
    gs_result = value;
    gs_has_result = 1;
    pthread_mutex_unlock(&gs_result_lock);
}

static int read_result(int *mut_value)
{
    int ret = -1;

    pthread_mutex_lock(&gs_result_lock);
    if (gs_has_result != 0) {
        *mut_value = gs_result;
        ret = 0;
    }
    pthread_mutex_unlock(&gs_result_lock);
    return ret;
}
```

올바른 예(`register`):

```c
static size_t count_set_bits(unsigned int value)
{
    register size_t count = 0;

    while (value != 0) {
        value &= value - 1;
        ++count;
    }

    return count;
}
```

올바른 예(시그널 핸들러와 공유하는 변수):

```c
static volatile sig_atomic_t gs_should_stop;

static void handle_sigterm(int signo)
{
    gs_should_stop = 1;
}
```

올바른 예(memory-mapped I/O):

```c
#define TIMER_COUNT (*(volatile const uint32_t *)0x40001024UL)

static uint32_t read_timer(void)
{
    return TIMER_COUNT;
}
```

올바른 예(`setjmp` 이후에 읽는 지역 변수):

```c
jmp_buf g_retry_point;

int parse_or_jump(const char *text);

static int parse_with_retry(const char *text)
{
    volatile size_t attempt_count = 0;

    if (setjmp(g_retry_point) != 0) {
        if (attempt_count >= 3) {
            return -1;
        }
    }

    ++attempt_count;
    return parse_or_jump(text);
}
```

올바른 예(벤치마크 sink):

```c
static volatile unsigned long gs_sink;

unsigned long hash_word(const char *word);

static void run_hash_benchmark(const char *const *words, size_t count)
{
    for (size_t i = 0; i < count; ++i) {
        gs_sink = hash_word(words[i]);
    }
}
```

올바른 예(인라인 어셈블리):

```c
static inline int read_hardware_random(uint64_t *mut_value)
{
    unsigned char is_ok;

    asm volatile("rdrand %0\n\t"
                 "setc %1"
                 : "=&r"(*mut_value), "=qm"(is_ok)
                 :
                 : "cc");
    return (is_ok != 0) ? 0 : -1;
}
```

### VLA

VLA(variable length array)를 쓰지 않는다. 실행 중에 크기가 정해지면 동적 할당을 쓰거나, 호출하는 쪽에서 미리 마련한 배열을 매개변수로 받는다.

틀린 예:

```c
static int print_histogram(const unsigned char *values, size_t count, size_t bucket_count)
{
    if (bucket_count == 0) {
        return -1;
    }

    size_t buckets[bucket_count];

    memset(buckets, 0, sizeof(buckets));
    for (size_t i = 0; i < count; ++i) {
        ++buckets[values[i] % bucket_count];
    }

    for (size_t i = 0; i < bucket_count; ++i) {
        printf("%zu: %zu\n", i, buckets[i]);
    }

    return 0;
}
```

올바른 예:

```c
static int print_histogram(const unsigned char *values, size_t count, size_t bucket_count)
{
    if (bucket_count == 0) {
        return -1;
    }

    size_t *buckets = calloc(bucket_count, sizeof(*buckets));
    if (buckets == NULL) {
        return -1;
    }

    for (size_t i = 0; i < count; ++i) {
        ++buckets[values[i] % bucket_count];
    }

    for (size_t i = 0; i < bucket_count; ++i) {
        printf("%zu: %zu\n", i, buckets[i]);
    }

    free(buckets);
    buckets = NULL;
    return 0;
}
```

올바른 예(배열을 매개변수로 받는 경우):

```c
static int print_histogram(size_t *mut_buckets, size_t bucket_count,
                           const unsigned char *values, size_t count)
{
    if (bucket_count == 0) {
        return -1;
    }

    memset(mut_buckets, 0, bucket_count * sizeof(*mut_buckets));
    for (size_t i = 0; i < count; ++i) {
        ++mut_buckets[values[i] % bucket_count];
    }

    for (size_t i = 0; i < bucket_count; ++i) {
        printf("%zu: %zu\n", i, mut_buckets[i]);
    }

    return 0;
}
```

### 초기화

- 초깃값이 의미 있는 변수는 선언과 함께 초기화한다.
- 배열을 선언하면서 0으로 채울 때는 `{ 0 }`으로 초기화한다. 생략한 나머지 원소는 static storage duration 객체처럼 0으로 초기화된다(C99 §6.7.8 ¶21).
- 구조체는 designated initializer로 초기화하기를 권장한다.

틀린 예:

```c
static double compute_average(const double *values, size_t count)
{
    if (count == 0) {
        return 0.0;
    }

    double sum;

    sum = 0.0;
    for (size_t i = 0; i < count; ++i) {
        sum += values[i];
    }

    return sum / (double)count;
}
```

올바른 예:

```c
static double compute_average(const double *values, size_t count)
{
    if (count == 0) {
        return 0.0;
    }

    double sum = 0.0;

    for (size_t i = 0; i < count; ++i) {
        sum += values[i];
    }

    return sum / (double)count;
}
```

```c
static void print_letter_counts(const char *text)
{
    size_t counts[26] = { 0 };

    for (size_t i = 0; text[i] != '\0'; ++i) {
        if (text[i] >= 'a' && text[i] <= 'z') {
            ++counts[text[i] - 'a'];
        }
    }

    for (size_t letter = 0; letter < sizeof(counts) / sizeof(counts[0]); ++letter) {
        printf("%c: %zu\n", 'a' + (int)letter, counts[letter]);
    }
}
```

```c
struct window_config {
    int width;
    int height;
    const char *title;
};

static void print_default_window(void)
{
    struct window_config config = {
        .width = 800,
        .height = 600,
        .title = "untitled"
    };

    printf("%s: %dx%d\n", config.title, config.width, config.height);
}
```

다만 C89를 고른 프로젝트는 예외로 한다. C89에는 designated initializer가 없으므로, 구조체는 `{ 0 }`으로 0을 채워 선언한 뒤 필요한 멤버에 할당한다.

C89 프로젝트의 올바른 예:

```c
struct window_config {
    int width;
    int height;
    const char *title;
};

static void print_default_window(void)
{
    struct window_config config = { 0 };

    config.width = 800;
    config.height = 600;
    config.title = "untitled";
    printf("%s: %dx%d\n", config.title, config.width, config.height);
}
```

실행 중에 값이 정해지는 file scope 변수는 되도록 프로그램 초반부에 초기화한다. 즉, `main`의 앞부분이나 초기화 함수에서 초기화한다. 실행 중에 채우는 구조체, 동적 할당한 메모리, 라이브러리나 OS에서 받아 오는 값이 여기에 해당한다. static storage duration 객체의 초기화식은 상수식이어야 하므로(C99 §6.7.8 ¶4), 이런 값은 정의할 때 초기화식으로 쓸 수 없다. 반면 컴파일할 때 값이 정해지는 상수와 const 표는 정의할 때 초기화한다.

올바른 예:

```c
#define SETTINGS_PATH "editor.conf"

struct settings;

struct settings *load_settings(const char *path);
void run_editor(const struct settings *settings);

static const unsigned int gs_autosave_intervals_s[] = { 30, 60, 300 };
static const struct settings *gs_settings;

int main(void)
{
    gs_settings = load_settings(SETTINGS_PATH);
    if (gs_settings == NULL) {
        fprintf(stderr, "cannot load settings: %s\n", SETTINGS_PATH);
        return EXIT_FAILURE;
    }

    run_editor(gs_settings);
    return 0;
}
```

## 식

### 리터럴 표기

- 16진 리터럴은 `0x` 접두어를 소문자로, 16진 숫자를 대문자로 쓴다.
- 2진 리터럴은 `0b` 접두어를 소문자로 쓴다. 2진 리터럴은 C23 표준에서 정식 문법이 되었다. C17 이하를 고른 프로젝트에서는 GNU 확장이므로 `__extension__`을 붙인다.
- 접미어는 모두 대문자로 쓴다(`U`, `L`, `UL`, `ULL`, `F`). 소문자 `l`은 숫자 `1`과 헷갈리기 쉬우므로, 다른 접미어도 함께 대문자로 맞춘다.
- 부동소수점 리터럴에는 소수점을 쓴다.

틀린 예:

```c
static const uint32_t gs_color_mask = 0Xff00ff00u;
static const unsigned int gs_mode_bits = 0B0110u;
static const long gs_max_delay_ms = 65536l;
static const unsigned long gs_high_bit = 0x80000000ul;
static const unsigned long long gs_poll_interval_ns = 4000000000ull;
static const float gs_half = .5f;
static const double gs_timeout_s = 2;
```

올바른 예:

```c
static const uint32_t gs_color_mask = 0xFF00FF00U;
static const unsigned int gs_mode_bits = __extension__ 0b0110U;
static const long gs_max_delay_ms = 65536L;
static const unsigned long gs_high_bit = 0x80000000UL;
static const unsigned long long gs_poll_interval_ns = 4000000000ULL;
static const float gs_half = 0.5F;
static const double gs_timeout_s = 2.0;
```

### 증감 연산자

증감 연산자는 기본적으로 전위형(`++i`)을 쓴다. 후위형은 이전 값이 꼭 필요할 때만 쓴다. 특별한 이유가 없는 한, 배열 첨자, 함수 인자, 다른 연산자의 피연산자 안에서는 증감 연산자를 쓰지 않고 별도 문장으로 뺀다. 한 식에서 같은 객체를 두 번 이상 수정하지 않는다. 역참조한 대상을 하나 늘리거나 줄일 때는 증감 연산자 대신 `*p += 1`, `*p -= 1`로 쓴다. 대상과 연산이 분명히 나뉘고, `*p++`와 헷갈릴 여지가 없기 때문이다.

틀린 예:

```c
static size_t copy_positive_values(int *mut_dst, const int *src, size_t count)
{
    size_t copied = 0;

    for (size_t i = 0; i < count; i++) {
        if (src[i] > 0) {
            mut_dst[copied++] = src[i];
        }
    }

    return copied;
}
```

```c
static int read_pair_sum(size_t *mut_index, const int *values)
{
    size_t i = *mut_index;
    int sum = values[i++] + values[i++];

    *mut_index = i;
    return sum;
}
```

올바른 예:

```c
static size_t copy_positive_values(int *mut_dst, const int *src, size_t count)
{
    size_t copied = 0;

    for (size_t i = 0; i < count; ++i) {
        if (src[i] > 0) {
            mut_dst[copied] = src[i];
            ++copied;
        }
    }

    return copied;
}
```

```c
static int read_pair_sum(size_t *mut_index, const int *values)
{
    size_t i = *mut_index;
    int sum = values[i] + values[i + 1];

    *mut_index = i + 2;
    return sum;
}
```

올바른 예(거꾸로 도는 반복문):

```c
static void print_reversed(const int *values, size_t count)
{
    for (size_t i = count; i > 0; --i) {
        printf("%d\n", values[i - 1]);
    }
}
```

틀린 예(역참조한 대상에 증감 연산자를 쓴 경우):

```c
static void skip_spaces(const char **mut_cursor)
{
    while (**mut_cursor == ' ') {
        ++*mut_cursor; // [!hl]
    }
}
```

올바른 예(역참조한 대상을 하나 늘리는 경우):

```c
static void skip_spaces(const char **mut_cursor)
{
    while (**mut_cursor == ' ') {
        *mut_cursor += 1; // [!hl]
    }
}
```

### sizeof

- 동적 할당 함수나 `memset`, `memcpy` 같은 함수에 넘기는 크기는 대상 변수에서 얻는다(`sizeof(*p)`, `sizeof(var)`).
- 배열 객체의 원소 수는 `sizeof(arr) / sizeof(arr[0])`로 구한다. 이 식을 파일이나 프로젝트 안에서 높은 빈도로 재사용할 때만 `ARRAY_SIZE` 같은 매크로로 따로 정의한다.
- 동적 할당한 메모리와 배열 매개변수는 포인터로 다루므로, 원소 수를 따로 넘긴다.

틀린 예:

```c
struct order {
    char customer[32];
    long amount_cents;
};

static struct order *create_order(const char *customer)
{
    struct order *order = malloc(sizeof(struct order));
    if (order == NULL) {
        return NULL;
    }

    memset(order, 0, sizeof(struct order));

    int length = snprintf(order->customer, 32, "%s", customer);
    if (length < 0 || length >= 32) {
        free(order);
        order = NULL;
        return NULL;
    }

    return order;
}

static void print_names(const char *const *names)
{
    for (size_t i = 0; i < sizeof(names) / sizeof(names[0]); ++i) {
        puts(names[i]);
    }
}
```

올바른 예:

```c
struct order {
    char customer[32];
    long amount_cents;
};

static struct order *create_order(const char *customer)
{
    struct order *order = malloc(sizeof(*order));
    if (order == NULL) {
        return NULL;
    }

    memset(order, 0, sizeof(*order));

    int length = snprintf(order->customer, sizeof(order->customer), "%s", customer);
    if (length < 0 || (size_t)length >= sizeof(order->customer)) {
        free(order);
        order = NULL;
        return NULL;
    }

    return order;
}

static void print_names(const char *const *names, size_t count)
{
    for (size_t i = 0; i < count; ++i) {
        puts(names[i]);
    }
}
```

### 캐스트

- `void *`를 다른 객체 포인터 타입으로 바꿀 때는 캐스트하지 않는다. `malloc`과 `calloc`의 반환값도 같다.
- 값이 잘릴 수 있는 변환과 부호가 섞인 연산에는 명시적 캐스트를 쓴다.
- 캐스트한 결과를 다시 캐스트할 때는 문법상 필요 없더라도 안쪽 캐스트 식을 소괄호로 감싼다. 그러면 각 변환이 안쪽부터 차례로 적용되는 별개의 단계로 읽힌다.

틀린 예:

```c
static int32_t *create_samples(size_t count)
{
    int32_t *samples = (int32_t *)calloc(count, sizeof(*samples));

    return samples;
}

static int compute_percent(size_t done, size_t total)
{
    if (total == 0 || done > total || done > SIZE_MAX / 100) {
        return -1;
    }

    int percent = (done * 100) / total;

    return percent;
}

static int64_t compute_balance_change(uint32_t deposit_cents, int32_t withdrawal_cents)
{
    return deposit_cents - withdrawal_cents;
}

static uint64_t convert_buffer_address(const unsigned char *buffer, size_t offset)
{
    return (uint64_t)(uintptr_t)(buffer + offset);
}
```

올바른 예:

```c
static int32_t *create_samples(size_t count)
{
    int32_t *samples = calloc(count, sizeof(*samples));

    return samples;
}

static int compute_percent(size_t done, size_t total)
{
    if (total == 0 || done > total || done > SIZE_MAX / 100) {
        return -1;
    }

    int percent = (int)((done * 100) / total);

    return percent;
}

static int64_t compute_balance_change(uint32_t deposit_cents, int32_t withdrawal_cents)
{
    return (int64_t)deposit_cents - withdrawal_cents;
}

static uint64_t convert_buffer_address(const unsigned char *buffer, size_t offset)
{
    return (uint64_t)((uintptr_t)(buffer + offset));
}
```

### 우선순위 괄호

- 비트 연산자와 시프트 연산자가 다른 연산자와 섞이면 소괄호로 묶는다.
- `&&`와 `||`가 섞이면 소괄호로 묶는다.
- `*`, `/`, `%`가 섞이거나 `/`·`%`가 이어지면 문법상 필요 없더라도 소괄호로 묶어 어느 연산이 먼저인지 드러낸다(`(done * 100) / total`, `(total / parts) / chunk_size`).
- 직관적인 이해, 덩어리 구분, 가독성을 위해 문법상 필요 없는 소괄호를 써도 된다. 문법상 필요 없다는 이유만으로 소괄호를 과감히 생략하는 것은 지양한다.

소괄호의 위치가 달라지면 식의 뜻도 달라진다. 캐스트는 `[]`, `->`, 함수 호출처럼 뒤에 붙는 연산자보다 나중에, 시프트 연산자 같은 이항 연산자보다는 먼저 적용된다. 아래 예에서 `pixels`는 픽셀마다 빨강, 초록, 파랑, 알파 값을 1바이트씩 차례로 담은 버퍼를 가리키는 `const uint8_t *`이다. 이때 다음 식은 모두 뜻이 다르다.

- `(uint32_t)pixels[2]`는 `pixels[2]`를 먼저 읽는다. 첨자 `2`는 2바이트 뒤를 뜻하므로, 첫 픽셀의 파랑 값 하나를 `uint32_t`로 바꾼다.
- `((const uint32_t *)pixels)[2]`는 포인터를 먼저 바꾼다. 첨자 `2`는 `uint32_t` 2개, 곧 8바이트 뒤를 뜻하므로, 세 번째 픽셀의 4바이트를 한꺼번에 읽는다. 바이트 버퍼를 이렇게 읽으면 strict aliasing 규칙을 어긴다. 주소가 `uint32_t`의 정렬 단위에 맞지 않으면 정렬 요구 사항도 어긴다.
- `(uint32_t)pixels[3] << 24`는 알파 값을 `uint32_t`로 바꾼 뒤 시프트한다. 반면 `(uint32_t)(pixels[3] << 24)`는 `int`로 승격된 값을 먼저 시프트한다. 따라서, 알파 값이 `0x80` 이상이면 시프트 결과가 `int`의 범위를 넘는다. 이런 시프트는 undefined behavior이다.

틀린 예:

```c
#define MODE_WRITE 0x2U

static int should_save(unsigned int mode, bool is_open, bool is_forced)
{
    return mode & MODE_WRITE != 0 && is_open || is_forced;
}

static unsigned int make_low_mask(size_t bits)
{
    if (bits >= sizeof(unsigned int) * CHAR_BIT) {
        return UINT_MAX;
    }

    return 1U << bits - 1;
}

static size_t compute_chunks(size_t free_bytes, size_t chunk_size)
{
    return free_bytes / 4 / chunk_size;
}
```

```c
static uint32_t read_first_blue(const uint8_t *pixels)
{
    return ((const uint32_t *)pixels)[2];
}

static uint32_t place_first_alpha(const uint8_t *pixels)
{
    return (uint32_t)(pixels[3] << 24);
}
```

지양하는 예:

```c
static size_t compute_offset(size_t header_size, size_t row, size_t row_stride, size_t column,
                             size_t cell_size)
{
    return header_size + row * row_stride + column * cell_size; // [!hl]
}
```

올바른 예:

```c
#define MODE_WRITE 0x2U

static int should_save(unsigned int mode, bool is_open, bool is_forced)
{
    return ((mode & MODE_WRITE) != 0 && is_open) || is_forced;
}

static unsigned int make_low_mask(size_t bits)
{
    if (bits >= sizeof(unsigned int) * CHAR_BIT) {
        return UINT_MAX;
    }

    return (1U << bits) - 1;
}

static size_t compute_chunks(size_t free_bytes, size_t chunk_size)
{
    return (free_bytes / 4) / chunk_size;
}
```

```c
static uint32_t read_first_blue(const uint8_t *pixels)
{
    return (uint32_t)pixels[2];
}

static uint32_t place_first_alpha(const uint8_t *pixels)
{
    return (uint32_t)pixels[3] << 24;
}
```

```c
static size_t compute_offset(size_t header_size, size_t row, size_t row_stride, size_t column,
                             size_t cell_size)
{
    return header_size + (row * row_stride) + (column * cell_size); // [!hl]
}
```

### 비교

- 상수를 왼쪽에 두는 비교(`NULL == p`)를 쓰지 않는다. `==`를 `=`로 잘못 쓰면 컴파일러가 경고한다.
- 부동소수점 값은 `==`로 비교하지 않고, 반복 변수로 쓰지 않는다.

틀린 예:

```c
static const double gs_tolerance = 1.0e-9;

void plot_point(double x);

static int matches_expected(const double *measured_nullable, double expected)
{
    if (NULL == measured_nullable) {
        return -1;
    }

    if (*measured_nullable == expected) {
        return 1;
    }

    return 0;
}

static void plot_unit_range(void)
{
    for (double x = 0.0; x != 1.0; x += 0.1) {
        plot_point(x);
    }
}
```

`0.1`은 2진수로 정확히 나타낼 수 없으므로, 열 번 더해도 정확히 `1.0`이 되지 않는다. 따라서, `plot_unit_range()`의 반복문은 끝나지 않는다.

올바른 예:

```c
static const double gs_tolerance = 1.0e-9;

void plot_point(double x);

static int matches_expected(const double *measured_nullable, double expected)
{
    if (measured_nullable == NULL) {
        return -1;
    }

    if (fabs(*measured_nullable - expected) < gs_tolerance) {
        return 1;
    }

    return 0;
}

static void plot_unit_range(void)
{
    for (size_t step = 0; step < 10; ++step) {
        plot_point((double)step * 0.1);
    }
}
```

### 널 포인터와 참 거짓 판정

- 널 포인터는 `NULL`로 쓰고, `0`으로 쓰지 않는다. `NULL`로 써야 그 값이 포인터임이 드러난다.
- 포인터는 `NULL`과, 정수는 `0`과 명시적으로 비교한다.
- `bool` 값만 그대로 조건에 쓴다.

틀린 예:

```c
static const char *pick_title(const char *title_nullable, size_t line_count, bool is_draft)
{
    if (title_nullable) {
        return title_nullable;
    }
    if (!line_count) {
        return 0;
    }
    if (is_draft == true) {
        return "untitled draft";
    }
    return "untitled";
}
```

올바른 예:

```c
static const char *pick_title(const char *title_nullable, size_t line_count, bool is_draft)
{
    if (title_nullable != NULL) {
        return title_nullable;
    }
    if (line_count == 0) {
        return NULL;
    }
    if (is_draft) {
        return "untitled draft";
    }
    return "untitled";
}
```

### 삼항 연산자

삼항 연산자는 값 하나를 고를 때만 쓴다. 조건은 문법상 필요 없어도 소괄호로 감싼다. 비교 하나뿐인 조건도 감싼다. 조건이 한 덩어리로 읽히게 하기 위해서이다.

삼항 연산자의 중첩은 지양한다. 굳이 중첩한다면 안쪽 삼항 연산자 식 전체를 소괄호로 감싼다. 중첩한 식이 한 줄에 들어가지 않으면, 안쪽 삼항 연산자의 결과를 별도 변수에 담는다.

지양하는 예:

```c
static const char *describe_count(size_t count)
{
    return (count == 0) ? "no files" : ((count == 1) ? "one file" : "several files");
}
```

틀린 예:

```c
static const char *pick_noun(size_t count)
{
    return count == 1 ? "file" : "files"; // [!hl]
}
```

```c
static const char *describe_count(size_t count)
{
    return count == 0 ? "no files" : count == 1 ? "one file" : "several files";
}
```

```c
void print_details(void);
void print_overview(void);

static void print_summary(bool is_verbose)
{
    (is_verbose) ? print_details() : print_overview();
}
```

```c
static const char *describe_copy(size_t file_count)
{
    const char *summary = (file_count == 0)
                                    ? "no files were copied"
                                    : ((file_count == 1) ? "one file was copied" : "several files were copied");

    return summary;
}
```

올바른 예:

```c
static const char *pick_noun(size_t count)
{
    return (count == 1) ? "file" : "files"; // [!hl]
}
```

```c
static const char *describe_count(size_t count)
{
    if (count == 0) {
        return "no files";
    }
    if (count == 1) {
        return "one file";
    }
    return "several files";
}
```

```c
void print_details(void);
void print_overview(void);

static void print_summary(bool is_verbose)
{
    if (is_verbose) {
        print_details();
    } else {
        print_overview();
    }
}
```

```c
static const char *describe_copy(size_t file_count)
{
    const char *copied = (file_count == 1) ? "one file was copied" : "several files were copied";
    const char *summary = (file_count == 0) ? "no files were copied" : copied;

    return summary;
}
```

### 할당

- 조건식 안에서 할당하지 않는다. 반복마다 값을 읽어 비교하는 반복문은 `for` 문으로 쓰고, 값을 읽는 식을 초기식과 증감식에 둔다. 조건식 안의 할당은 이렇게 바꿀 수 없을 때만 쓴다.
- 할당을 연쇄하지 않는다.

틀린 예:

```c
static long long count_bytes(const char *path)
{
    FILE *file;

    if ((file = fopen(path, "rb")) == NULL) {
        return -1;
    }

    long long total = 0;
    int c;

    while ((c = fgetc(file)) != EOF) {
        ++total;
    }

    if (ferror(file) != 0) {
        total = -1;
    }

    fclose(file);
    file = NULL;
    return total;
}
```

```c
struct range {
    int first;
    int last;
};

static void clear_range(struct range *mut_range)
{
    mut_range->first = mut_range->last = -1;
}
```

올바른 예:

```c
static long long count_bytes(const char *path)
{
    FILE *file = fopen(path, "rb");
    if (file == NULL) {
        return -1;
    }

    long long total = 0;

    for (int c = fgetc(file); c != EOF; c = fgetc(file)) {
        ++total;
    }

    if (ferror(file) != 0) {
        total = -1;
    }

    fclose(file);
    file = NULL;
    return total;
}
```

```c
struct range {
    int first;
    int last;
};

static void clear_range(struct range *mut_range)
{
    mut_range->first = -1;
    mut_range->last = -1;
}
```

### 쉼표 연산자

쉼표 연산자는 쓰지 않는다. 다만 `for` 문에서 변수 두 개 이상을 함께 다룰 때는 초기식과 증감식에 쓸 수 있다. 이런 `for` 문이 한 줄에 들어가지 않으면 `;` 뒤에서 나누고, 다음 줄은 `for`의 여는 소괄호 바로 다음 열에서 시작한다.

틀린 예:

```c
static void add_sample(long *mut_total, size_t *mut_count, int value)
{
    *mut_total += value, *mut_count += 1;
}
```

```c
struct cursor {
    size_t row;
    size_t column;
};

static void reset_cursor(struct cursor *mut_cursor)
{
    mut_cursor->row = 0, mut_cursor->column = 0;
}
```

올바른 예:

```c
static void add_sample(long *mut_total, size_t *mut_count, int value)
{
    *mut_total += value;
    *mut_count += 1;
}
```

```c
struct cursor {
    size_t row;
    size_t column;
};

static void reset_cursor(struct cursor *mut_cursor)
{
    mut_cursor->row = 0;
    mut_cursor->column = 0;
}
```

```c
static void reverse_text(char *mut_text, size_t length)
{
    if (length == 0) {
        return;
    }

    for (size_t left = 0, right = length - 1; left < right; ++left, --right) {
        char saved = mut_text[left];

        mut_text[left] = mut_text[right];
        mut_text[right] = saved;
    }
}
```

```c
static void copy_to_free_slots(int *mut_slots, size_t capacity, const int *sources,
                               size_t source_count, size_t first_free_slot)
{
    for (size_t source = 0, slot = first_free_slot; source < source_count && slot < capacity;
         ++source, ++slot) {
        mut_slots[slot] = sources[source];
    }
}
```

## 문장

### switch

- 열거형의 모든 값을 다루는 `switch`에는 `default`를 두지 않는다. 열거형에 새로 더한 enumeration constant의 `case`를 빠뜨리면 `-Wswitch`가 경고한다. `default`가 있으면 이 경고가 나오지 않는다.
- 그 밖의 모든 `switch`에는 `default`를 둔다.
- 모든 `case`는 `break`, `return`, `goto`, `continue` 가운데 하나로 끝낸다.
- 의도한 fallthrough에는 `/* fallthrough */` 주석을 단다. GCC의 `-Wimplicit-fallthrough`가 이 주석을 인식한다.

틀린 예:

```c
enum shape {
    SHAPE_TRIANGLE = 0,
    SHAPE_SQUARE,
    SHAPE_PENTAGON,
};

static size_t count_corners(enum shape shape)
{
    switch (shape) {
    case SHAPE_TRIANGLE:
        return 3;
    case SHAPE_SQUARE:
        return 4;
    case SHAPE_PENTAGON:
        return 5;
    default:
        return 0;
    }
}
```

```c
#define RIGHT_VIEW 0x1U
#define RIGHT_EDIT 0x2U
#define RIGHT_MANAGE 0x4U

static unsigned int grant_rights(char role)
{
    unsigned int rights = 0;

    switch (role) {
    case 'a':
        rights |= RIGHT_MANAGE;
    case 'e':
        rights |= RIGHT_EDIT;
    case 'v':
        rights |= RIGHT_VIEW;
        break;
    }

    return rights;
}
```

올바른 예:

```c
enum shape {
    SHAPE_TRIANGLE = 0,
    SHAPE_SQUARE,
    SHAPE_PENTAGON,
};

static size_t count_corners(enum shape shape)
{
    switch (shape) {
    case SHAPE_TRIANGLE:
        return 3;
    case SHAPE_SQUARE:
        return 4;
    case SHAPE_PENTAGON:
        return 5;
    }

    return 0;
}
```

```c
#define RIGHT_VIEW 0x1U
#define RIGHT_EDIT 0x2U
#define RIGHT_MANAGE 0x4U

static unsigned int grant_rights(char role)
{
    unsigned int rights = 0;

    switch (role) {
    case 'a':
        rights |= RIGHT_MANAGE;
        /* fallthrough */
    case 'e':
        rights |= RIGHT_EDIT;
        /* fallthrough */
    case 'v':
        rights |= RIGHT_VIEW;
        break;
    default:
        break;
    }

    return rights;
}
```

### 빈 반복문 본문

본문이 빈 반복문은 되도록 피한다. 조건 안에 넣은 할당, 증감 같은 연산과 함수 호출은 본문이나 `for` 문의 초기식과 증감식으로 옮긴다. 다만 그 작업이 반드시 조건 안에 있어야 하면 그대로 둔다.

본문을 비워 둔 반복문은 중괄호 안에 `;`만 있는 줄을 하나 둔다. 본문을 일부러 비웠음을 드러내기 위해서이다.

틀린 예:

```c
static size_t measure_text(const char *text)
{
    const char *cursor = text;

    while (*cursor++ != '\0') {
        ;
    }

    return (size_t)(cursor - text - 1);
}
```

```c
static void echo_line(FILE *mut_output, FILE *mut_input)
{
    int c;

    while ((c = fgetc(mut_input)) != EOF && fputc(c, mut_output) != '\n') {
        ;
    }
}
```

틀린 예(빈 본문에 `;`를 넣지 않은 경우):

```c
#define ADC_STATUS (*(volatile const uint32_t *)0x40012000UL)
#define ADC_CONVERSION_DONE 0x2U

static void wait_for_conversion(void)
{
    while ((ADC_STATUS & ADC_CONVERSION_DONE) == 0) {
    }
}
```

올바른 예:

```c
static size_t measure_text(const char *text)
{
    size_t length = 0;

    while (text[length] != '\0') {
        ++length;
    }

    return length;
}
```

```c
static void echo_line(FILE *mut_output, FILE *mut_input)
{
    for (int c = fgetc(mut_input); c != EOF; c = fgetc(mut_input)) {
        fputc(c, mut_output);
        if (c == '\n') {
            break;
        }
    }
}
```

올바른 예(작업이 반드시 조건 안에 있어야 하는 경우):

```c
#define ADC_STATUS (*(volatile const uint32_t *)0x40012000UL)
#define ADC_CONVERSION_DONE 0x2U

static void wait_for_conversion(void)
{
    while ((ADC_STATUS & ADC_CONVERSION_DONE) == 0) {
        ;
    }
}
```

### goto

goto는 다음 경우에 쓴다.

- 이중 이상으로 중첩된 제어문이나 반복문을 한 번에 빠져나갈 때
- 단계별 초기화가 중간에 실패했을 때. 1단계부터 N단계까지 차례로 초기화하다가 어느 단계에서 실패하면, 이미 마친 초기화를 그 직전 단계부터 1단계까지 거꾸로 해제한다. 정리할 자원이 있으면 함수 끝에 정리 레이블을 두고 `goto`로 점프한다. 레이블은 자원을 얻은 순서의 역순으로 둔다.

뒤로 점프하는 goto도 쓸 수 있다. 다만 그런 구조를 애초에 만들지 않도록 로직을 짜는 쪽을 지향한다.

지양하는 예:

```c
int read_sensor(int *mut_value);
void wait_briefly(void);

static int read_sensor_with_retry(int *mut_value)
{
    size_t attempts = 0;

retry:
    if (read_sensor(mut_value) != 0) {
        ++attempts;
        if (attempts < 3) {
            wait_briefly();
            goto retry;
        }
        return -1;
    }

    return 0;
}
```

올바른 예:

```c
#define GRID_SIZE 8

struct grid {
    pthread_mutex_t lock;
    int cells[GRID_SIZE][GRID_SIZE];
};

static int find_cell(size_t *mut_row, size_t *mut_column, struct grid *mut_grid,
                     int target)
{
    int result = -1;

    pthread_mutex_lock(&mut_grid->lock);
    for (size_t row = 0; row < GRID_SIZE; ++row) {
        for (size_t column = 0; column < GRID_SIZE; ++column) {
            if (mut_grid->cells[row][column] == target) {
                *mut_row = row;
                *mut_column = column;
                result = 0;
                goto out_unlock;
            }
        }
    }

out_unlock:
    pthread_mutex_unlock(&mut_grid->lock);
    return result;
}
```

```c
#define BMP_HEADER_SIZE 54

struct image {
    FILE *file;
    unsigned char *header;
    unsigned char *pixels;
};

static int open_image(struct image *mut_image, const char *path, size_t pixel_bytes)
{
    mut_image->file = fopen(path, "rb");
    if (mut_image->file == NULL) {
        return -1;
    }

    mut_image->header = malloc(BMP_HEADER_SIZE);
    if (mut_image->header == NULL) {
        goto handle_header_alloc;
    }

    mut_image->pixels = malloc(pixel_bytes);
    if (mut_image->pixels == NULL) {
        goto handle_pixels_alloc;
    }

    return 0;

handle_pixels_alloc:
    free(mut_image->header);
    mut_image->header = NULL;
handle_header_alloc:
    fclose(mut_image->file);
    mut_image->file = NULL;
    return -1;
}
```

```c
int read_sensor(int *mut_value);
void wait_briefly(void);

static int read_sensor_with_retry(int *mut_value)
{
    for (size_t attempts = 0; attempts < 3; ++attempts) {
        if (attempts > 0) {
            wait_briefly();
        }
        if (read_sensor(mut_value) == 0) {
            return 0;
        }
    }

    return -1;
}
```

## 함수

### 매개변수 목록

매개변수가 없는 함수는 소괄호를 비우지 않고 `(void)`로 선언한다.

함수 선언에는 모든 매개변수의 이름을 쓴다. 선언과 정의의 매개변수 이름은 같아야 한다.

틀린 예:

```c
static int gs_volume;

int read_volume();
int set_volume(int);

int read_volume()
{
    return gs_volume;
}

int set_volume(int volume)
{
    if (volume < 0 || volume > 100) {
        return -1;
    }

    gs_volume = volume;
    return 0;
}
```

올바른 예:

```c
static int gs_volume;

int read_volume(void);
int set_volume(int volume);

int read_volume(void)
{
    return gs_volume;
}

int set_volume(int volume)
{
    if (volume < 0 || volume > 100) {
        return -1;
    }

    gs_volume = volume;
    return 0;
}
```

### 매개변수 순서

결과를 써 넣을 출력 매개변수는 가능하면 앞에 두는 것을 지향한다. `memcpy`와 같은 순서이다.

지양하는 예:

```c
struct date {
    int year;
    int month;
    int day;
};

static int format_date(const struct date *date, char *mut_buffer, size_t buffer_size) // [!hl]
{
    return snprintf(mut_buffer, buffer_size, "%04d-%02d-%02d", date->year, date->month, date->day);
}
```

올바른 예:

```c
struct date {
    int year;
    int month;
    int day;
};

static int format_date(char *mut_buffer, size_t buffer_size, const struct date *date) // [!hl]
{
    return snprintf(mut_buffer, buffer_size, "%04d-%02d-%02d", date->year, date->month, date->day);
}
```

### 포인터 매개변수

포인터 매개변수는 다음 규칙을 따른다.

- 매개변수 자체에는 `const`를 붙이지 않는다. 즉, `*const`를 쓰지 않고, typedef로 정의한 포인터 타입 앞에도 `const`를 붙이지 않는다(`HANDLE file`). 이 `const`는 호출하는 쪽에 아무 효과가 없기 때문이다(C99 §6.7.5.3 ¶15).
- 값 매개변수든 포인터 매개변수든, 매개변수 자체를 함수 안에서 바꾸는 것은 최대한 지양한다. 넘겨받은 값이나 주소와 다른 값이 필요하면 첨자 연산이나 지역 변수처럼 매개변수를 바꾸지 않는 수단으로 얻는다.
- 함수가 대상을 바꾸지 않으면 `const T *name`으로 선언한다.
- 함수가 대상을 바꾸면 `T *mut_name`으로 선언한다. 이름은 `mut_`으로 시작한다.
- 매개변수가 포인터(혹은 문자열, 배열)이고, 그 값에 `NULL`이 들어갈 수 있고, 그래도 문제가 없는 경우에만 이름을 `_nullable`로 끝낸다(`const T *name_nullable`, `T *mut_name_nullable`). `NULL`이 들어가면 문제가 되는 매개변수에는 이 접미어를 붙이지 않는다.
- `main`의 매개변수는 표준이 정한 형태를 그대로 쓴다.

함수의 선언과 정의에는 같은 형태를 쓴다. 그러면 선언만 보고도 함수가 포인터 인자의 대상을 바꾸는지, 인자로 `NULL`을 받아도 되는지 알 수 있다.

지양하는 예(매개변수 자체를 바꾸는 경우):

```c
static size_t count_digits(const char *text)
{
    size_t count = 0;

    for (; *text != '\0'; ++text) {
        if (isdigit((unsigned char)*text) != 0) {
            ++count;
        }
    }

    return count;
}
```

올바른 예(매개변수 자체를 바꾸지 않는 경우):

```c
static size_t count_digits(const char *text)
{
    size_t count = 0;

    for (size_t i = 0; text[i] != '\0'; ++i) {
        if (isdigit((unsigned char)text[i]) != 0) {
            ++count;
        }
    }

    return count;
}
```

```c
static size_t count_digits(const char *text)
{
    size_t count = 0;

    for (const char *cursor = text; *cursor != '\0'; ++cursor) {
        if (isdigit((unsigned char)*cursor) != 0) {
            ++count;
        }
    }

    return count;
}
```

틀린 예:

```c
static void convert_to_upper(char *text)
{
    for (size_t i = 0; text[i] != '\0'; ++i) {
        if (text[i] >= 'a' && text[i] <= 'z') {
            text[i] = (char)(text[i] - 'a' + 'A');
        }
    }
}

size_t count_vowels(char *text);
```

```c
static void print_label(const char *label)
{
    if (label == NULL) {
        puts("(none)");
        return;
    }

    puts(label);
}
```

올바른 예:

```c
size_t count_nonzero(const int *values, size_t count);
void clear_values(int *mut_values, size_t count);
void clear_values_if_present(int *mut_values_nullable, size_t count);
```

```c
static void convert_to_upper(char *mut_text)
{
    for (size_t i = 0; mut_text[i] != '\0'; ++i) {
        if (mut_text[i] >= 'a' && mut_text[i] <= 'z') {
            mut_text[i] = (char)(mut_text[i] - 'a' + 'A');
        }
    }
}

size_t count_vowels(const char *text);
```

```c
static void print_label(const char *label_nullable)
{
    if (label_nullable == NULL) {
        puts("(none)");
        return;
    }

    puts(label_nullable);
}
```

### 반환값 규약

- 일을 하고 성공 여부만 알리는 함수는 성공하면 `0`, 실패하면 음수를 돌려주는 것을 우선한다. 다만 이 규약은 그 함수를 부르는 상위 API의 설계에 따라 달라질 수 있다. 상위 API가 정한 규약이 있으면 그 규약에 맞춘다.
- 실패할 수 없는 함수는 `void`를 돌려준다.
- 참 거짓을 돌려주는 함수는 `int`나 `bool`을 돌려준다.
- 값을 계산해 돌려주는 함수는 범위 밖 값(`NULL`, `-1`)으로 실패를 알린다.

`int`를 돌려주면 거짓인 이유를 값으로 구별할 수 있다. 예를 들어 음수 전체를 거짓에 대응시키면, 음수 값마다 거짓인 이유를 따로 나타낼 수 있다. 또 Linux API에서는 많은 값이 `int`로 오간다. Linux API의 규약을 따르는 코드에서는 `int`가 자연스러운 경우가 많다.

틀린 예:

```c
struct profile;
struct statistics;

bool load_profile(const char *path);
int reset_statistics(struct statistics *mut_statistics);
bool is_profile_loaded(void);
int is_cache_warm(void);
struct profile *find_profile(const char *name);
long parse_count(const char *text);
```

올바른 예:

```c
struct profile;
struct statistics;

int load_profile(const char *path);
void reset_statistics(struct statistics *mut_statistics);
bool is_profile_loaded(void);
int is_cache_warm(void);
struct profile *find_profile(const char *name);
long parse_count(const char *text);
```

올바른 예(상위 API가 정한 규약을 따르는 경우):

```c
struct task {
    char name[32];
    int priority;
};

static int compare_priority(const void *left, const void *right)
{
    const struct task *first = left;
    const struct task *second = right;

    if (first->priority < second->priority) {
        return -1;
    }
    if (first->priority > second->priority) {
        return 1;
    }
    return 0;
}

static void sort_tasks(struct task *mut_tasks, size_t count)
{
    qsort(mut_tasks, count, sizeof(*mut_tasks), compare_priority);
}
```

### 함수의 크기

함수의 길이와 지역 변수 개수에는 상한을 두지 않는다. 함수 하나가 여러 일을 맡아도 된다. 다만 지역 변수는 스택에 놓이므로, 함수가 쓰는 스택 메모리의 크기는 고려한다. 스택은 스레드와 실행 환경마다 크기가 제한되어 있다. 따라서, 크기가 큰 배열이나 구조체는 동적 할당이나, 외부 매개변수로 포인터를 전달하는 방법을 강구하도록 권장한다.

지양하는 예:

```c
#define COPY_BUFFER_SIZE (1024 * 1024)

static int copy_file(FILE *mut_dst, FILE *mut_src)
{
    unsigned char buffer[COPY_BUFFER_SIZE];

    for (size_t read_size = fread(buffer, 1, sizeof(buffer), mut_src); read_size > 0;
         read_size = fread(buffer, 1, sizeof(buffer), mut_src)) {
        if (fwrite(buffer, 1, read_size, mut_dst) != read_size) {
            return -1;
        }
    }

    return (ferror(mut_src) != 0) ? -1 : 0;
}
```

올바른 예:

```c
#define COPY_BUFFER_SIZE (1024 * 1024)

static int copy_file(FILE *mut_dst, FILE *mut_src)
{
    unsigned char *buffer = malloc(COPY_BUFFER_SIZE);
    if (buffer == NULL) {
        return -1;
    }

    int result = -1;

    for (size_t read_size = fread(buffer, 1, COPY_BUFFER_SIZE, mut_src); read_size > 0;
         read_size = fread(buffer, 1, COPY_BUFFER_SIZE, mut_src)) {
        if (fwrite(buffer, 1, read_size, mut_dst) != read_size) {
            goto out_free_buffer;
        }
    }

    if (ferror(mut_src) == 0) {
        result = 0;
    }

out_free_buffer:
    free(buffer);
    buffer = NULL;
    return result;
}
```

올바른 예(호출하는 쪽에서 버퍼를 받는 경우):

```c
static int copy_file(FILE *mut_dst, FILE *mut_src, unsigned char *mut_buffer,
                     size_t buffer_size)
{
    for (size_t read_size = fread(mut_buffer, 1, buffer_size, mut_src); read_size > 0;
         read_size = fread(mut_buffer, 1, buffer_size, mut_src)) {
        if (fwrite(mut_buffer, 1, read_size, mut_dst) != read_size) {
            return -1;
        }
    }

    return (ferror(mut_src) != 0) ? -1 : 0;
}
```

### inline

`inline`은 실행 시간이 짧은 코드 조각에 붙일지 검토한다. 여러 곳에서 되풀이해 쓰지만 독립된 함수로 이름을 붙이기는 어색하고, 매크로로 쓰기에는 문법, 코드 탐색과 읽기, IDE의 코드 분석 같은 면에서 아쉬운 조각이 그 대상이다. 줄 수가 적어도 실행 시간이 긴 함수에는 붙이지 않는다. 실제로 인라인할지는 컴파일러가 정한다.

형태는 `static inline`을 기본으로 하되, `static`을 강제하지는 않는다. `static` 없이 `inline`만 붙인 함수의 정의는 C99 §6.7.4 ¶6에 따라 inline definition이다. 이 정의만으로는 external definition이 생기지 않는다. 따라서, 이런 함수는 `.c` 파일 하나에 `extern inline` 선언을 두어 external definition을 만든다. 그러지 않으면 컴파일러가 인라인하지 않은 호출에서 링크가 실패한다. 예를 들어 GCC와 Clang에서 `-std=gnu11 -O0`으로 빌드하면 링크 단계에서 `undefined reference` 오류가 난다. 반대로 GNU89 모드에서는 헤더를 포함한 파일마다 external definition이 생겨 `multiple definition` 오류가 난다.

지양하는 예(`util/listing.h`):

```c
#ifndef UTIL_LISTING_H
#define UTIL_LISTING_H

#include <stddef.h>
#include <stdio.h>

static inline void print_table(const int *values, size_t count)
{
    for (size_t i = 0; i < count; ++i) {
        printf("%zu: %d\n", i, values[i]);
    }
}

#endif /* UTIL_LISTING_H */
```

올바른 예(`util/listing.h`):

```c
#ifndef UTIL_LISTING_H
#define UTIL_LISTING_H

#include <stddef.h>

void print_table(const int *values, size_t count);

#endif /* UTIL_LISTING_H */
```

올바른 예(`util/color.h`):

```c
#ifndef UTIL_COLOR_H
#define UTIL_COLOR_H

#include <stdint.h>

static inline uint8_t blend_channel(uint8_t back, uint8_t front, uint8_t alpha)
{
    return (uint8_t)((front * alpha + back * (255 - alpha)) / 255);
}

#endif /* UTIL_COLOR_H */
```

올바른 예(`util/clamp.h`):

```c
#ifndef UTIL_CLAMP_H
#define UTIL_CLAMP_H

inline int clamp_percent(int value)
{
    if (value < 0) {
        return 0;
    }
    if (value > 100) {
        return 100;
    }
    return value;
}

#endif /* UTIL_CLAMP_H */
```

올바른 예(`util/clamp.c`):

```c
#include "util/clamp.h"

extern inline int clamp_percent(int value);
```

다만 C89를 고른 프로젝트는 예외로 한다. C89에는 `inline`이 없으므로 헤더에 함수를 정의하지 않는다. 함수는 `.c` 파일에 정의하고, 헤더에는 선언만 둔다.

C89 프로젝트의 올바른 예(`util/color.h`):

```c
#ifndef UTIL_COLOR_H
#define UTIL_COLOR_H

unsigned char blend_channel(unsigned char back, unsigned char front, unsigned char alpha);

#endif /* UTIL_COLOR_H */
```

### 함수 배치

- static 함수는 같은 파일에서 그 함수를 호출하는 함수보다 위에 둔다. 전방 선언은 상호 재귀처럼 특수한 경우에만 쓴다.
- `main`은 파일의 마지막 함수이다.
- 선언은 storage-class specifier, `inline`, 반환형, 이름 순서로 쓴다.

틀린 예:

```c
#include <stdio.h>

static void print_banner(const char *title);

int main(void)
{
    print_banner("inventory");
    return 0;
}

void static print_banner(const char *title)
{
    printf("== %s ==\n", title);
}
```

올바른 예:

```c
#include <stdio.h>

static void print_banner(const char *title)
{
    printf("== %s ==\n", title);
}

int main(void)
{
    print_banner("inventory");
    return 0;
}
```

올바른 예(서로 부르는 함수):

```c
static int parse_sum(double *mut_sum, const char **mut_cursor);

static int parse_factor(double *mut_value, const char **mut_cursor)
{
    if (**mut_cursor == '(') {
        *mut_cursor += 1;
        if (parse_sum(mut_value, mut_cursor) != 0 || **mut_cursor != ')') {
            return -1;
        }

        *mut_cursor += 1;
        return 0;
    }

    char *end = NULL;

    errno = 0;
    *mut_value = strtod(*mut_cursor, &end);
    if (end == *mut_cursor || errno == ERANGE) {
        return -1;
    }

    *mut_cursor = end;
    return 0;
}

static int parse_sum(double *mut_sum, const char **mut_cursor)
{
    if (parse_factor(mut_sum, mut_cursor) != 0) {
        return -1;
    }

    while (**mut_cursor == '+') {
        *mut_cursor += 1;

        double value = 0.0;

        if (parse_factor(&value, mut_cursor) != 0) {
            return -1;
        }

        *mut_sum += value;
    }

    return 0;
}
```

### main

- `main`은 `int main(void)` 또는 `int main(int argc, char *argv[])` 형태로 쓴다.
- 성공하면 `return 0;`, 실패하면 `return EXIT_FAILURE;`로 끝낸다.

틀린 예:

```c
void main()
{
    puts("ready");
    exit(0);
}
```

올바른 예:

```c
int main(void)
{
    puts("ready");
    return 0;
}
```

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[])
{
    if (argc != 2) {
        const char *program = (argc > 0) ? argv[0] : "forecast";

        fprintf(stderr, "usage: %s <city>\n", program);
        return EXIT_FAILURE;
    }

    printf("forecast for %s\n", argv[1]);
    return 0;
}
```

## 전처리기와 매크로

### 조건부 컴파일

- 어떤 파일을 컴파일할지는 빌드 시스템이 정한다. 소스 안의 `#if`는 구현이 실제로 달라지는 곳에만 둔다.
- 운영체제 이름보다 기능을 기준으로 분기한다. 아키텍처는 컴파일러가 정의한 매크로로 판별한다.
- `#error`는 "이 환경에서는 올바를 수 없다"는 뜻으로만 쓴다. 알 수 없는 플랫폼은 `unknown` 같은 기본값으로 처리한다.
- 20줄이 넘는 조건부 컴파일 블록은 `#endif` 뒤에 조건을 주석으로 쓴다.

틀린 예(`src/sleep_win32.c`):

```c
#if defined(_WIN32)

#include <windows.h>

static void sleep_ms(unsigned int milliseconds)
{
    Sleep(milliseconds);
}

#endif
```

```c
#define _POSIX_C_SOURCE 200809L

#include <fcntl.h>

static int open_for_reading(const char *path)
{
#if defined(__linux__) // [!hl]
    return open(path, O_RDONLY | O_CLOEXEC);
#else
    return open(path, O_RDONLY);
#endif
}
```

```c
#if defined(__x86_64__) || defined(_M_X64)
#define TARGET_ARCH_NAME "x86_64"
#elif defined(__aarch64__) || defined(_M_ARM64)
#define TARGET_ARCH_NAME "aarch64"
#else
#error "unsupported architecture" // [!hl]
#endif
```

```c
int clamp_percent(int value);
size_t count_words(const char *text);

#if defined(ENABLE_SELF_TEST)
static int run_self_test(void)
{
    size_t failures = 0;

    if (clamp_percent(-5) != 0) {
        ++failures;
    }
    if (clamp_percent(50) != 50) {
        ++failures;
    }
    if (clamp_percent(150) != 100) {
        ++failures;
    }
    if (count_words("one two  three") != 3) {
        ++failures;
    }
    if (count_words("") != 0) {
        ++failures;
    }

    printf("self test: %zu failures\n", failures);
    return (failures == 0) ? 0 : -1;
}
#endif // [!hl]
```

올바른 예(`CMakeLists.txt`):

```cmake
if(WIN32)
    target_sources(app PRIVATE src/sleep_win32.c)
else()
    target_sources(app PRIVATE src/sleep_posix.c)
endif()
```

```c
#define _POSIX_C_SOURCE 200809L

#include <fcntl.h>

static int open_for_reading(const char *path)
{
#if defined(O_CLOEXEC) // [!hl]
    return open(path, O_RDONLY | O_CLOEXEC);
#else
    return open(path, O_RDONLY);
#endif
}
```

```c
#if defined(__x86_64__) || defined(_M_X64)
#define TARGET_ARCH_NAME "x86_64"
#elif defined(__aarch64__) || defined(_M_ARM64)
#define TARGET_ARCH_NAME "aarch64"
#else
#define TARGET_ARCH_NAME "unknown" // [!hl]
#endif
```

```c
#include <limits.h>

#if CHAR_BIT != 8
#error "this code reads files as 8-bit bytes"
#endif
```

```c
int clamp_percent(int value);
size_t count_words(const char *text);

#if defined(ENABLE_SELF_TEST)
static int run_self_test(void)
{
    size_t failures = 0;

    if (clamp_percent(-5) != 0) {
        ++failures;
    }
    if (clamp_percent(50) != 50) {
        ++failures;
    }
    if (clamp_percent(150) != 100) {
        ++failures;
    }
    if (count_words("one two  three") != 3) {
        ++failures;
    }
    if (count_words("") != 0) {
        ++failures;
    }

    printf("self test: %zu failures\n", failures);
    return (failures == 0) ? 0 : -1;
}
#endif /* defined(ENABLE_SELF_TEST) */ // [!hl]
```

### 매크로를 대신할 수단

함수처럼 동작하는 코드에는 매크로보다 `static inline` 함수를 검토한다. 매크로는 타입과 무관하게 동작해야 하거나, `#`, `##`, `__FILE__`처럼 전처리기가 필요할 때 쓴다.

틀린 예:

```c
#define PERCENT_OF(part, whole) ((int)(((long long)(part) * 100) / (whole)))
```

올바른 예:

```c
static inline int compute_percent_of(int part, int whole)
{
    return (int)(((long long)part * 100) / whole);
}
```

```c
#define MEMBER_SIZE(type, member) sizeof(((type *)0)->member)
#define PRINT_INT(expr) printf("%s = %d\n", #expr, (expr))
#define GLUE(prefix, name) prefix##name
#define TRACE(message) fprintf(stderr, "%s:%d: %s\n", __FILE__, __LINE__, (message))
```

### 괄호

매크로 본문이 식이면 본문 전체를 괄호로 감싼다. 인자는 쓰는 곳마다 괄호로 감싼다. 리터럴 하나뿐인 본문은 괄호로 감싸지 않는다.

틀린 예:

```c
#define MAX_SEATS (120)
#define SEATS_PER_ROW MAX_SEATS / 10
#define AVERAGE(a, b) a + b / 2
```

올바른 예:

```c
#define MAX_SEATS 120
#define SEATS_PER_ROW (MAX_SEATS / 10)
#define AVERAGE(a, b) (((a) + (b)) / 2)
```

### 여러 문장 매크로

문장 여러 개로 펼쳐지는 매크로는 `do { ... } while (0)`로 감싼다. 그래야 `if` 본문에서도 문장 하나처럼 동작한다.

틀린 예:

```c
#define REPORT_FAILURE(what)                \
    fprintf(stderr, "%s failed\n", (what)); \
    ++gs_failure_count

static size_t gs_failure_count;
```

올바른 예:

```c
#define REPORT_FAILURE(what)                    \
    do {                                        \
        fprintf(stderr, "%s failed\n", (what)); \
        ++gs_failure_count;                     \
    } while (0)

static size_t gs_failure_count;
```

### 피해야 할 매크로

다음 매크로는 쓰지 않는다.

- 매크로를 쓴 함수에서 빠져나가는 등 제어 흐름을 바꾸는 매크로
- 특정 이름의 지역 변수가 있다고 가정하는 매크로
- 할당의 왼쪽에 쓰는 매크로
- 문법을 바꾸는 매크로

틀린 예:

```c
#define RETURN_IF_FAILED(expr) \
    do {                       \
        if ((expr) < 0) {      \
            return -1;         \
        }                      \
    } while (0)

int save_file(const char *path);

static int save_both(const char *first_path, const char *second_path)
{
    RETURN_IF_FAILED(save_file(first_path));
    RETURN_IF_FAILED(save_file(second_path));
    return 0;
}
```

```c
#define PRINT_PROGRESS() printf("%zu/%zu done\n", done, total)

static void process_jobs(size_t total)
{
    for (size_t done = 1; done <= total; ++done) {
        PRINT_PROGRESS();
    }
}
```

```c
#define LAST_SCORE(board) ((board)->scores[(board)->count - 1])

struct scoreboard {
    int scores[16];
    size_t count;
};

static void reset_last_score(struct scoreboard *mut_board)
{
    if (mut_board->count > 0) {
        LAST_SCORE(mut_board) = 0;
    }
}
```

```c
#define BEGIN {
#define END }

static size_t count_negatives(const int *values, size_t count)
BEGIN
    size_t negatives = 0;

    for (size_t i = 0; i < count; ++i) BEGIN
        if (values[i] < 0) BEGIN
            ++negatives;
        END
    END

    return negatives;
END
```

올바른 예:

```c
int save_file(const char *path);

static int save_both(const char *first_path, const char *second_path)
{
    if (save_file(first_path) < 0) {
        return -1;
    }
    if (save_file(second_path) < 0) {
        return -1;
    }
    return 0;
}
```

```c
static void print_progress(size_t done, size_t total)
{
    printf("%zu/%zu done\n", done, total);
}

static void process_jobs(size_t total)
{
    for (size_t done = 1; done <= total; ++done) {
        print_progress(done, total);
    }
}
```

```c
struct scoreboard {
    int scores[16];
    size_t count;
};

static void reset_last_score(struct scoreboard *mut_board)
{
    if (mut_board->count > 0) {
        mut_board->scores[mut_board->count - 1] = 0;
    }
}
```

```c
static size_t count_negatives(const int *values, size_t count)
{
    size_t negatives = 0;

    for (size_t i = 0; i < count; ++i) {
        if (values[i] < 0) {
            ++negatives;
        }
    }

    return negatives;
}
```

### 매크로 안의 지역 이름

매크로 안에서 지역 변수를 선언해야 하면 이름 끝에 밑줄 하나(`_`)를 붙인다(`a_`). 그래야 매크로를 호출하는 쪽의 이름과 겹치지 않는다. 이름 앞에는 밑줄 두 개(`__`)를 붙이지 않는다(`__a`). 밑줄 두 개로 시작하는 이름은 예약된 식별자이기 때문이다.

틀린 예:

```c
#define MIN_OF(a, b)             \
    __extension__({              \
        __typeof__(a) __a = (a); \
        __typeof__(b) __b = (b); \
        (__a < __b) ? __a : __b; \
    })
```

올바른 예:

```c
#define MIN_OF(a, b)            \
    __extension__({             \
        __typeof__(a) a_ = (a); \
        __typeof__(b) b_ = (b); \
        (a_ < b_) ? a_ : b_;    \
    })
```

## 주석

### 주석의 양과 내용

주석은 꼭 필요할 때만 쓴다. 먼저 이름과 구조를 고쳐 코드만으로 뜻이 드러나게 한다. 주석에는 무엇을 왜 하는지 쓰고, 어떻게 하는지는 코드에 맡긴다.

코드만 읽어서는 이해할 수 없는 곳에는 다음 내용을 주석으로 적는다.

- 코드를 이해하는 데 필요한 배경 지식
- 그 동작이 꼭 필요한 이유
- 언뜻 쓸데없어 보이는 구현을 택한 이유

주석을 길게 쓰는 곳은 외부에 공개하는 API, 라이브러리, 인터페이스 역할을 하는 코드와 교육용 코드이다. 이런 코드에는 내부 동작, 인자, 반환값, 경우별 시나리오 등을 자세히 적는다.

틀린 예:

```c
/* 86400 */
#define SECONDS_PER_DAY 86400
```

```c
static int publish_file(const char *temp_path, const char *final_path)
{
    /* rename the temporary file */ // [!hl]
    if (rename(temp_path, final_path) != 0) {
        return -1;
    }

    return 0;
}
```

```c
static int is_valid_index(int index, size_t count)
{
    /* cast index */ // [!hl]
    return (size_t)index < count;
}
```

올바른 예:

```c
/* POSIX time counts every day as exactly 86400 seconds, so leap seconds are not counted */
#define SECONDS_PER_DAY 86400
```

```c
static int publish_file(const char *temp_path, const char *final_path)
{
    /* readers see either the old file or the complete new one, never a half-written file */ // [!hl]
    if (rename(temp_path, final_path) != 0) {
        return -1;
    }

    return 0;
}
```

```c
static int is_valid_index(int index, size_t count)
{
    /* a negative index wraps to a large unsigned value, so one comparison rejects it too */ // [!hl]
    return (size_t)index < count;
}
```

올바른 예(교육용 코드):

```c
/*
 * Binary search keeps the answer inside [low, high). Each step compares the
 * middle element with the key and drops the half that cannot contain it, so a
 * sorted array of n elements needs about log2(n) steps instead of n.
 */
static int find_sorted(size_t *mut_index, const int *values, size_t count, int key)
{
    size_t low = 0;
    size_t high = count;

    while (low < high) {
        size_t middle = low + (high - low) / 2;

        if (values[middle] < key) {
            low = middle + 1;
        } else if (values[middle] > key) {
            high = middle;
        } else {
            *mut_index = middle;
            return 0;
        }
    }

    return -1;
}
```

### 주석 형식

- `/* */`만 쓴다. `//`는 쓰지 않는다.
- 여러 줄 주석은 `/*` 줄, ` * `로 시작하는 줄, `*/` 줄로 쓴다.
- 줄 끝 주석은 코드 뒤에 공백 하나를 두고 쓴다. 줄 끝 주석끼리 열을 맞추지 않는다.

틀린 예:

```c
struct date;

// returns the number of days from start to end
long count_days_between(const struct date *start, const struct date *end);
```

```c
struct date;

/* Returns the number of days from start to end. Both dates use the
   proleptic Gregorian calendar, and end may come before start. */
long count_days_between(const struct date *start, const struct date *end);
```

```c
static int gs_retry_limit = 3;   /* attempts before giving up */
static int gs_delay_ms = 250;    /* wait between attempts */
```

올바른 예:

```c
struct date;

/* returns the number of days from start to end */
long count_days_between(const struct date *start, const struct date *end);
```

```c
struct date;

/*
 * Returns the number of days from start to end. Both dates use the
 * proleptic Gregorian calendar, and end may come before start.
 */
long count_days_between(const struct date *start, const struct date *end);
```

```c
static int gs_retry_limit = 3; /* attempts before giving up */
static int gs_delay_ms = 250; /* wait between attempts */
```

### 함수 주석과 문서 주석

함수 주석은 이름과 시그니처만으로 쓰임이 드러나지 않는 함수와, 외부에 공개하는 API, 라이브러리, 인터페이스 역할을 하는 함수에 단다. 필요하면 Doxygen이나 kernel-doc 같은 문서 주석 형식을 쓸 수 있다.

틀린 예:

```c
struct rect {
    int width;
    int height;
};

/**
 * @brief Gets the width.
 * @return The width.
 */
static int get_width(const struct rect *rect)
{
    return rect->width;
}
```

올바른 예:

```c
struct rect {
    int width;
    int height;
};

static int get_width(const struct rect *rect)
{
    return rect->width;
}
```

```c
/* returns -1 when year is outside 1..9999 */
static int count_leap_days(int year)
{
    if (year < 1 || year > 9999) {
        return -1;
    }

    return year / 4 - year / 100 + year / 400;
}
```

```c
/*
 * Parses a duration such as "90s", "15m" or "2h30m" and stores it in seconds.
 *
 * text must not be NULL. The units are h, m and s, each used at most once and
 * in that order. Whitespace is not accepted.
 *
 * Returns 0 on success. Returns -1 with errno set to EINVAL when text is not a
 * valid duration, or to ERANGE when the result does not fit in a long.
 */
int parse_duration(long *mut_seconds, const char *text);
```

```c
/**
 * @brief Parses a duration such as "90s", "15m" or "2h30m".
 * @param mut_seconds Receives the duration in seconds.
 * @param text Duration text; must not be NULL.
 * @return 0 on success, or -1 with errno set to EINVAL or ERANGE.
 */
int parse_duration(long *mut_seconds, const char *text);
```

## 오류 처리와 자원

### 반환값 검사

실패하면 결과가 달라지는 호출은 반환값을 검사한다. 반환값을 버리는 호출에는 `(void)`를 붙이지 않는다. GCC는 `warn_unused_result` 속성이 붙은 함수의 반환값을 `(void)`로 캐스트해도 경고를 낸다. 따라서, `(void)`는 이식성 있는 경고 억제 수단도 아니다.

틀린 예:

```c
static int save_notes(const char *path, const char *text)
{
    FILE *file = fopen(path, "w");
    if (file == NULL) {
        return -1;
    }

    fputs(text, file);
    fclose(file);
    file = NULL;
    return 0;
}

static long measure_file(const char *path)
{
    FILE *file = fopen(path, "rb");
    if (file == NULL) {
        return -1;
    }

    long size = -1;

    if (fseek(file, 0, SEEK_END) == 0) {
        size = ftell(file);
    }

    (void)fclose(file);
    file = NULL;
    return size;
}
```

올바른 예:

```c
static int save_notes(const char *path, const char *text)
{
    FILE *file = fopen(path, "w");
    if (file == NULL) {
        return -1;
    }

    int write_result = fputs(text, file);
    int close_result = fclose(file);

    file = NULL;
    if (write_result == EOF || close_result != 0) {
        return -1;
    }

    return 0;
}

static long measure_file(const char *path)
{
    FILE *file = fopen(path, "rb");
    if (file == NULL) {
        return -1;
    }

    long size = -1;

    if (fseek(file, 0, SEEK_END) == 0) {
        size = ftell(file);
    }

    fclose(file);
    file = NULL;
    return size;
}
```

### unused 진단

unused 계열 진단을 소스 코드 안에서 숨기지 않는다. `(void)x;`, unused 속성, 진단 pragma를 쓰지 않고, 진단을 피하려고 이름을 지우지도 않는다. 이 진단을 무시할지, 컴파일러 옵션(`-Wno-unused-parameter`, `/wd4100` 등)으로 끌지는 프로젝트의 재량으로 정한다.

틀린 예:

```c
static int print_entry(const char *path, void *mut_context)
{
    (void)mut_context;
    puts(path);
    return 0;
}

static int count_entry(const char *path, __attribute__((unused)) void *mut_context)
{
    return (path[0] == '.') ? 0 : 1;
}
```

```c
#pragma GCC diagnostic ignored "-Wunused-parameter"

static int print_entry(const char *path, void *mut_context)
{
    puts(path);
    return 0;
}
```

틀린 예(C23 이후):

```c
static int print_entry(const char *path, void *)
{
    puts(path);
    return 0;
}
```

올바른 예:

```c
static int print_entry(const char *path, void *mut_context)
{
    puts(path);
    return 0;
}

static int count_entry(const char *path, void *mut_context)
{
    return (path[0] == '.') ? 0 : 1;
}
```

### errno

`errno`는 실패한 호출 바로 뒤에 읽는다. 읽기 전에 다른 라이브러리 호출이 끼면, 그 호출보다 먼저 `errno`를 지역 변수에 저장한다.

틀린 예:

```c
static int replace_file(const char *temp_path, const char *final_path)
{
    if (rename(temp_path, final_path) != 0) {
        remove(temp_path);
        perror("rename"); // [!hl]
        return -1;
    }

    return 0;
}
```

올바른 예:

```c
static int replace_file(const char *temp_path, const char *final_path)
{
    if (rename(temp_path, final_path) != 0) {
        perror("rename"); // [!hl]
        remove(temp_path);
        return -1;
    }

    return 0;
}
```

```c
static int replace_file(const char *temp_path, const char *final_path)
{
    if (rename(temp_path, final_path) != 0) {
        int saved_errno = errno;

        remove(temp_path);
        /* remove() may overwrite errno, so restore the error of rename() for the caller */
        errno = saved_errno;
        return -1;
    }

    return 0;
}
```

### 자원 정리

- 정리할 자원이 없으면 실패한 곳에서 바로 반환한다.
- 정리할 자원이 있으면 함수 끝에 정리 레이블을 두고 `goto`로 점프한다. 레이블은 자원을 얻은 순서의 역순으로 둔다.
- 레이블 이름은 뜻이 드러나게 짓는다(`handle_buffer_alloc`, `out_close_file`). `err1`, `err2`처럼 번호를 붙이지 않는다.

틀린 예:

```c
#define LINE_LEN 256

struct palette;

int parse_palette(struct palette *mut_palette, const char *line);

static int load_palette(struct palette *mut_palette, const char *path)
{
    FILE *file = fopen(path, "r");
    char *line = malloc(LINE_LEN);

    if (file == NULL || line == NULL) {
        goto err1;
    }

    if (fgets(line, LINE_LEN, file) == NULL) {
        goto err1;
    }

    return parse_palette(mut_palette, line);

err1:
    free(line);
    line = NULL;
    fclose(file);
    file = NULL;
    return -1;
}
```

올바른 예:

```c
#define LINE_LEN 256

struct palette;

int parse_palette(struct palette *mut_palette, const char *line);

static int load_palette(struct palette *mut_palette, const char *path)
{
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        return -1;
    }

    int result = -1;

    char *line = malloc(LINE_LEN);
    if (line == NULL) {
        goto out_close_file;
    }

    if (fgets(line, LINE_LEN, file) != NULL && parse_palette(mut_palette, line) == 0) {
        result = 0;
    }

    free(line);
    line = NULL;
out_close_file:
    fclose(file);
    file = NULL;
    return result;
}
```

### 동적 메모리

- 동적 할당받은 포인터의 base 값을 수정하는 시도는 구조적으로 어쩔 수 없는 경우가 아닌 한 최대한 지양한다. 순회에는 별도 포인터를 쓴다.
- `free`를 부른 바로 다음 문장에서 그 포인터에 `NULL`을 할당한다. 특별한 이유가 없는 한, 그 포인터를 다시 쓰지 않더라도, 함수가 곧바로 반환하더라도 할당한다. `fclose`처럼 대상을 해제하는 다른 함수를 부른 뒤에도 같다. 다만 매개변수로 받은 포인터는 예외로 한다. 매개변수에 `NULL`을 할당해도 호출하는 쪽의 포인터는 바뀌지 않기 때문이다.

틀린 예:

```c
struct history {
    int *entries;
    size_t count;
};

static int fill_history(struct history *mut_history, size_t count, int initial)
{
    if (count == 0 || count > SIZE_MAX / sizeof(*mut_history->entries)) {
        return -1;
    }

    mut_history->entries = malloc(count * sizeof(*mut_history->entries));
    if (mut_history->entries == NULL) {
        return -1;
    }

    for (size_t i = 0; i < count; ++i) {
        *mut_history->entries = initial;
        ++mut_history->entries;
    }

    mut_history->count = count;
    return 0;
}

static void clear_history(struct history *mut_history)
{
    free(mut_history->entries);
    mut_history->count = 0;
}
```

```c
static long sum_squares(const int *values, size_t count)
{
    if (count == 0) {
        return 0;
    }
    if (count > SIZE_MAX / sizeof(long)) {
        return -1;
    }

    long *squares = malloc(count * sizeof(*squares));
    if (squares == NULL) {
        return -1;
    }

    long sum = 0;

    for (size_t i = 0; i < count; ++i) {
        squares[i] = (long)values[i] * values[i];
        sum += squares[i];
    }

    free(squares);
    return sum;
}
```

올바른 예:

```c
struct history {
    int *entries;
    size_t count;
};

static int fill_history(struct history *mut_history, size_t count, int initial)
{
    if (count == 0 || count > SIZE_MAX / sizeof(*mut_history->entries)) {
        return -1;
    }

    int *entries = malloc(count * sizeof(*entries));
    if (entries == NULL) {
        return -1;
    }

    const int *end = entries + count;

    for (int *entry = entries; entry < end; ++entry) {
        *entry = initial;
    }

    mut_history->entries = entries;
    mut_history->count = count;
    return 0;
}

static void clear_history(struct history *mut_history)
{
    free(mut_history->entries);
    mut_history->entries = NULL;
    mut_history->count = 0;
}
```

```c
static long sum_squares(const int *values, size_t count)
{
    if (count == 0) {
        return 0;
    }
    if (count > SIZE_MAX / sizeof(long)) {
        return -1;
    }

    long *squares = malloc(count * sizeof(*squares));
    if (squares == NULL) {
        return -1;
    }

    long sum = 0;

    for (size_t i = 0; i < count; ++i) {
        squares[i] = (long)values[i] * values[i];
        sum += squares[i];
    }

    free(squares);
    squares = NULL;
    return sum;
}
```

### assert

`assert`는 내부 불변식 검사에만 쓴다. 내부 불변식은 프로그램이 올바르면 깨지지 않는 조건이다. 외부 입력, 파일, 시스템 호출의 실패는 `assert`로 검사하지 않는다.

`assert`는 반드시 debug 빌드에서만 동작하게 하고, 배포하는 release 빌드에는 들어가지 않게 한다. release 빌드에서 `assert`를 빼는 일은 소스 파일이 아니라 빌드 구조에서 처리한다. 즉, 빌드 시스템의 모든 release 구성에서 `NDEBUG`를 정의한다. release 빌드에서 `assert`가 빠지면 조건식의 side effect도 함께 사라진다. 따라서, `assert`의 조건식에는 side effect를 두지 않는다.

틀린 예:

```c
static FILE *open_settings(const char *path)
{
    FILE *file = fopen(path, "r");

    assert(file != NULL);
    return file;
}
```

올바른 예:

```c
static FILE *open_settings(const char *path)
{
    FILE *file = fopen(path, "r");
    if (file == NULL) {
        fprintf(stderr, "cannot open %s\n", path);
        return NULL;
    }

    return file;
}
```

```c
#define STACK_CAPACITY 32

struct stack {
    int items[STACK_CAPACITY];
    size_t top;
};

static void push_item(struct stack *mut_stack, int item)
{
    assert(mut_stack->top < STACK_CAPACITY);
    mut_stack->items[mut_stack->top] = item;
    ++mut_stack->top;
}
```

### static assertion

`_Static_assert`는 C11 표준부터 키워드이다. C11 이후 표준을 고른 프로젝트는 상수 조건을 `_Static_assert`로 컴파일할 때 검사한다. C11 이전 표준을 고른 프로젝트는 런타임 `assert`나 빌드 시스템의 검사로 대신한다.

올바른 예(C11 이후):

```c
struct record_header {
    uint16_t kind;
    uint16_t flags;
    uint32_t length;
};

_Static_assert(sizeof(struct record_header) == 8, "record_header must stay 8 bytes");
```

올바른 예(C11 이전):

```c
struct record_header {
    uint16_t kind;
    uint16_t flags;
    uint32_t length;
};

static void check_record_layout(void)
{
    assert(sizeof(struct record_header) == 8);
}
```

### 종료

- 복구할 수 없는 내부 오류가 나면 `return`이나 `exit()`로 끝낸다.
- 사용자 입력이나 환경 때문에 계속할 수 없으면 메시지를 출력하고 `EXIT_FAILURE`로 끝낸다. 이때 `main`에서는 `return EXIT_FAILURE;`로 끝내고, `main`이 아닌 함수에서는 `exit(EXIT_FAILURE);`로 끝낸다. 처음 호출된 `main`에서 반환하면 그 반환값으로 `exit`를 부른 것과 같기 때문이다(C99 §5.1.2.2.3 ¶1). 반면 다른 함수에서 반환하면 호출한 함수로 돌아갈 뿐이다.

틀린 예:

```c
enum job_state {
    JOB_STATE_IDLE = 0,
    JOB_STATE_RUNNING,
    JOB_STATE_DONE,
};

static enum job_state gs_job_state = JOB_STATE_IDLE;

static void finish_job(void)
{
    if (gs_job_state != JOB_STATE_RUNNING) {
        abort();
    }

    gs_job_state = JOB_STATE_DONE;
}

static long parse_dimension(const char *text)
{
    char *end = NULL;

    errno = 0;
    long value = strtol(text, &end, 10);

    if (errno == ERANGE || end == text || *end != '\0' || value <= 0) {
        abort();
    }

    return value;
}

int main(int argc, char *argv[])
{
    if (argc != 3) {
        abort();
    }

    long width = parse_dimension(argv[1]);
    long height = parse_dimension(argv[2]);

    if (width > LONG_MAX / height) {
        abort();
    }

    gs_job_state = JOB_STATE_RUNNING;
    printf("area: %ld\n", width * height);
    finish_job();
    return 0;
}
```

올바른 예:

```c
enum job_state {
    JOB_STATE_IDLE = 0,
    JOB_STATE_RUNNING,
    JOB_STATE_DONE,
};

static enum job_state gs_job_state = JOB_STATE_IDLE;

static void finish_job(void)
{
    if (gs_job_state != JOB_STATE_RUNNING) {
        exit(EXIT_FAILURE);
    }

    gs_job_state = JOB_STATE_DONE;
}

static long parse_dimension(const char *text)
{
    char *end = NULL;

    errno = 0;
    long value = strtol(text, &end, 10);

    if (errno == ERANGE || end == text || *end != '\0' || value <= 0) {
        fprintf(stderr, "invalid dimension: %s\n", text);
        exit(EXIT_FAILURE);
    }

    return value;
}

int main(int argc, char *argv[])
{
    if (argc != 3) {
        const char *program = (argc > 0) ? argv[0] : "area";

        fprintf(stderr, "usage: %s <width> <height>\n", program);
        return EXIT_FAILURE;
    }

    long width = parse_dimension(argv[1]);
    long height = parse_dimension(argv[2]);

    if (width > LONG_MAX / height) {
        fprintf(stderr, "area too large: %ld x %ld\n", width, height);
        return EXIT_FAILURE;
    }

    gs_job_state = JOB_STATE_RUNNING;
    printf("area: %ld\n", width * height);
    finish_job();
    return 0;
}
```

## 도구

### clang-format

서식 검사와 정리에는 프로젝트 루트의 `.clang-format`을 쓸 수 있다. 커밋하기 전에 바꾼 파일을 검사한다. Visual Studio도 파일에서 가장 가까운 `.clang-format`으로 서식을 맞춘다.

clang-format의 기본 동작이 이 문서의 규칙과 어긋나면, clang-format 전체를 끄지 않고 그 동작을 정하는 설정만 골라 바꾼다. 루트 `.clang-format`의 설정은 다음과 같다.

- `BasedOnStyle: LLVM`과 `IndentWidth: 4`: LLVM 스타일을 바탕으로 하고, 들여쓰기를 공백 4칸으로 바꾼다([5.1절](#들여쓰기)).
- `ColumnLimit: 0`: 줄은 사람이 나눈다([5.2절](#줄-길이), [5.3절](#긴-식-나누기)). clang-format은 사람이 나눈 줄을 대부분 그대로 두고, 들여쓰기와 정렬만 맞춘다.
- `BreakBeforeBraces: Custom`과 `BraceWrapping`의 `AfterFunction: true`: 함수 본문의 여는 중괄호만 새 줄에 둔다([5.6절](#함수-중괄호), [5.7절](#제어문과-반복문-중괄호)).
- `InsertBraces: true`: 제어문과 반복문의 본문을 중괄호로 감싼다([5.8절](#중괄호-생략-금지)).
- `AllowShortFunctionsOnASingleLine`의 `Empty: false`, `Inline: false`, `Other: false`: 짧은 함수의 본문을 한 줄로 합치지 않는다([5.6절](#함수-중괄호)).
- `AllowShortEnumsOnASingleLine: false`: enum의 원소를 한 줄로 합치지 않는다([5.11절](#중괄호를-이용한-초기화)).
- `Cpp11BracedListStyle: Block`: 중괄호 초기화 목록의 안쪽에 공백을 둔다([5.11절](#중괄호를-이용한-초기화)).
- `AlignTrailingComments`의 `Kind: Never`: 줄 끝 주석을 열에 맞추지 않는다([5.18절](#정렬)).
- `AlignEscapedNewlines: Left`: 여러 줄 매크로의 줄 끝 `\`를 같은 열에 맞춘다([5.18절](#정렬)).
- `IndentPPDirectives: None`: 중첩된 조건부 컴파일 블록 안에서도 directive를 들여쓰지 않고 줄의 맨 앞에 붙인다([5.19절](#preprocessing-directive)).
- `IncludeBlocks: Regroup`과 `IncludeCategories`: include를 유형별로 묶어 정렬한다([4.8절](#include-순서)). clang-format은 main header를 category 0으로 분류해 맨 앞에 둔다. 종속성이 있는 헤더는 다른 `<...>` 헤더와 같은 `Priority`에 더 큰 `SortPriority`를 주어 같은 묶음의 맨 뒤로 보낸다. 이때 `"..."` 묶음의 `Priority`는 그 `SortPriority`보다 크게 둔다.
- `InsertNewlineAtEOF: true`: 파일 끝에 개행이 없으면 더한다([4.4절](#마지막-개행)).

이 설정으로 clang-format이 맞추는 규칙은 마지막 개행([4.4절](#마지막-개행)), C 소스의 공백 문자([4.5절](#공백-문자)), include 순서([4.8절](#include-순서)), 들여쓰기([5.1절](#들여쓰기)), 소괄호 안에서 나눈 줄의 정렬([5.3절](#긴-식-나누기)), 중괄호의 위치([5.6절](#함수-중괄호)~[5.10절](#범위를-좁히는-블록)), 중괄호 초기화 목록 안쪽의 공백([5.11절](#중괄호를-이용한-초기화)), `switch`와 goto 레이블([5.12절](#switch와-case), [5.13절](#goto-레이블)), 공백과 포인터 표기([5.14절](#공백), [5.15절](#포인터-표기)), 한 줄에 한 문장([5.16절](#한-줄에는-가능한-한-문장만)), 정렬([5.18절](#정렬)), preprocessing directive([5.19절](#preprocessing-directive)), 줄 끝 주석의 열 맞춤([12.2절](#주석-형식))이다. 그 밖의 규칙은 사람이 맞춘다. 특히 다음 항목은 clang-format이 고치지 않거나 규칙과 다르게 바꾸므로 직접 확인한다.

- 100열을 넘는 줄은 나누지 않는다([5.2절](#줄-길이)). 이항 연산자 앞에서 나눈 줄은 한 줄로 합치므로, 합친 줄이 100열을 넘을 수 있다.
- 할당 연산자 뒤나 반환형 뒤에서 나눈 줄([5.3절](#긴-식-나누기), [5.5절](#함수-선언부))과 나눠 쓴 메시지 문자열([5.4절](#사용자에게-보이는-문자열))은 그대로 둔다.
- 늘어날 목록의 마지막 쉼표를 더하지 않고, 한 줄에 쓴 designated initializer를 멤버마다 나누지 않는다([5.11절](#중괄호를-이용한-초기화)).
- 중괄호 없는 본문에 중괄호를 넣지만, 들여쓰기만 맞춰 둔 다음 문장은 본문에 넣지 않는다([5.8절](#중괄호-생략-금지)).
- 소괄호 없이 중첩한 삼항 연산자는 줄을 나눠 정렬한다([8.8절](#삼항-연산자)). 안쪽 삼항 연산자를 소괄호로 감싸면 한 줄로 둔다.
- 빈 줄([5.17절](#빈-줄))과 인라인 어셈블리의 문자열([5.20절](#gccclang-인라인-어셈블리))은 고치지 않는다.

종속성 코드의 디렉터리에는 `DisableFormat: true`만 담은 `.clang-format`과 그 코드에 맞춘 `.editorconfig`를 둔다([1.2절](#프로젝트와-종속성의-코딩-스타일-차이)). 이 `.editorconfig`에는 `root = true`를 넣는다. 그러지 않으면 프로젝트 루트의 `.editorconfig`에서 줄 끝 공백 제거 같은 설정을 물려받는다.

플랫폼마다 독립된 트리로 나눈 프로젝트([1.1절](#적용-대상))는 트리마다 그 플랫폼의 스타일을 담은 `.clang-format`을 나란히 둔다. clang-format은 파일에서 가장 가까운 `.clang-format`을 쓰므로, 한 트리의 설정이 다른 트리에 섞이지 않는다. `.editorconfig`는 상위 디렉터리의 `.editorconfig`와 설정이 합쳐지므로, 트리마다 둔 `.editorconfig`에는 `root = true`를 넣는다.

종속성 코드가 프로젝트 코드와 같은 디렉터리에 있으면, 프로젝트 루트의 `.clang-format-ignore`에 종속성 파일의 경로를 적고, 루트 `.editorconfig`에는 종속성 파일만 고르는 섹션을 따로 둔다. clang-format은 `.clang-format-ignore`에 적힌 파일을 건너뛴다.

올바른 예:

```sh
clang-format --dry-run --Werror path/to/file.c
clang-format -i path/to/file.c
```

올바른 예(프로젝트 루트의 `.clang-format`):

```yaml
BasedOnStyle: LLVM
IndentWidth: 4
ColumnLimit: 0
BreakBeforeBraces: Custom
BraceWrapping:
    AfterFunction: true
InsertBraces: true
AllowShortFunctionsOnASingleLine:
    Empty: false
    Inline: false
    Other: false
AllowShortEnumsOnASingleLine: false
Cpp11BracedListStyle: Block
AlignTrailingComments:
    Kind: Never
AlignEscapedNewlines: Left
IndentPPDirectives: None
IncludeBlocks: Regroup
IncludeCategories:
    - Regex: '^<readline/'
      Priority: 1
      SortPriority: 2
    - Regex: '^<'
      Priority: 1
      SortPriority: 1
    - Regex: '^"'
      Priority: 3
InsertNewlineAtEOF: true
```

올바른 예(종속성 코드 디렉터리의 `.clang-format`):

```yaml
DisableFormat: true
```

올바른 예(공백 2칸으로 들여쓰는 종속성 코드 디렉터리의 `.editorconfig`):

```ini
root = true

[*.{c,h}]
indent_style = space
indent_size = 2
```

올바른 예(플랫폼마다 독립된 트리로 나눈 프로젝트):

```text
netprobe/
    linux/
        .clang-format
        .editorconfig
        netprobe.c
    windows/
        .clang-format
        .editorconfig
        netprobe.c
```

올바른 예(프로젝트 루트의 `.clang-format-ignore`):

```text
src/ini.c
src/ini.h
```

올바른 예(프로젝트 루트의 `.editorconfig` 끝에 더하는 섹션):

```ini
[src/ini.{c,h}]
indent_style = space
indent_size = 2
trim_trailing_whitespace = false
```

### clang-tidy

이름 규칙([6장](#이름))은 프로젝트 루트의 `.clang-tidy`에서 `readability-identifier-naming`으로 검사한다. 예약된 식별자([6.10절](#예약된-식별자))는 `bugprone-reserved-identifier`로, 선언과 정의의 매개변수 이름([10.1절](#매개변수-목록))은 `readability-inconsistent-declaration-parameter-name`으로 검사한다. 포인터 매개변수의 `mut_` 접두어([10.3절](#포인터-매개변수))는 clang-tidy로 검사할 수 없다. `readability-identifier-naming`은 포인터 매개변수를 매개변수 자체의 `const` 여부로만 나누고, 대상의 `const` 여부로는 나누지 않기 때문이다.

매개변수 자체에 붙은 `const`([10.3절](#포인터-매개변수))는 `readability-avoid-const-params-in-decls`로 검사한다. 이 검사는 함수 선언만 보고, 함수 정의와 함수 포인터 타입의 매개변수는 보지 않는다.

Visual Studio는 코드 분석 설정에서 Enable Clang-Tidy를 켜면 같은 `.clang-tidy`로 clang-tidy를 실행한다. CMake 프로젝트는 `CMakePresets.json`의 Visual Studio vendor 설정에 `enableClangTidyCodeAnalysis`를 두어 켠다.

올바른 예:

```sh
clang-tidy path/to/file.c -- -std=gnu11
```

### 컴파일러 경고

- GCC와 GNU 드라이버 Clang(Windows용 `clang` 포함)은 `-Wall -Wextra -pedantic -Wshadow -Wformat -Werror=format-security`로 빌드한다. GNU11(`-std=gnu11`)에서 `asm`을 쓸 때 Clang은 `-pedantic` 아래 `-Wlanguage-extension-token` 경고를 내므로 Clang 빌드와 `clangd`·`clang-tidy`에는 `-Wno-language-extension-token`을 덧붙인다.
- MSVC는 `/W4`로 빌드한다. `/W4`에는 shadowing 경고 C4456, C4457, C4459가 들어 있다. MSVC에는 C용 `-pedantic`에 해당하는 옵션이 없다. 언어 확장을 끄는 `/Za`를 주면 컴파일러가 C89만 받아들인다.
- clang-cl은 `/W4 -Wshadow /clang:-pedantic`으로 빌드한다. clang-cl에서 `/W4`를 주면 `-Wall -Wextra`만 켜진다.
- unused 계열을 뺀 경고는 모두 고친다([13.2절](#unused-진단)).
- `-Werror`와 `/WX`는 쓰지 않는다. 다만 포맷 문자열 보안 경고(`-Werror=format-security`)처럼 보안 결함으로 이어지는 개별 경고만 예외로 둘 수 있다.

Visual Studio 프로젝트는 이 옵션을 트리 루트 디렉터리의 `Directory.Build.targets`에서 받는다. MSBuild는 각 프로젝트에서 가장 가까운 `Directory.Build.targets`를 프로젝트 파일 다음에 가져온다. 따라서, 나중에 더한 프로젝트를 포함해 트리의 모든 프로젝트가 이 설정을 받는다. 프로젝트 템플릿이 적는 `WarningLevel`도 이 설정으로 바뀐다. `Directory.Build.props`는 이 용도에 맞지 않는다. MSBuild가 `Directory.Build.props`를 프로젝트 파일보다 먼저 가져오므로, 거기에 둔 값은 템플릿 값에 밀린다.

unused 계열 진단만 모아 볼 때는 그 진단만 걸러 따로 검사한다. GCC는 `-fsyntax-only`로 검사하면 `-Wunused-function`을 내지 않으므로, 오브젝트 코드까지 컴파일한 뒤 `-Wunused`가 붙은 진단만 고른다. Clang은 `-Wno-everything`으로 모든 경고를 끈 뒤 unused 계열만 다시 켠다. 어느 파일에서도 쓰지 않는 external linkage 함수는 컴파일러가 찾지 못하므로, Cppcheck의 `unusedFunction` 검사로 프로젝트 전체를 검사한다.

올바른 예:

```sh
gcc -std=gnu11 -Wall -Wextra -pedantic -Wshadow -Wformat -Werror=format-security -Wno-unused-parameter -c foo.c
clang -std=gnu11 -Wall -Wextra -pedantic -Wshadow -Wformat -Werror=format-security -Wno-unused-parameter -Wno-language-extension-token -c foo.c
```

올바른 예(MSVC와 clang-cl):

<!-- caption kind="none" -->
```bat
cl /std:c11 /W4 /wd4100 /c foo.c
clang-cl /clang:-std=gnu11 /W4 -Wshadow /clang:-pedantic /wd4100 /c foo.c
```

올바른 예(Visual Studio 트리 루트의 `Directory.Build.targets`):

```xml
<Project>
  <ItemDefinitionGroup>
    <ClCompile>
      <WarningLevel>Level4</WarningLevel>
      <DisableSpecificWarnings>4100;%(DisableSpecificWarnings)</DisableSpecificWarnings>
      <UseStandardPreprocessor Condition="'$(PlatformToolset)' != 'ClangCL'">true</UseStandardPreprocessor>
      <LanguageStandard_C Condition="'$(PlatformToolset)' != 'ClangCL'">stdc11</LanguageStandard_C>
    </ClCompile>
  </ItemDefinitionGroup>
  <ItemDefinitionGroup Condition="'$(PlatformToolset)' == 'ClangCL'">
    <ClCompile>
      <AdditionalOptions>/clang:-std=gnu11 -Wshadow /clang:-pedantic %(AdditionalOptions)</AdditionalOptions>
    </ClCompile>
  </ItemDefinitionGroup>
</Project>
```

올바른 예(unused 계열 진단만 모아 보는 경우):

```sh
gcc -std=gnu11 -Wall -Wextra -Wunused-macros -c -o /dev/null foo.c 2>&1 | grep -- '-Wunused'
clang -std=gnu11 -fsyntax-only -Wno-everything -Wunused -Wunused-parameter -Wunused-macros foo.c
cppcheck --std=c11 --enable=unusedFunction --quiet src/
```

매개변수 자체를 바꾸는 코드([10.3절](#포인터-매개변수))는 cppcheck의 MISRA addon으로 찾을 수 있다. 이 검사는 MISRA C:2012 Rule 17.8(advisory)에 해당한다. 아래 명령은 다른 규칙의 결과를 걸러 낸다.

올바른 예(매개변수를 바꾸는 코드를 찾는 경우):

```sh
cppcheck --std=c11 --enable=style --addon=misra.py --quiet src/ 2>&1 | grep 'misra-c2012-17\.8'
```

### 에디터 설정

C 소스와 헤더의 인코딩, 줄 끝, 들여쓰기, 마지막 개행, 줄 끝 공백 규칙은 `.editorconfig`로 에디터에 알린다. Visual Studio는 이 핵심 EditorConfig 속성을 자체 텍스트 편집기 설정보다 앞서 적용한다. 프로젝트 안의 줄 끝은 `.gitattributes`로 고정한다. 에디터 설정은 이 두 파일에만 두고, 소스 파일에 에디터 modeline을 넣지 않는다.

올바른 예:

```ini
root = true

[*.{c,h}]
charset = utf-8
end_of_line = lf
indent_style = space
indent_size = 4
insert_final_newline = true
trim_trailing_whitespace = true

[*.{bat,ps1,sln,slnx,vcxproj,filters,props,targets}]
end_of_line = crlf
```

## 참고 자료

- C99 Foreword: N1256의 [PDF 9쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=9), [PDF 10쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=10) ([7.2절](#선언-위치), [7.6절](#참-거짓-값), [7.11절](#초기화), [10.6절](#inline))
- C99: [ISO/IEC 9899:1999](https://www.iso.org/standard/29237.html). WG14 공개 초안 N1256의 [§5.1.1.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=22), [§4](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=19), [Annex J.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=504) ([4.4절](#마지막-개행))
- C99 §6.3.2.3: N1256의 [PDF 59쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=59) ([8.5절](#우선순위-괄호), [8.7절](#널-포인터와-참-거짓-판정))
- C99 §6.5: N1256의 [PDF 79쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=79), [PDF 80쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=80) ([8.2절](#증감-연산자), [8.5절](#우선순위-괄호))
- C99 §6.5.7: N1256의 [PDF 97쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=97) ([8.5절](#우선순위-괄호))
- C99 §6.7.2.2: N1256의 [PDF 117쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=117) ([7.7절](#상수))
- C99 §6.7.4: N1256의 [PDF 124쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=124) ([10.6절](#inline))
- C99 §6.7.8: N1256의 [PDF 137쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=137), [PDF 139쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=139) ([7.11절](#초기화))
- C99 §7.1.3: N1256의 [PDF 178쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=178) ([6.10절](#예약된-식별자))
- C99 §5.1.2.2.3: N1256의 [PDF 25쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1256.pdf#page=25) ([13.8절](#종료))
- C11 Foreword: N1570의 [PDF 14쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=14) ([3.6절](#쓰지-않는-표준-함수), [13.7절](#static-assertion))
- C11: WG14 공개 초안 N1570의 [§5.1.1.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=29) ([4.4절](#마지막-개행))
- C11 §7.24.5.8: N1570의 [PDF 388쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n1570.pdf#page=388) ([3.5절](#표준-함수의-선택))
- C23: WG14 공개 초안 N3220의 [§5.1.1.2](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=24) ([4.4절](#마지막-개행))
- C23 §6.4.1: N3220의 [PDF 66쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=66) ([2.2절](#이름-충돌-피하기))
- C23 §6.4.4.1: N3220의 [PDF 70쪽](https://www.open-std.org/jtc1/sc22/wg14/www/docs/n3220.pdf#page=70) ([8.1절](#리터럴-표기))
- POSIX.1-2024 XBD [4.19](https://pubs.opengroup.org/onlinepubs/9799919799/basedefs/V1_chap04.html#tag_04_19) ([12.1절](#주석의-양과-내용))
- POSIX.1-2024 XSH [2.2.1.1](https://pubs.opengroup.org/onlinepubs/9799919799/functions/V2_chap02.html#tag_16_02_01_01), [2.2.1.2](https://pubs.opengroup.org/onlinepubs/9799919799/functions/V2_chap02.html#tag_16_02_01_02) ([4.7절](#feature-test-macro))
- POSIX.1-2024 XSH [2.2.2](https://pubs.opengroup.org/onlinepubs/9799919799/functions/V2_chap02.html#tag_16_02_02) ([6.10절](#예약된-식별자))
- POSIX.1-2024 XSH [strlcpy](https://pubs.opengroup.org/onlinepubs/9799919799/functions/strlcpy.html) ([2.2절](#이름-충돌-피하기))
- POSIX.1-2024 XSH [pthread_mutex_lock](https://pubs.opengroup.org/onlinepubs/9799919799/functions/pthread_mutex_lock.html) ([3.2절](#플랫폼-api))
- glibc 매뉴얼 [Feature Test Macros](https://sourceware.org/glibc/manual/latest/html_node/Feature-Test-Macros.html) ([4.7절](#feature-test-macro))
- glibc 매뉴얼 [Reserved Names](https://sourceware.org/glibc/manual/latest/html_node/Reserved-Names.html) ([6.10절](#예약된-식별자))
- glibc 소스 [sysdeps/gnu/net/if.h](https://sourceware.org/git/?p=glibc.git;a=blob;f=sysdeps/gnu/net/if.h;hb=HEAD) ([4.9절](#include-guard))
- Linux man-pages [feature_test_macros(7)](https://man7.org/linux/man-pages/man7/feature_test_macros.7.html) ([4.7절](#feature-test-macro))
- GCC 매뉴얼 [C Dialect Options](https://gcc.gnu.org/onlinedocs/gcc/C-Dialect-Options.html) ([3.1절](#언어-표준))
- GCC 매뉴얼 [Built-in Functions for Memory Model Aware Atomic Operations](https://gcc.gnu.org/onlinedocs/gcc/_005f_005fatomic-Builtins.html) ([3.2절](#플랫폼-api))
- GCC 매뉴얼 [Integers implementation](https://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html) ([3.4절](#implementation-defined-behavior))
- GCC 매뉴얼 [-Wunused-function](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wunused-function) ([4.10절](#헤더의-내용))
- GCC 매뉴얼 [Extended Asm](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#AssemblerTemplate) ([5.20절](#gccclang-인라인-어셈블리))
- GCC 매뉴얼 Extended Asm의 [Volatile](https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html#Volatile-1) ([7.9절](#register와-volatile))
- GCC 매뉴얼 [Binary Constants using the ‘0b’ Prefix](https://gcc.gnu.org/onlinedocs/gcc/Binary-constants.html) ([8.1절](#리터럴-표기))
- GCC 매뉴얼 [-Wimplicit-fallthrough](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html#index-Wimplicit-fallthrough) ([9.1절](#switch))
- GCC Bugzilla [Bug 66425](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=66425) ([13.1절](#반환값-검사))
- GCC 매뉴얼 [Warning Options](https://gcc.gnu.org/onlinedocs/gcc/Warning-Options.html) ([14.3절](#컴파일러-경고))
- Clang 매뉴얼 [Differences between various standard modes](https://clang.llvm.org/docs/UsersManual.html#differences-between-various-standard-modes) ([3.1절](#언어-표준))
- Clang 문서 [Diagnostic flags: -Wunused](https://clang.llvm.org/docs/DiagnosticsReference.html#wunused) ([14.3절](#컴파일러-경고))
- Microsoft [/std (Specify Language Standard Version)](https://learn.microsoft.com/en-us/cpp/build/reference/std-specify-language-standard-version), [/Zc:preprocessor (Enable preprocessor conformance mode)](https://learn.microsoft.com/en-us/cpp/build/reference/zc-preprocessor) ([3.1절](#언어-표준))
- CMake 소스 [Modules/Compiler/MSVC-C.cmake](https://gitlab.kitware.com/cmake/cmake/-/blob/master/Modules/Compiler/MSVC-C.cmake) ([3.1절](#언어-표준))
- Microsoft [Slim Reader/Writer (SRW) Locks](https://learn.microsoft.com/en-us/windows/win32/sync/slim-reader-writer--srw--locks), [InterlockedIncrement](https://learn.microsoft.com/en-us/windows/win32/api/winnt/nf-winnt-interlockedincrement) ([3.2절](#플랫폼-api))
- clang-format [IncludeCategories](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#includecategories), [IncludeIsMainRegex](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#includeismainregex) ([14.1절](#clang-format))
- clang-format [AlignEscapedNewlines](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#alignescapednewlines), [AlignTrailingComments](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#aligntrailingcomments), [AllowShortEnumsOnASingleLine](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#allowshortenumsonasingleline), [AllowShortFunctionsOnASingleLine](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#allowshortfunctionsonasingleline), [IndentPPDirectives](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#indentppdirectives) ([14.1절](#clang-format))
- clang-format [ColumnLimit](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#columnlimit), [DisableFormat](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#disableformat), [BasedOnStyle](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#basedonstyle), [BraceWrapping](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#bracewrapping), [InsertBraces](https://clang.llvm.org/docs/ClangFormatStyleOptions.html#insertbraces) ([14.1절](#clang-format))
- clang-format [.clang-format-ignore](https://clang.llvm.org/docs/ClangFormat.html#clang-format-ignore) ([14.1절](#clang-format))
- clang-tidy [bugprone-reserved-identifier](https://clang.llvm.org/extra/clang-tidy/checks/bugprone/reserved-identifier.html), [readability-identifier-naming](https://clang.llvm.org/extra/clang-tidy/checks/readability/identifier-naming.html), [readability-inconsistent-declaration-parameter-name](https://clang.llvm.org/extra/clang-tidy/checks/readability/inconsistent-declaration-parameter-name.html) ([14.2절](#clang-tidy))
- Cppcheck [Manual](https://cppcheck.sourceforge.io/manual.pdf) ([14.3절](#컴파일러-경고))
- EditorConfig [Specification](https://spec.editorconfig.org/) ([14.1절](#clang-format), [14.4절](#에디터-설정))
- [Linux kernel coding style](https://www.kernel.org/doc/html/latest/process/coding-style.html) ([1.1절](#적용-대상), [2장](#대원칙))
- Linux kernel coding style의 [Placing Braces and Spaces](https://www.kernel.org/doc/html/latest/process/coding-style.html#placing-braces-and-spaces) ([5.6절](#함수-중괄호), [5.7절](#제어문과-반복문-중괄호))
- Linux kernel coding style의 [Spaces](https://www.kernel.org/doc/html/latest/process/coding-style.html#spaces) ([5.15절](#포인터-표기))
- Linux kernel coding style의 [Naming](https://www.kernel.org/doc/html/latest/process/coding-style.html#naming) ([6.1절](#함수와-변수의-이름), [6.5절](#이름의-길이와-내용))
- Linux kernel coding style의 [Typedefs](https://www.kernel.org/doc/html/latest/process/coding-style.html#typedefs) ([6.9절](#typedef))
- Linux kernel coding style의 [The inline disease](https://www.kernel.org/doc/html/latest/process/coding-style.html#the-inline-disease) ([10.6절](#inline))
- Linux kernel coding style의 [Conditional Compilation](https://www.kernel.org/doc/html/latest/process/coding-style.html#conditional-compilation) ([11.1절](#조건부-컴파일))
- Linux kernel 문서 [Kernel-doc comments](https://www.kernel.org/doc/html/latest/doc-guide/kernel-doc.html) ([12.3절](#함수-주석과-문서-주석))
- MISRA C:2025: MISRA [MISRA C](https://misra.org.uk/misra-c/) ([2장](#대원칙))
- Doxygen 매뉴얼 [Documenting the code](https://www.doxygen.nl/manual/docblocks.html) ([12.3절](#함수-주석과-문서-주석))
- GNU Readline 매뉴얼 [Custom Functions](https://tiswww.cwru.edu/php/chet/readline/readline.html#Custom-Functions) ([4.8절](#include-순서))
- Git 문서 [gitattributes](https://git-scm.com/docs/gitattributes), [git-config](https://git-scm.com/docs/git-config) ([4.2절](#인코딩과-줄-끝))
- GNU Coding Standards의 [Naming Variables, Functions, and Files](https://www.gnu.org/prep/standards/html_node/Names.html) ([6.1절](#함수와-변수의-이름), [6.2절](#매크로와-enumeration-constant의-이름))
- Microsoft [Naming Files, Paths, and Namespaces](https://learn.microsoft.com/en-us/windows/win32/fileio/naming-a-file) ([4.1절](#파일-이름))
- Microsoft [Using the Windows Headers](https://learn.microsoft.com/en-us/windows/win32/winprog/using-the-windows-headers) ([4.7절](#feature-test-macro))
- Microsoft [Module 1. Your First Windows Program](https://learn.microsoft.com/en-us/windows/win32/learnwin32/your-first-windows-program) ([5.7절](#제어문과-반복문-중괄호))
- Microsoft [Define consistent coding styles with EditorConfig](https://learn.microsoft.com/en-us/visualstudio/ide/create-portable-custom-editor-options) ([4.2절](#인코딩과-줄-끝), [14.1절](#clang-format), [14.4절](#에디터-설정))
- Microsoft [Options, Text Editor, C/C++, Formatting](https://learn.microsoft.com/en-us/visualstudio/ide/reference/options-text-editor-c-cpp-formatting) ([14.1절](#clang-format))
- Microsoft [Using Clang-Tidy in Visual Studio](https://learn.microsoft.com/en-us/cpp/code-quality/clang-tidy) ([14.2절](#clang-tidy))
- Microsoft [/w, /W0, /W1, /W2, /W3, /W4, /w1, /w2, /w3, /w4, /Wall, /wd, /we, /wo, /Wv, /WX (Warning level)](https://learn.microsoft.com/en-us/cpp/build/reference/compiler-option-warning-level), [Compiler Warning (level 4) C4456](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-4-c4456), [C4457](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-4-c4457), [C4459](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-4-c4459), [/Za, /Ze (Disable Language Extensions)](https://learn.microsoft.com/en-us/cpp/build/reference/za-ze-disable-language-extensions) ([14.3절](#컴파일러-경고))
- Microsoft [Compiler Warning (level 4) C4100](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-4-c4100) ([13.2절](#unused-진단))
- Microsoft [Customize the build by folder](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-by-directory), [Customize C++ builds for MSBuild](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-cpp-builds) ([14.3절](#컴파일러-경고))
- Clang 매뉴얼 [clang-cl](https://clang.llvm.org/docs/UsersManual.html#clang-cl) ([14.3절](#컴파일러-경고))
