# GoCourse

Этот репозиторий содержит основной конспект курса по Go и короткие материалы для быстрого повторения отдельных тем.

> **Как пользоваться:** открывай полную неделю, если нужно восстановить тему целиком. Перед собеседованием используй ссылку **«Быстро повторить»** — это небольшой конспект по одной из самых полезных подтем недели.

## Содержание

### 1. [Неделя 1. Введение в Go. Простые типы данных и управляющие конструкции](./weeks/week-01/README.md)

[Строки, UTF-8, byte и rune](./weeks/week-01/strings-runes.md)

### 2. [Неделя 2. Составные типы данных. Знакомство с исходным кодом Go](./weeks/week-02/README.md)

[Слайсы и map: что важно помнить](./weeks/week-02/slices-maps.md)

### 3. [Неделя 3. Функции. Структуры и методы. Интерфейсы](./weeks/week-03/README.md)

[Интерфейсы, any и type assertion](./weeks/week-03/interfaces.md)

### 4. [Неделя 4. Дженерики. Ошибки и их обработка. Паники](./weeks/week-04/README.md)

[defer, panic и recover](./weeks/week-04/defer-panic-recover.md)

### 5. [Неделя 5. Структура проекта](./weeks/week-05/README.md)

[Пакеты, модули, go.mod и go.sum](./weeks/week-05/packages-modules.md)

### 6. [Неделя 6. Конкурентность и параллелизм](./weeks/week-06/README.md)

[Горутины, каналы, select и deadlock](./weeks/week-06/goroutines-channels-select.md)

### 7. [Неделя 7. Контекст и примитивы синхронизации](./weeks/week-07/README.md)

[context, Mutex и WaitGroup](./weeks/week-07/context-sync.md)

### 8. [Неделя 8. Работа с файлами](./weeks/week-08/README.md)

[io.Reader / io.Writer и буферизация](./weeks/week-08/io-reader-writer.md)

### 9. [Неделя 9. Тулинг в Golang](./weeks/week-09/README.md)

[Инструменты перед PR](./weeks/week-09/tooling-checklist.md)

### 10. [Неделя 10. Конфигурация, JSON и YAML](./weeks/week-10/README.md)

[JSON: Marshal, Unmarshal и struct tags](./weeks/week-10/json-tags.md)

### 11. [Неделя 11. HTTP и REST-архитектура](./weeks/week-11/README.md)

[Handler и middleware](./weeks/week-11/http-middleware.md)

### 12. [Неделя 12. Тестирование кодовой базы и API-сервисов](./weeks/week-12/README.md)

[Table-driven tests и httptest](./weeks/week-12/table-tests-httptest.md)

---

## Быстрый маршрут перед собеседованием

Если времени мало, в первую очередь повтори:

1. [Слайсы и map](./weeks/week-02/slices-maps.md)
2. [Интерфейсы](./weeks/week-03/interfaces.md)
3. [defer, panic и recover](./weeks/week-04/defer-panic-recover.md)
4. [Горутины, каналы и select](./weeks/week-06/goroutines-channels-select.md)
5. [context, Mutex и WaitGroup](./weeks/week-07/context-sync.md)
6. [HTTP Handler и middleware](./weeks/week-11/http-middleware.md)
7. [Table-driven tests и httptest](./weeks/eek-12/table-tests-httptest.md)
