### **Контекст (`context`) в Go**

**Контекст** (`context`) — это пакет в Go, который предоставляет способ управления временем жизни горутин, передачи значений и сигналов отмены между горутинами. Основная цель контекста — синхронизировать выполнение связанных горутин и контролировать их поведение.

---

### **Основные задачи контекста**

1. **Отмена выполнения (`cancellation`):**
    - Позволяет отменить выполнение всей цепочки связанных горутин, когда выполнение больше не требуется.

1. **Тайм-ауты (`timeout`):**
    - Позволяет ограничить время выполнения операции.

1. **Передача данных (`values`):**
    - Контекст может содержать ключи-значения, которые можно использовать для передачи настроек или информации между горутинами.

1. **Сигнализация завершения:**
    - Контекст может использоваться для отправки сигналов завершения другим горутинам.

---

### **Создание контекста**

Контекст создаётся с помощью функций из пакета `context`:

1. **`context.Background()`**:
    - Используется как корневой (основной) контекст, который не содержит данных и никогда не отменяется.
    - Часто применяется как базовый контекст для создания других.

1. **`context.TODO()`**:
    - Используется в случаях, когда контекст ещё не определён, но будет добавлен позже.

1. **`context.WithCancel(parent)`**:
    - Создаёт дочерний контекст, который можно отменить вызовом функции `cancel()`.

1. **`context.WithTimeout(parent, timeout)`**:
    - Создаёт дочерний контекст с заданным тайм-аутом. По истечении времени контекст автоматически отменяется.

1. **`context.WithDeadline(parent, deadline)`**:    
    - Аналогичен `WithTimeout`, но позволяет задать конкретное время (абсолютный дедлайн).

1. **`context.WithValue(parent, key, value)`**:
    - Создаёт дочерний контекст с переданным ключом и значением.

---

### **Примеры использования**

#### **1. Использование `context.WithCancel`**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done(): // Контекст был отменён
			fmt.Printf("Worker %d stopped\n", id)
			return
		default:
			fmt.Printf("Worker %d working...\n", id)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	// Запускаем горутины
	for i := 1; i <= 3; i++ {
		go doWork(ctx, i)
	}

	time.Sleep(2 * time.Second)
	fmt.Println("Cancelling context...")
	cancel() // Отменяем контекст
	time.Sleep(1 * time.Second)
	fmt.Println("All workers stopped")
}
```

**Вывод:**

```
Worker 1 working...
Worker 2 working...
Worker 3 working...
...
Cancelling context...
Worker 1 stopped
Worker 2 stopped
Worker 3 stopped
All workers stopped
```

---

#### **2. Использование `context.WithTimeout`**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doTask(ctx context.Context) {
	select {
	case <-time.After(3 * time.Second): // Имитация долгой задачи
		fmt.Println("Task completed")
	case <-ctx.Done(): // Контекст отменён из-за тайм-аута
		fmt.Println("Task cancelled:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	go doTask(ctx)

	time.Sleep(3 * time.Second)
	fmt.Println("Main function finished")
}
```

**Вывод:**

```
Task cancelled: context deadline exceeded
Main function finished
```

---

#### **3. Использование `context.WithValue`**

```go
package main

import (
	"context"
	"fmt"
)

func processRequest(ctx context.Context) {
	userID := ctx.Value("userID") // Извлекаем значение из контекста
	if userID != nil {
		fmt.Println("Processing request for user:", userID)
	} else {
		fmt.Println("No userID found in context")
	}
}

func main() {
	ctx := context.WithValue(context.Background(), "userID", 42)

	processRequest(ctx) // Передаём контекст с данными
}
```

**Вывод:**

```
Processing request for user: 42
```

---

#### **4. Комбинация `WithCancel` и `WithTimeout`**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context, id int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("Worker %d stopped: %v\n", id, ctx.Err())
			return
		default:
			fmt.Printf("Worker %d is working...\n", id)
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	parentCtx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	ctx, cancelChild := context.WithCancel(parentCtx)
	defer cancelChild()

	go doWork(ctx, 1)
	go doWork(ctx, 2)

	time.Sleep(1 * time.Second)
	fmt.Println("Manually cancelling child context...")
	cancelChild() // Ручная отмена дочернего контекста

	time.Sleep(2 * time.Second)
	fmt.Println("Program finished")
}
```

**Вывод:**

```
Worker 1 is working...
Worker 2 is working...
Manually cancelling child context...
Worker 1 stopped: context canceled
Worker 2 stopped: context canceled
Program finished
```

 ---
#### 5. Использование `context.WithDeadline`

```Go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Дедлайн через 3 секунды от текущего времени
	deadline := time.Now().Add(3 * time.Second)

	// Создаём контекст с дедлайном
	ctx, cancel := context.WithDeadline(context.Background(), deadline)
	defer cancel() // Всегда вызываем cancel() после использования контекста

	// Канал, который завершится через 5 секунд
	result := make(chan string)

	go func() {
		time.Sleep(5 * time.Second) // Долго выполняющийся процесс
		result <- "Данные загружены!"
	}()

	select {
	case res := <-result:
		fmt.Println(res) // Успех
	case <-ctx.Done():
		fmt.Println("Операция прервана:", ctx.Err()) // Deadline exceeded
	}
}
```

Вывод:
```
Операция прервана: context deadline exceeded
```

---

### **Типичные ошибки и рекомендации**

1. **Контекст не используется в запросах.**
    - Всегда передавай контекст в функции, чтобы контролировать выполнение.

1. **Не вызывается `cancel()`.**
    - Если контекст создан через `WithCancel`, `WithTimeout` или `WithDeadline`, не забудь вызвать `cancel()`, чтобы освободить ресурсы.

1. **Неправильное использование `WithValue`.**
    - Используй `WithValue` только для передачи данных, которые актуальны для всей цепочки горутин. Не передавай большие структуры.


---

### **Когда использовать `context`**

1. **Серверные приложения:**
    - Для управления временем выполнения запросов.
    - Для передачи метаданных (например, ID пользователя).

1. **Параллельные задачи:**
    - Для синхронизации выполнения связанных горутин.

1. **Обработка запросов в API:**
    - Контекст используется для тайм-аутов и отмены, если клиент прервал соединение.

---

### **`ctx.Done()` и `ctx.Err()` в Go**

Эти два метода — важные инструменты, которые предоставляет пакет `context` для работы с контекстами. Они позволяют отслеживать завершение или отмену контекста и причину его завершения.

---

### **1. Что такое `ctx.Done()`?**

Метод **`ctx.Done()`** возвращает канал типа `<-chan struct{}`:

- **Когда вызывается**: Когда контекст отменён (`cancel`) или истёк тайм-аут/дедлайн.
- **Как использовать**: Чтение из этого канала сигнализирует о том, что нужно прекратить выполнение задачи, связанной с этим контекстом.

#### **Пример использования `ctx.Done()`**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context) {
	select {
	case <-time.After(2 * time.Second): // Имитация долгой работы
		fmt.Println("Work completed")
	case <-ctx.Done(): // Контекст отменён или истёк
		fmt.Println("Work cancelled:", ctx.Err())
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel() // Освобождаем ресурсы

	go doWork(ctx)

	time.Sleep(3 * time.Second)
	fmt.Println("Main finished")
}
```

