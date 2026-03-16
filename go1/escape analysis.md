
# Go: Stack, Heap, Escape Analysis (конспект)

## Stack
- Память для выполнения вызовов функций.
- Каждый вызов функции создает `stack frame`.
- Во frame: аргументы, локальные переменные, временные значения, return-slots.
- После `return` frame снимается (LIFO).
- У каждой goroutine свой стек.

## Heap
- Общая память процесса.
- Для объектов, которые могут жить дольше одного вызова функции.
- Память в heap управляется GC.

## Escape Analysis
- Анализ на этапе компиляции.
- Решает для **каждой точки аллокации**: `EscHeap` или `EscNone`.
- Решение не для всей функции, а для конкретного значения/объекта.

---

## Главная идея (формула)

\[
escapes(l) = hardReason(l)\ \lor\ \exists root:\big(derefs(root \to l) < 0 \land outlives(root,l)\big)
\]

- `l` — конкретный объект (location).
- `hardReason(l)` — принудительный heap (слишком большой/выравнивание и т.д.).
- `derefs < 0` — адрес `l` дошел до `root` (по пути больше `&`, чем `*`).
- `outlives(root,l)` — `root` живет дольше `l`.

---

## Вес ребер в графе

- `x = y` -> `0`
- `x = &y` -> `-1`
- `x = *p` -> `+1`

Если при обходе для `l` получаем `derefs < 0`, значит адрес `l` утекает в `root`.

---

## Как работает `outlives`

`outlives(root, l)` возвращает `true`, если `root` может жить дольше `l`.

Основные случаи:
1. `root` уже heap-like -> `true`.
2. `root` = result-параметр функции (`~r0`, `PPARAMOUT`) -> обычно `true`.
3. В одной функции: внешний loop-scope живет дольше внутреннего.
4. Внешняя closure-область живет дольше внутренней.

---

## Почему `return &x` => heap

```go
func f() *int {
    x := 10
    return &x
}
```

1. `return` моделируется как присваивание в скрытый result-slot `~r0`.
2. Получается поток: `~r0 <- &x` (`derefs = -1`).
3. `outlives(~r0, x) = true`.
4. Значит `x` помечается `EscHeap`.

---

## Hard reasons (forced heap)

Если `HeapAllocReason` срабатывает, объект сразу идет в heap, например:
- `too large for stack`
- `too aligned for stack`
- отдельные случаи `new`, `closure`, `makeslice` и т.д.

---

## Частые примеры

### 1) Обычно stack/register
```go
func a() int {
    x := 1
    return x
}
```

### 2) Адрес взяли, но не утек
```go
func b() int {
    x := 1
    p := &x
    *p = 2
    return x
}
```

### 3) Утечка через return
```go
func c() *int {
    x := 1
    return &x
}
```

### 4) Утечка в глобал
```go
var G *int

func d() {
    x := 1
    G = &x
}
```

### 5) Утечка через closure
```go
func e() func() int {
    x := 1
    return func() int {
        x++
        return x
    }
}
```

### 6) Forced heap из-за размера
```go
func f() *[1 << 20]int {
    a := new([1 << 20]int)
    return a
}
```

### 7) Map literal
```go
func g() map[*int]int {
    x := 1
    return map[*int]int{&x: 1}
}
```

---

## Как проверить в проекте

```bash
go build -gcflags='-m=2 -l' ./...
```

- `-m=2` показывает `escapes to heap`, `moved to heap`, flow.
- `-l` отключает inlining для более понятной диагностики.

---

## Короткий ответ на интервью

“Escape analysis в Go работает на compile-time. Компилятор строит граф потоков значений/адресов (`&=-1`, `*=+1`), затем проверяет: дошел ли адрес объекта до root (`derefs<0`) и живет ли root дольше объекта (`outlives`). Если да — объект уходит в heap; иначе остается `EscNone` (обычно stack/register). Отдельно есть forced-heap причины вроде `too large for stack`.”
```