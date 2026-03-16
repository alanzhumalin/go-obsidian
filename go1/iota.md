# `iota` в Go (конспект)

## Что это
`iota` — специальный идентификатор для `const`-блоков, который автоматически даёт последовательные значения.

- Работает только внутри `const (...)`
- В каждом новом `const`-блоке начинается с `0`
- Увеличивается на `1` на каждой новой строке констант

---

## Базовый пример (enum-подобно)

```go
package main

import "fmt"

type Status int

const (
	StatusNew Status = iota // 0
	StatusPaid              // 1
	StatusShipped           // 2
	StatusCanceled          // 3
)

func main() {
	fmt.Println(StatusNew, StatusPaid, StatusShipped, StatusCanceled)
}
```

---

## Пропуск значения через `_`

```go
const (
	A = iota // 0
	_        // 1 (пропущено)
	C        // 2
)
```

---

## Повтор предыдущего выражения

```go
const (
	X = iota * 10 // 0
	Y             // 10
	Z             // 20
)
```

`Y` и `Z` повторяют выражение `iota * 10`, но с новым значением `iota`.

---

## Битовые флаги (`1 << iota`)

```go
package main

import "fmt"

const (
	PermRead = 1 << iota // 1  (001)
	PermWrite            // 2  (010)
	PermDelete           // 4  (100)
)

func main() {
	perms := PermRead | PermWrite
	fmt.Println(perms)                 // 3
	fmt.Println(perms&PermWrite != 0)  // true
	fmt.Println(perms&PermDelete != 0) // false
}
```

---

## Размеры (KB/MB/GB)

```go
const (
	_  = iota
	KB = 1 << (10 * iota) // 1024
	MB                    // 1048576
	GB                    // 1073741824
)
```

---

## Важные моменты
- `iota` сбрасывается в каждом новом `const`-блоке.
- Это инструмент для констант времени компиляции.
- Чаще всего используется для enum-подобных значений и флагов.