---
title:  "프로그래밍: Bitwise Operator 이해하기"
date:   2021-08-03 12:27:19 +0900
permalink: ":categories/programming/:title"
---

### Golang: Bitwise Operator 이해하기

>자료1. [Algorithms 101: the basics of Bit Manipulation explained](https://www.educative.io/blog/bit-manipulation-algorithm)
>자료2. [Bitwise Operators in C](https://www.youtube.com/watch?v=8aFik6lPPaA&ab_channel=NesoAcademy)
>자료3. [Mathematics in R Markdown](https://rpruim.github.io/s341/S19/from-class/MathinRmd.html)

`Bitwise Operator`란 `Bitwise Manipulation`을 하기 위한 `operator`입니다. `Bitwise Manipulation`이란 컴퓨터가 이해하는 가장 작은 단위의 데이터인 `0`과 `1`을 변경하고 조작하는 행위로 다양한 데이터 타입에 적용할 수 있으며 우리가 잘 아는 `encod`, `decode`, `compress`등은 이러한 프로그래밍의 가장 낮은 영역인 `bit-level`에서 이루어집니다. `Bitwise Manipulation`은 특히 과거에 컴퓨터 메모리 크기가 상대적으로 매우 작았을 때, 프로그램의 효율을 높이기 위해 자주 사용 되었다고 합니다. 지금같이 하드웨어가 빵빵하게 지원되는 시대에 잘 사용되지 않는듯 하지만 결국 프로그램의 퍼포먼스 개선과 유지/관리 용이한 프로그램을 작성하기 위해서는 반드시 알아야 하는 개념중에 하나입니다.

결국에는 컴퓨터가 이해하는 `01010001010110`과 같은 데이터를 사용자가 직접 변경하고 기호에 맞게 쓸 수 있도록 도와주는 애들이 바로 `Bitwise Operator`라는 것입니다. 그럼 가장 기본적이고 대표적인 `Bitwise Operator`를 하나씩 알아보도록 합시다.

#### AND Operator

1. `&`기호를 사용합니다.
1. 두 개의 동일 길이의 `operands`에 적용할 수 있습니다.
1. 두 개의 `Bit`의 값이 `1`이면 결과값으로 `1`을 갖게 됩니다.
1. 예시:
  
    ```zsh
        0 1 1 1   <- 인풋1: 숫자 7
      & 0 1 0 0   <- 인풋2: 숫자 4
      ----------
        0 1 0 0   <- 결과값: 숫자 4
    ```

    왼쪽에서 부터 Column을 비교해보면
    1. `0` `AND` `0`은 `0`이 됩니다.
    1. `1` `AND` `1`은 `1`이 됩니다.
    1. `1` `AND` `0`은 `0`이 됩니다.
    1. 결국 `7 & 4`는 `4`가 됩니다.

#### OR Operator

1. `|`기호를 사용합니다.
1. 두 개의 동일 길이의 `operands`에 적용할 수 있습니다.
1. 둘 중 한개의 `Bit`의 값이 `1`이면 결과값으로 `1`을 갖게 됩니다.
1. 예시:
  
    ```zsh
        0 1 1 1   <- 인풋1: 숫자 7
      | 0 1 0 0   <- 인풋2: 숫자 4
      ----------
        0 1 1 1   <- 결과값: 숫자 7
    ```

    왼쪽에서 부터 Column을 비교해보면
    1. `0` `AND` `0`은 `0`이 됩니다.
    1. `1` `AND` `1`은 `1`이 됩니다.
    1. `1` `AND` `0`은 `1`이 됩니다.
    1. 결국 `7 & 4`는 `7`이 됩니다.

#### XOR Operator

1. `^`기호를 사용합니다.
1. 두 개의 동일 길이의 `operands`에 적용할 수 있습니다.
1. 둘 중 한개의 `Bit` 값이 `1`이면 결과값으로 `1`을 갖게 됩니다. 하지만, 둘 다 `1`인 경우 결과값으로 `0`을 갖게 됩니다.
1. 예시:
  
    ```zsh
        0 1 1 1   <- 인풋1: 숫자 7
      ^ 0 1 0 0   <- 인풋2: 숫자 4
      ----------
        0 0 1 1   <- 결과값: 숫자 3
    ```

    왼쪽에서 부터 Column을 비교해보면
    1. `0` `AND` `0`은 `0`이 됩니다.
    1. `1` `AND` `1`은 `0`이 됩니다.
    1. `1` `AND` `0`은 `1`이 됩니다.
    1. 결국 `7 & 4`는 `3`이 됩니다.

#### NOT Operator

1. `~`기호를 사용합니다.
1. 한 개의 `operands`에 적용할 수 있습니다.
1. 항상 특정 `Bit`의 반대값을 갖게 됩니다.
1. 예시:
  
    ```zsh
      ~ 0 1 1 1   <- 인풋: 숫자 7
      ----------
        1 0 0 0   <- 결과값: 숫자 8
    ```

    왼쪽에서 부터 Column을 비교해보면
    1. `0` `NOT` 은 `1`이 됩니다.
    1. `1` `NOT` 은 `0`이 됩니다.
    1. 결국 `7~`은 `8`이 됩니다.

#### Bitwise Operator vs. Logical Operator

그렇다면 우리가 어떤 프로그래밍 언어를 사용하더라도 정말 자주 사용하는 `Logical Operator`과 `Bitwise Operator`가 어떻게 다르게 작동하는지 살펴보도록 합시다.

```C
  #include <stdio.h>

  int main() {
    char x = 1, y = 2;                  // #1
    if (x&y)                            // #2
      printf("Result of x&y is 1");     // #3
    if (x&&y)                           // #4
      printf("Result of x&&y is 1");    // #5
    
    return 0;
  }
```

C 언어로 작성된 간단한 예시를 보고 설명을 해보겠습니다.

1. x = 1이며 타입은 `8-bits` `character`이기 때문에 1의 `Binary` 는 `0000 0001` 입니다. 그리고 y = 2이며 2의 `Binary`는 `0000 0010`입니다.
1. `x&y`는 `Bitwise AND`를 수행하며 이는 다음과 같습니다:
  
      ```zsh
          0000 0001 <- 인풋 1: 1
        & 0000 0010 <- 인풋 2: 2
        ------------
          0000 0000 -> 결과값: 0
      ```

    `#2`의 `Bitwise Operation` 결과는 0이며 0은 `False`를 의미합니다. 따라서 `#2`의 `if`문의 조건을 충족하지 못합니다.
1. `#2`의 조건이 충족되지 않기때문에 `#3`는 실행되지 않습니다.
1. `x&&y`는 `Logical AND`를 수행하며 이는 다음과 같습니다:

      ```zsh
        1&&2 = TRUE && TRUE = TRUE
      ```

    `1`과 `2`값 모두 `0`이 아니기 때문에 `TRUE`로 취급됩니다. 따라서 `Logical AND`의 결과값은 `TRUE`가 됩니다. 이는 `#4`의 `if`문 조건을 충족시킵니다.
1. `#4`의 조건이 충족되기 때문에 `printf("Result of x&&y is 1")`이 실행됩니다.

#### Left Shift Operator

1. `<<`기호를 사용합니다.
1. 두 개의 `operands`에 적용할 수 있습니다.
1. `첫번째 operand` `<<` `두번째 operand` 구조를 갖습니다.
1. `첫번째 operand`의 `bits`가 왼쪽으로 이동합니다.
1. `두번째 operand`는 얼만큼 왼쪽으로 이동하고자 하는지 정의해 줍니다.
1. `bits`가 왼쪽으로 이동할 때마다 우측의 빈 곳은 `0`으로 채워집니다.

    ```C
      var = 3;    // 3 = 0000 0011
      var << 1;   // 0000 011_
                  // 0000 0110   <- 우측 빈 부분이 `0`으로 채워집니다.
                  // 결과값 = 0000 0110 = 6

    ```

1. `Left Shifting`은 결과값은  `첫번째 operand`x $2^{두번째 operand}$ 와 같습니다.

    ```C
      var = 3;   
      var << 1;
      // 결과값 = (3 x 2^1) = 6
      
      var << 4;
      // 결과값 = (3 x 2^4) = 48
    ```

#### Right Shift Operator

1. `>>`기호를 사용합니다.
1. 두 개의 `operands`에 적용할 수 있습니다.
1. `첫번째 operand` `>>` `두번째 operand` 구조를 갖습니다.
1. `첫번째 operand`의 `bits`가 오른쪽으로 이동합니다.
1. `두번째 operand`는 얼만큼 오른쪽으로 이동하고자 하는지 정의해 줍니다.
1. `bits`가 오른쪽으로 이동할 때마다 좌측의 빈 곳은 `0`으로 채워집니다.

    ```C
      var = 3;    // 3 = 0000 0011
      var >> 1;   // _000 0001
                  // 0000 0001   <- 좌측 빈 부분이 `0`으로 채워집니다.
                  // 결과값 = 0000 0001 = 1

    ```

1. `Right Shifting`은 결과값은  $\frac { 첫번째 operand } { 2^{두번째 operand}  }$ 와 같습니다.

    ```C
      var = 3;   
      var >> 1;
      // 결과값 = (3 / 2^1) = 1.5 -> 소수는 드랍 -> 1
      
      var = 32;
      var >> 4;
      // 결과값 = (32 x 2^4) = 2
    ```
