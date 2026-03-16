# `defer` в Go

## Что это
`defer` откладывает вызов функции до выхода из текущей функции.

```go
defer someFunc()
```

`someFunc()` выполнится перед `return` (в самом конце функции).

---

## Зачем используют
- `file.Close()`
- `rows.Close()`
- `mu.Unlock()`
- `recover()` в `defer` для обработки panic
- Логирование времени/завершения функции

---

## Базовый пример

```go
package main

import "fmt"

func main() {
	fmt.Println("start")
	defer fmt.Println("defer")
	fmt.Println("end")
}
```

Вывод:
```text
start
end
defer
```

---

## Важное правило: LIFO
Несколько `defer` выполняются в обратном порядке (Last In, First Out).

```go
defer fmt.Println("1")
defer fmt.Println("2")
defer fmt.Println("3")
// выполнится: 3, 2, 1
```

---

## Аргументы вычисляются сразу
В `defer` выражения аргументов считаются в момент объявления `defer`, а не в конце.

```go
x := 10
defer fmt.Println(x) // запомнит 10
x = 20
// напечатает 10
```

---

## Типичный кейс: файл

```go
f, err := os.Open("data.txt")
if err != nil {
	return err
}
defer f.Close()
```

Даже если дальше будет ранний `return`, `Close()` вызовется.

---

## `defer` + `panic/recover`

```go
defer func() {
	if r := recover(); r != nil {
		fmt.Println("recovered:", r)
	}
}()
panic("boom")
```

`recover()` работает только внутри deferred-функции.

---

## Нюансы производительности
- `defer` очень удобен и обычно его стоит использовать.
- В очень горячих циклах иногда делают ручной `Close/Unlock` для микропроизводительности, но сначала профилировать.

---

## Коротко
`defer` = “выполни это в конце функции”; главный инструмент для безопасной очистки ресурсов.