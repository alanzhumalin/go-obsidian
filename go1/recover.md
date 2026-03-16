# `recover` в Go

## Что это
`recover` — встроенная функция, которая перехватывает `panic` и позволяет продолжить выполнение программы без аварийного падения.

## Главные правила
- Работает только внутри `defer`-функции.
- Должен вызываться в той же горутине, где произошёл `panic`.
- Возвращает значение из `panic(...)`, либо `nil`, если паники нет.
- Используется для “последней линии защиты”, а не вместо обычных `error`.

## Базовый пример

```go
package main

import "fmt"

func main() {
	defer func() {
		if r := recover(); r != nil {
			fmt.Println("panic пойман:", r)
		}
	}()

	panic("boom")
}
```

Вывод:
```text
panic пойман: boom
```

## Без `defer` не работает

```go
r := recover() // почти всегда nil (вне defer)
fmt.Println(r)
```

## Пример с безопасной обёрткой

```go
func safeRun(fn func()) (panicked bool) {
	defer func() {
		if r := recover(); r != nil {
			panicked = true
		}
	}()
	fn()
	return false
}
```

## Когда использовать
- В `main`, worker-loop, HTTP middleware, goroutine wrapper — чтобы не уронить весь процесс.
- Для логирования паники и graceful деградации.

## Когда не использовать
- Для обычных бизнес-ошибок.  
  Для них нужен `error`, а не `panic/recover`.

## Коротко
`panic` поднимает аварийную ошибку, `recover` (в `defer`) её перехватывает. Это защитный механизм, а не основной стиль обработки ошибок.