**Вывод:**

```
Work cancelled: context deadline exceeded
Main finished
```

#### **Объяснение:**

1. Канал `ctx.Done()` "срабатывает", когда истекает тайм-аут контекста или он отменяется.
2. Это сигнал для горутины, что нужно завершить работу.

---

### **2. Что такое `ctx.Err()`?**

Метод **`ctx.Err()`** возвращает ошибку (`error`), которая объясняет, почему контекст был завершён:

- **`context.Canceled`**: Контекст был отменён явно с помощью `cancel()`.
- **`context.DeadlineExceeded`**: Истёк тайм-аут или дедлайн контекста.
- **`nil`**: Если контекст ещё активен, метод возвращает `nil`.

#### **Пример использования `ctx.Err()`**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	// Создаём контекст с тайм-аутом 1 секунду
	ctx, cancel := context.WithTimeout(context.Background(), 1*time.Second)
	defer cancel()

	// Ждём истечения тайм-аута
	time.Sleep(2 * time.Second)

	// Проверяем причину завершения контекста
	if ctx.Err() == context.DeadlineExceeded {
		fmt.Println("Context timed out")
	} else if ctx.Err() == context.Canceled {
		fmt.Println("Context was cancelled")
	} else {
		fmt.Println("Context is still active")
	}
}
```

**Вывод:**

```
Context timed out
```

---

### **Как работают `ctx.Done()` и `ctx.Err()` вместе**

Обычно они используются в комбинации:

- **`ctx.Done()`**: Для получения сигнала о завершении контекста.
- **`ctx.Err()`**: Для выяснения причины завершения.

#### **Пример**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			fmt.Println("Worker stopped due to:", ctx.Err())
			return
		default:
			fmt.Println("Worker is processing...")
			time.Sleep(500 * time.Millisecond)
		}
	}
}

func main() {
	ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	go worker(ctx)

	time.Sleep(3 * time.Second)
	fmt.Println("Main finished")
}
```

**Вывод:**

```
Worker is processing...
Worker is processing...
Worker is processing...
Worker stopped due to: context deadline exceeded
Main finished
```

---

### **Ключевые различия**

|**Метод**|**Тип возврата**|**Когда используется**|
|---|---|---|
|`ctx.Done()`|`<-chan struct{}`|Для получения сигнала о завершении/отмене контекста.|
|`ctx.Err()`|`error`|Для проверки причины завершения/отмены контекста.|

---

### **Практические примеры**

#### **Пример 1: Явная отмена контекста**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func doWork(ctx context.Context) {
	select {
	case <-ctx.Done():
		fmt.Println("Work cancelled:", ctx.Err())
	case <-time.After(2 * time.Second):
		fmt.Println("Work completed")
	}
}

func main() {
	ctx, cancel := context.WithCancel(context.Background())

	go doWork(ctx)

	time.Sleep(1 * time.Second)
	fmt.Println("Cancelling context...")
	cancel() // Отменяем контекст

	time.Sleep(2 * time.Second)
}
```

**Вывод:**

```
Cancelling context...
Work cancelled: context canceled
```

---

#### **Пример 2: Тайм-аут с дочерними контекстами**

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func subTask(ctx context.Context, name string) {
	select {
	case <-ctx.Done():
		fmt.Printf("%s stopped due to: %v\n", name, ctx.Err())
	case <-time.After(3 * time.Second):
		fmt.Printf("%s completed\n", name)
	}
}

func main() {
	parentCtx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
	defer cancel()

	childCtx, childCancel := context.WithCancel(parentCtx)
	defer childCancel()

	go subTask(parentCtx, "Parent Task")
	go subTask(childCtx, "Child Task")

	time.Sleep(4 * time.Second)
	fmt.Println("Program finished")
}
```

**Вывод:**

```
Parent Task stopped due to: context deadline exceeded
Child Task stopped due to: context deadline exceeded
Program finished
```

---

### **Заключение**

- **`ctx.Done()`**: Сигнал завершения контекста.
- **`ctx.Err()`**: Причина завершения (`context.Canceled` или `context.DeadlineExceeded`).
- Используй их вместе для управления временем выполнения горутин и контроля их завершения.
