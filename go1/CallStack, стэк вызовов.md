# Стэк вызовов (Call Stack) в Go

## Что это
Стэк вызовов — это структура LIFO, где Go хранит цепочку активных вызовов функций.

- Каждая запись называется stack frame.
- В frame обычно находятся аргументы, локальные данные и адрес возврата.
- При вызове функции frame добавляется, при `return` снимается.

## Важно в Go
- У **каждой goroutine свой стек**.
- Стек goroutine начинается маленьким и автоматически растёт/сжимается.
- `panic` печатает stack trace текущей goroutine.

## Пример цепочки вызовов
```go
package main

import "fmt"

func c() { panic("boom") }
func b() { c() }
func a() { b() }

func main() {
	fmt.Println("start")
	a()
}
```

Идея stack trace:
```text
main.c(...)
main.b(...)
main.a(...)
main.main(...)
```

## `defer` и разворачивание стека
При `panic` Go разворачивает стек вверх и выполняет `defer` в обратном порядке (LIFO).

```go
func f() {
	defer fmt.Println("defer in f")
	panic("err")
}
```

## Как получить stack trace в коде
```go
import (
	"fmt"
	"runtime/debug"
)

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("panic:", r)
			fmt.Println(string(debug.Stack()))
		}
	}()
	panic("boom")
}
```

## Частые проблемы
- Бесконечная рекурсия -> `stack overflow`.
- `panic` в одной goroutine не перехватывается `recover` из другой.
- Для обычных ошибок используй `error`, а не `panic`.