**JSON** (JavaScript Object Notation) — это текстовый формат передачи данных, который стал стандартом для обмена информацией между различными приложениями и сервисами. Хотя изначально JSON основан на JavaScript, он может использоваться в любом языке программирования. Файлы JSON используются для работы с REST API.

## Как устроен JSON-объект?

JSON-объект — это ключевая структура JSON. Он представляет собой набор пар «ключ:значение», записанных внутри фигурных скобок `{}`. Каждый ключ в объекте — это строка. Скобки отмечают начало и конец объекта.

Ключ всегда записывается в двойных кавычках (“). Этим JSON отличается от JavaScript, где кавычки не всегда обязательны. А вот цифры можно в кавычки не брать, как в примере ниже.

Внутри одного JSON-объекта может содержаться несколько пар «ключ:значение», записанных через запятую. Обычно каждую пару пишут на новой строке, но это делается только для удобства чтения.

Значение может содержать данные различного типа: строка, число, массив, другой объект, с любым количеством уровней вложенности.

#### Простой пример JSON-объекта

```json
{
  "name": "John",
  "age": 30,
  "address": {
    "city": "Moscow",
    "street": "Academician Petrovsky Street"
  }
}
```

---
### **Пример в Go:**

```go
package main

import (
	"encoding/json"
	"fmt"
)

type User struct {
	Name  string `json:"name"`
	Age   int    `json:"age"`
	Email string `json:"email"`
}

func main() {
	jsonData := `{"name":"Alice","age":25,"email":"alice@example.com"}`
	var user User

	err := json.Unmarshal([]byte(jsonData), &user)
	if err != nil {
		fmt.Println("Ошибка декодирования:", err)
		return
	}
	fmt.Println(user) // {Alice 25 alice@example.com}
}
```

✅ **Что важно:**

- `json.Unmarshal()` принимает **байтовый слайс** (`[]byte(jsonData)`).
- Передаём **указатель на структуру** (`&user`), иначе данные не запишутся.

---
## 🔹 **JSON с вложенными структурами**

Можно вложить одну структуру в другую.
### **Пример:**

```go
package main

import (
	"encoding/json"
	"fmt"
)

type Address struct {
	City  string `json:"city"`
	Zip   string `json:"zip"`
}

type User struct {
	Name    string  `json:"name"`
	Age     int     `json:"age"`
	Address Address `json:"address"`
}

func main() {
	jsonData := `{"name":"Eve","age":28,"address":{"city":"NY","zip":"10001"}}`
	var user User

	err := json.Unmarshal([]byte(jsonData), &user)
	if err != nil {
		fmt.Println("Ошибка:", err)
		return
	}

	fmt.Println(user) // {Eve 28 {NY 10001}}
}
```

