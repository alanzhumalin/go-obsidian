
# Composition vs Embedding in Go

## Composition

**Composition** — это когда мы строим один тип из других типов через обычные поля структуры.

### Пример
```go
type Address struct {
	City string
}

type User struct {
	Name    string
	Address Address
}
```

### Использование
```go
u := User{
	Name: "Alan",
	Address: Address{
		City: "Almaty",
	},
}

fmt.Println(u.Address.City)
```

Здесь доступ идет явно через поле:

```go
u.Address.City
```

---

## Embedding

**Embedding** — это специальная форма composition в Go, когда тип встраивается в структуру **без имени поля**.

### Пример
```go
type Address struct {
	City string
}

type User struct {
	Name string
	Address
}
```

### Использование
```go
u := User{
	Name: "Alan",
	Address: Address{
		City: "Almaty",
	},
}

fmt.Println(u.City)
fmt.Println(u.Address.City)
```

Здесь поле `City` можно использовать напрямую через `User`:

```go
u.City
```

---

## Главная разница

### Composition
При обычной composition нужно явно обращаться через поле:

```go
u.Address.City
```

### Embedding
При embedding поля и методы встроенного типа **promoted** во внешний тип:

```go
u.City
u.SomeMethod()
```

---

## Что значит promoted

Это значит, что поля и методы встроенного типа становятся доступны через внешний тип, как будто они принадлежат ему.

Но важно:

- это не inheritance
- внешний тип все равно просто содержит встроенный тип внутри себя

---

## Пример с методами

```go
type Logger struct{}

func (Logger) Log(msg string) {
	fmt.Println("LOG:", msg)
}

type Service struct {
	Logger
}
```

Теперь можно писать:

```go
s := Service{}
s.Log("hello")
```

Хотя метод `Log` определен у `Logger`.

---

## Проблема embedding

Если встроить два типа с одинаковыми именами полей или методов, прямой доступ станет неоднозначным.

### Пример
```go
type A struct {
	Name string
}

type B struct {
	Name string
}

type C struct {
	A
	B
}
```

Так писать нельзя:

```go
c.Name
```

Потому что компилятор не поймет:

- `A.Name`
- или `B.Name`

Но так писать можно:

```go
c.A.Name
c.B.Name
```

---

## Когда использовать Composition

Composition лучше использовать, когда:

- нужна явная и понятная структура
- важно видеть, откуда пришло поле
- не хочется лишних конфликтов имен
- внешний тип не должен “поднимать” чужие поля и методы

---

## Когда использовать Embedding

Embedding лучше использовать, когда:

- нужно переиспользовать методы
- нужен удобный доступ к полям встроенного типа
- встроенный тип логически является частью внешнего типа
- хочется сделать что-то вроде mixin-поведения

---

## Короткий вывод

- `Composition` — обычное включение одного типа в другой через поле
- `Embedding` — специальная форма composition без имени поля
- при embedding поля и методы встроенного типа поднимаются во внешний тип
- embedding удобнее, но может приводить к конфликтам имен
- если есть сомнения, composition обычно безопаснее и понятнее
```