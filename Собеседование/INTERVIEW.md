# Go Interview --- быстрый прогон вопросов

> Ответы на эти вопросы находятся в соответствующих 14 тематических
> README.

## Core

-   Почему `len("привет")` не равен количеству букв?
-   Чем array отличается от slice?
-   Что хранит slice и что меняется после `append`?
-   Чем `len` отличается от `cap`?
-   Что произойдёт при `append`, если capacity закончился?
-   Чем nil slice отличается от empty slice?
-   Как получить значение из map и проверить наличие ключа?
-   Что произойдёт при записи в nil map?
-   Что делают `&` и `*`?
-   Когда стоит использовать pointer receiver?

## Structs и interfaces

-   Есть ли в Go классы?
-   Как методы связываются с типом?
-   Что такое embedding?
-   Как тип реализует interface?
-   Что такое `any`?
-   Что такое type assertion?
-   Что такое type switch?
-   Что такое method set?
-   Почему interface удобно объявлять на стороне потребителя?

## Errors

-   Почему errors в Go являются значениями?
-   Как обернуть ошибку?
-   Зачем `%w`?
-   Чем `errors.Is` отличается от `errors.As`?
-   Когда вызываются defer-функции?
-   Чем panic отличается от error?
-   Что делает recover?

## Concurrency

-   Concurrency vs parallelism?
-   Что такое goroutine?
-   Buffered vs unbuffered channel?
-   Когда отправитель блокируется?
-   Кто должен закрывать channel?
-   Как узнать, что channel закрыт?
-   Как работает `select`?
-   Что происходит с nil channel?
-   Что такое race condition?
-   Как запустить race detector?
-   Когда нужен Mutex?
-   Для чего WaitGroup?
-   Что делает Once?
-   Чем `atomic` отличается от mutex-подхода?

## Context

-   Для чего нужен Context?
-   Почему Context первым параметром?
-   Почему Context не хранят в struct?
-   `WithCancel` vs `WithTimeout` vs `WithDeadline`?
-   Что возвращает `Done()`?
-   Что происходит при отмене родительского context?
-   Как через Context корректно остановить воркер?
-   Как сделать graceful shutdown HTTP-сервера?

## HTTP

-   Что такое `http.Handler`?
-   Какая сигнатура `ServeHTTP`?
-   Что такое `HandlerFunc`?
-   Как прочитать query/header/body?
-   Как вернуть HTTP error?
-   Что такое middleware?
-   Почему `http.Client.Do` может вернуть `err == nil` при HTTP 500?
-   Зачем закрывать `resp.Body`?

## Testing

-   Как называется файл с тестами?
-   Что принимает `TestXxx`?
-   Что такое table-driven tests?
-   `Errorf` vs `Fatal`?
-   Как запустить все тесты?
-   Как проверить coverage?
-   Как включить race detector?
-   Для чего mocks?
-   Для чего `httptest`?
-   Что такое fuzzing?
