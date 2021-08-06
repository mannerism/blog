---
title:  "Go: 포인터 이해하기"
date:   2021-08-03 12:27:19 +0900
permalink: ":categories/programming/:title"
---

### Go: 포인터 이해하기

제 주 언어인 `swift`는 포인터를 잘 사용하지 않습니다. 언어의 디폴트가 [memory safe](https://www.raywenderlich.com/7181017-unsafe-swift-using-pointers-and-interacting-with-c)하기 때문인데, `swift`의 `unsafe` feature를 사용하면 포인터 사용이 가능하지만 평소에는 잘 사용하지 않습니다. 그래서 포인터란 개념이 저에겐 좀 낯선것이 사실입니다. 이번에 새로 `Go`언어를 공부하면서 나에게 다소 생소한 개념인 `pointer`를 좀 자세히 정리해볼 것입니다.

#### Go의 포인터

> [Go Pointers](https://tour.golang.org/moretypes/1)
> [Pointers in Golang](https://www.geeksforgeeks.org/pointers-in-golang/)

`Go` 프로그래밍 언어에서 포인터는 다른 변수의 값을 저장하는 메모리 주소를 저장하는 특별한 변수입니다. 변수는 어떠한 데이터를 특정 메모리 주소에 저장하고 사용합니다. 메모리 주소는 `hexadecimal`(0xFFAAF... 등등) 포맷으로 저장됩니다.

`*T` 타입은 `T`값에대한 포인터를 의미합니다.

  ```go
    var p *int
  ```

그리고 `&` operator를 사용하면 피연산자에 도달할 수 있는 포인터를 만들어낼 수 있습니다.

  ```go
    i := 42
    p = &i
  ```

예시에서 보이는바와 같이 `&`를 사용하여 변수 `p` 변수에 `i`에게 도달하는 포인터를 저장합니다.

`*` operator를 사용하여 포인터가 가리키는 값을 가지고올 수 있습니다.

  ```go
    fmt.Println(*p) // 1. Read
    *p = 21         // 1. Write
  ```

1. `*`를 사용하여 `p` 포인터가 가르키고있는 값을 읽을 수 있습니다.
1. `*`를 사용하여 `p` 포인터가 가르키고있는 값에 새로운 값을 쓸 수 있습니다.
1. `*`는 `dereferencing` 또는 `indirecting`이라고 칭합니다.

#### 포인터 사용 예시

  ```go
    package main

    import "fmt"

    func main() {
      i, j := 42, 2701

      p := &i         // 1. point to i
      fmt.Println(*p) // 2. read i through the pointer
      fmt.Println(p)  // 3. read i pointer address
      *p = 21         // 4. set i through the pointer
      fmt.Println(i)  // 5. see the new value of i

      p = &j         // 6. point to j
      *p = *p / 37   // 7. divide j through the pointer
      fmt.Println(j) // 8. see the new value of j
    }
  ```

설명:

  1. `&`를 사용하여 `i`변수의 메모리 주소를 `p` 포인터 변수에 저장합니다.
  1. `*`를 사용하여 `i`에 저장된 값인 `42`를 콘솔에 출력합니다.
  1. `p`가 가르키고 있는 메모리 주소인 `0xc0000be020`와 같은 형식의 `hexadecimal`값을 출력합니다.
  1. `*`를 사용하여 `p` 포인터 변수를 통해 `i`의 값을 변경합니다.
  1. 4번에서 변경된 `i`의 값인 `21`을 출력합니다.
  1. `p`가 가르키고있던 주소를 `j`변수의 메모리 주소로 변경합니다.
  1. `*`를 사용하여 `j`에 저장된 값인 `2701`을 `37`로 나누어 줍니다.
  1. 7번에서 변경된 `j`값을 출력합니다.

  결과:
  
  ```go
    42
    0xc0000be020
    21
    73
  ```

#### Go Structs에서 포인터 사용

Go 언어의 Struct에서도 포인터를 사용할 수 있습니다.

```go
  package main

  import "fmt"

  type Vertex struct {
    X int
    Y int
  }

  func main() {
    v := Vertex{1, 2} // #1
    p := &v           // #2
    (*p).X = 1e9      // #3
    fmt.Println(v)    // #4
  }
```

설명:

1. `v` 변수에 `Vertex` struct를 init해 줍니다.
2. `v`에 도달하는 포인터를 변수 `p`를 `&`를 사용하여 init해 줍니다.
3. `*`를 사용하여 포인터 변수 `p`를 통해 `v`의 `X`값을 변경해 줍니다.
4. 새로운 `v`값을 콘솔에 프린트 해 줍니다. 결과값: `{1e9, 2}`

3번같이 포인터를 사용하여 `v` struct 값을 바꾸는 경우 `*` 노테이션은 생략할 수 있습니다.

```go
    func main() {
    v := Vertex{1, 2}
    p := &v           
    p.X = 1e9         
    fmt.Println(v)    
  }
```
