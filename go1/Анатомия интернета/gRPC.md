***gRPC*** - Фреймворк/набор инструментов/платформа от Google

Популярен в микросервисной архитектуре. С помощью ***gRPC*** микросервисы общаются между собой

![[Pasted image 20250313190513.png]]

## Что отличает gRPC от REST API?

- Используется ***HTTP 2*** вместо ***HTTP 1/1*** (быстрее на 10-15% процентов)
- Работает в разы быстрее
- Простой и быстрый стриминг данных 
- Вместо ***JSON*** используется бинарный формат ***(protobuf)***
- Инструментарий из коробки:
	- Генерация кода для многих ЯП
	- Аутентификация
	- Потоковая передача данных (в том числе двунаправленная)
	- и т.д.
- Удобный вызов процедур

### Формат данных:
- бинарный формат ***(protobuf)***

![[Pasted image 20250313190526.png]]

![[Pasted image 20250313190535.png]]

### **Что такое gRPC?**

gRPC (Google Remote Procedure Call) – это **фреймворк удалённого вызова процедур (RPC)**, разработанный Google. Он использует **HTTP/2**, бинарный формат **Protocol Buffers (protobuf)** и поддерживает **стриминг**, **аутентификацию** и **мультиплексирование соединений**.

📌 **Главные преимущества gRPC**: ✅ **Высокая производительность** – использует HTTP/2 и бинарный формат данных.  
✅ **Поддержка стриминга** – двусторонний стриминг, клиентский и серверный.  
✅ **Язык-независимый** – работает с Go, Python, Java, C++, Rust и другими языками.  
✅ **Автоматическая генерация кода** – protobuf генерирует код для клиента и сервера.  
✅ **Поддержка аутентификации и авторизации** – легко интегрируется с TLS и OAuth2.

---

## **Как работает gRPC?**

1. **Определение сервиса** – создаётся `.proto`-файл с описанием методов и сообщений.
2. **Генерация кода** – с помощью `protoc` создаются серверная и клиентская части.
3. **Реализация сервиса** – пишется код сервера, который реализует методы.
4. **Клиент вызывает удалённые методы** – клиент подключается и отправляет запросы.

---

## **Типы вызовов в gRPC**

gRPC поддерживает **4 типа взаимодействия**:

### 1️⃣ **Обычный запрос (Unary RPC)**

📌 Один запрос → Один ответ (как обычный HTTP-запрос).

```proto
service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}
```

### 2️⃣ **Серверный стриминг (Server Streaming RPC)**

📌 Один запрос → Несколько ответов (сервер отправляет поток данных).

```proto
service ChatService {
  rpc GetMessages (MessageRequest) returns (stream Message);
}
```

### 3️⃣ **Клиентский стриминг (Client Streaming RPC)**

📌 Несколько запросов → Один ответ (клиент отправляет поток данных, сервер отвечает один раз).

```proto
service UploadService {
  rpc UploadFile (stream FileChunk) returns (UploadStatus);
}
```

### 4️⃣ **Двусторонний стриминг (Bidirectional Streaming RPC)**

📌 Клиент и сервер обмениваются сообщениями в **обе стороны** (например, чат).

```proto
service ChatService {
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}
```

---

## **Пример gRPC-сервиса на Go**

1️⃣ **Создаём `.proto`-файл** (`user.proto`):

```proto
syntax = "proto3";

package user;

service UserService {
  rpc GetUser (UserRequest) returns (UserResponse);
}

message UserRequest {
  string id = 1;
}

message UserResponse {
  string id = 1;
  string name = 2;
}
```

**Генерируем код**:

```sh
protoc --go_out=. --go-grpc_out=. user.proto
```

**Реализация сервера (`server.go`)**:

```go
package main

import (
	"context"
	"fmt"
	"net"

	pb "example.com/user"
	"google.golang.org/grpc"
)

type server struct {
	pb.UnimplementedUserServiceServer
}

func (s *server) GetUser(ctx context.Context, req *pb.UserRequest) (*pb.UserResponse, error) {
	fmt.Println("Запрос пользователя:", req.Id)
	return &pb.UserResponse{Id: req.Id, Name: "Джон Доу"}, nil
}

func main() {
	listener, err := net.Listen("tcp", ":50051")
	if err != nil {
		panic(err)
	}

	grpcServer := grpc.NewServer()
	pb.RegisterUserServiceServer(grpcServer, &server{})

	fmt.Println("gRPC сервер запущен на :50051")
	if err := grpcServer.Serve(listener); err != nil {
		panic(err)
	}
}
```

**Реализация клиента (`client.go`)**:

```go
package main

import (
	"context"
	"fmt"
	"time"

	pb "example.com/user"
	"google.golang.org/grpc"
)

func main() {
	conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
	if err != nil {
		panic(err)
	}
	defer conn.Close()

	client := pb.NewUserServiceClient(conn)

	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()

	resp, err := client.GetUser(ctx, &pb.UserRequest{Id: "123"})
	if err != nil {
		panic(err)
	}

	fmt.Println("Ответ сервера:", resp.Name)
}
```

---

## **Когда использовать gRPC?**

✅ **Микросервисы** – gRPC быстрее, чем REST API, из-за HTTP/2 и бинарного формата.  
✅ **Реальное время** – чаты, видеозвонки, стриминг.  
✅ **Взаимодействие между сервисами на разных языках**.  
✅ **Низкая задержка** – идеально для высоконагруженных систем.

---

## **Когда лучше REST API вместо gRPC?**

❌ **Открытые API** – REST API проще использовать в браузере.  
❌ **Если нужен дебаг в браузере** – REST легко тестировать через `curl` или Postman.  
❌ **Когда HTTP/1.1** – gRPC использует HTTP/2, а старые системы могут не поддерживать.

---

## **Вывод**

- **gRPC** – мощный инструмент для быстрого взаимодействия сервисов.
- Работает быстрее, чем REST API, но сложнее в настройке.
- Поддерживает **стриминг, безопасность, балансировку нагрузки**.
- Подходит для **микросервисов, распределённых систем, low-latency API**

