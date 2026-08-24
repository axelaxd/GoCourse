# Быстро повторить: `http.Handler` и middleware

[← К полной неделе](./README.md) · [↑ К содержанию курса](../README.md)

## Handler

В `net/http` обработчик соответствует поведению:

```go
ServeHTTP(w http.ResponseWriter, r *http.Request)
```

`ResponseWriter` используется для формирования ответа, а `*http.Request` содержит данные запроса.

Функцию можно превратить в обработчик через `http.HandlerFunc`.

## Разбор запроса

Из `*http.Request` доступны:

```go
r.Method
r.URL.Path
r.URL.Query()
r.Header
r.Body
```

Для JSON-тела в конспекте используется:

```go
json.NewDecoder(r.Body).Decode(&obj)
```

## HTTP-клиент

Важный момент из конспекта: HTTP-код `404` или `500` сам по себе не означает, что `err` от выполнения запроса будет ненулевым.

Поэтому отдельно проверяется:

```go
resp.StatusCode
```

И после успешного получения ответа нужно закрывать:

```go
defer resp.Body.Close()
```

## Middleware

Middleware выполняет код вокруг другого handler.

Типичная форма:

```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("before")

        next.ServeHTTP(w, r)

        fmt.Println("after")
    })
}
```

То есть:

```text
request
  ↓
middleware
  ↓
next.ServeHTTP(...)
  ↓
handler
```

Middleware подходит для логирования, авторизации и другой общей логики вокруг обработчиков.

## Что проговорить перед собеседованием

- Что такое `http.Handler`?
- Что делает `ServeHTTP`?
- Для чего нужен `http.HandlerFunc`?
- Где лежат method, query, headers и body?
- Почему нужно проверять `StatusCode` отдельно от `err`?
- Почему закрываем `resp.Body`?
- Как устроен middleware?
