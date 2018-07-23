+++
title  = "lambda"
toc    = true
weight = 0
+++

## Lambda Calculus

- Anonymous Function: 이름을 갖을 필요가 없다.
- Currying: 두 개 이상의 입력이 있는 함수는 최종적으로 1개의 입력만 받는 Lambda Calculus로 단순화 될 수 있다.

## Lambda Function

- Anonymous Function
- First-class Function

## Labda Function & Lambda Expression

## IIFE
Immediately Invoked Function Expression
```
func(twoSeconds time.Duration) {
    // use twoSeconds
}(time.Second * 2)
```

## Closure

- Free&Bound Variable
- Opend&Closed Expression
 
익명 함수가 함수 내 정의 되지 않은 변수를 참조하는 경우.
지역 함수가 전역 변수를 참조하는것과 유사함.

## Usage
### Isolating Data
```go
package main

import "fmt"

func main() {
  gen := makeFibGen()
  for i := 0; i < 10; i++ {
    fmt.Println(gen())
  }
}

func makeFibGen() func() int {
  f1 := 0
  f2 := 1
  return func() int {
    f2, f1 = (f1 + f2), f2
    return f1
  }
}
```

### Wrapping Functions and Creating Middleware
```go
package main

import (
  "fmt"
  "net/http"
  "time"
)

func main() {
  http.HandleFunc("/hello", timed(hello))
  http.ListenAndServe(":3000", nil)
}

func timed(f func(http.ResponseWriter, *http.Request)) func(http.ResponseWriter, *http.Request) {
  return func(w http.ResponseWriter, r *http.Request) {
    start := time.Now()
    f(w, r)
    end := time.Now()
    fmt.Println("The request took", end.Sub(start))
  }
}

func hello(w http.ResponseWriter, r *http.Request) {
  fmt.Fprintln(w, "<h1>Hello!</h1>")
}
```

### Accessing data that typically isn't available
```go
package main

import (
  "fmt"
  "net/http"
)

type Database struct {
  Url string
}

func NewDatabase(url string) Database {
  return Database{url}
}

func main() {
  db := NewDatabase("localhost:5432")

  http.HandleFunc("/hello", hello(db))
  http.ListenAndServe(":3000", nil)
}

func hello(db Database) func(http.ResponseWriter, *http.Request) {
  return func(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, db.Url)
  }
}
```

### Binary Searching with the sort package
```go
package main

import (
  "fmt"
  "sort"
)

func main() {
  numbers := []int{1, 11, -5, 8, 2, 0, 12}
  sort.Ints(numbers)
  fmt.Println("Sorted:", numbers)

  index := sort.Search(len(numbers), func(i int) bool {
    return numbers[i] >= 7
  })
  fmt.Println("The first number >= 7 is at index:", index)
  fmt.Println("The first number >= 7 is:", numbers[index])
}
```

### Deferring Work
```go
go func() {
  result := doWork1(a, b)
  result = doWork2(result)
  result = doWork3(result)
  // Use the final result
}()
fmt.Println("hi!")
```
