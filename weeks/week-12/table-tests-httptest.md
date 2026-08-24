# Быстро повторить: table-driven tests и `httptest`

[← К полной неделе](./README.md) · [↑ К содержанию курса](../README.md)

## База

Go-тест находится в файле:

```text
*_test.go
```

и имеет форму:

```go
func TestSomething(t *testing.T) {
    // ...
}
```

Запуск:

```bash
go test ./...
go test -race ./...
go test -cover ./...
```

## Table-driven tests

Вместо копирования похожих тестов создаётся таблица кейсов:

```go
func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"negative", -2, -3, -5},
        {"zero", 5, 0, 5},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got := Add(tt.a, tt.b)

            if got != tt.expected {
                t.Errorf("got %d, want %d", got, tt.expected)
            }
        })
    }
}
```

Смысл подхода из конспекта: меньше дублирования и проще добавлять новые сценарии.

## `httptest`

Для тестирования HTTP handler без запуска обычного сервера используются:

```go
httptest.NewRequest(...)
httptest.NewRecorder()
```

Пример:

```go
req := httptest.NewRequest(http.MethodGet, "/upper?word=abc", nil)
w := httptest.NewRecorder()

upperCaseHandler(w, req)

res := w.Result()
defer res.Body.Close()
```

`ResponseRecorder` заменяет `http.ResponseWriter`, а `NewRequest` создаёт запрос для handler.

## Что проговорить перед собеседованием

- Как называется тестовый файл?
- Как выглядит тестовая функция?
- Что такое table-driven test?
- Зачем `t.Run`?
- Для чего `httptest.NewRequest`?
- Что делает `httptest.NewRecorder`?
- Как проверить HTTP handler без реального запуска сервера?
- Что даёт `go test -race`?
