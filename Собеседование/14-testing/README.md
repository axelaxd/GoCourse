# Testing, table-driven tests, mocks, httptest и fuzzing

> Быстрая шпаргалка для Go-собеседования. Источник: Неделя 12.
> Тестирование кодовой базы и API-сервисов.md.

## Что нужно уметь объяснить вслух

-   [ ] Как устроен обычный Go test?
-   [ ] Что такое table-driven test?
-   [ ] Что такое AAA pattern?
-   [ ] Чем `Errorf` отличается от `Fatal`?
-   [ ] Как запускать coverage и race detector?
-   [ ] Зачем нужны mocks?
-   [ ] Как тестировать HTTP через `httptest`?
-   [ ] Что такое fuzzing?

## Ответы на вопросы

### Как устроен обычный Go test?

Тест находится в `*_test.go`, функция обычно называется `TestXxx` и
принимает `*testing.T`. Запускается `go test`, а для всех пакетов модуля
обычно `go test ./...`.

### Что такое table-driven test?

Набор входов и ожиданий описывают как slice структур, а затем прогоняют
одинаковую проверку циклом. Часто каждый case запускают через `t.Run`,
чтобы получить отдельное имя и отчёт.

### Что такое AAA pattern?

Arrange --- подготовить данные и зависимости. Act --- выполнить
тестируемое действие. Assert --- проверить результат. Это способ сделать
тест линейным и читаемым.

### Чем `Errorf` отличается от `Fatal`?

`t.Errorf` помечает тест неуспешным, но текущая goroutine теста
продолжает выполнение. `t.Fatal`/`Fatalf` логирует сообщение и
немедленно завершает текущий тест через `FailNow`; его используют, когда
дальнейшие проверки уже бессмысленны.

### Как запускать coverage и race detector?

Например: `go test -cover ./...` для coverage и `go test -race ./...`
для динамического поиска data races. Для профиля покрытия можно
использовать `go test -coverprofile=coverage.out ./...`.

### Зачем нужны mocks?

Чтобы изолировать тестируемую единицу от внешних зависимостей и
управлять их поведением: вернуть нужный результат/ошибку и проверить
ожидаемые взаимодействия. В Go это часто удобно делать через маленькие
интерфейсы и fake/mock реализации.

### Как тестировать HTTP через `httptest`?

Для handler можно создать `req := httptest.NewRequest(...)` и
`rec := httptest.NewRecorder()`, вызвать handler и проверить
`rec.Result()`. Для кода, который сам делает HTTP-запросы, можно поднять
локальный тестовый сервер через `httptest.NewServer`.

### Что такое fuzzing?

Fuzzing автоматически генерирует множество входных значений и ищет
случаи, приводящие к panic, нарушению инвариантов или неожиданному
результату. В Go fuzz-test объявляется `FuzzXxx(f *testing.F)` и
запускается, например, `go test -fuzz=FuzzName`.

------------------------------------------------------------------------

`unit-тесты` призваны **тестировать отдельные функции** и целые пакеты /
другие структурные единицы на функциональность написанного кода. Есть
функция подстроки в строке? Будь добр, напиши тесты под отдельную
функцию со всеми кейсами, которые придумаешь: **и плохими, и хорошими**;

### Теория unit-тестирования

-   **подтверждают**, что текущая функциональность не нарушается после
    рефакторинга или добавления нового кода;
-   **позволяют** определить проблемные области, особенно связанные
    модули (tight coupling)

1.  **Метрики покрытия** **Показывают**, какая доля исходного кода была
    выполнена хотя бы одним тестом

    (однако не всегда говорит о высоком качестве тестов, из-за
    избыточности тестов)

-   **code coverage** (покрытие строк кода) Рассчитывается как
    соотношение количества строк кода, выполненных хотя бы одним тестом,
    к общему количеству строк кода

-   **branch coverage** (покрытие ветвей) **Оценивает выполнение**
    управляющих структур: условий `if`, `switch`.

    Пример: в методе с двумя ветвями (условие `if` и его `else`) если
    тест покрывает только одну из них, то `branch coverage` составит 50%

> \[!NOTE\] Тест должен быть изолированным от внешнего мира: не зависеть
> от базы данных, сети или файловой системы

### Unit-тестирование в Golang

2.  **testing** - пакет для написания тестов

```{=html}
<!-- -->
```
    // calculator.go
    package main

    func Add(a, b int) int {
        return a + b
    }

Сразу напишем первый тест:

    // calculator_test.go

    package main

    import "testing"

    func TestAdd(t *testing.T) {
        result := Add(2, 3)
        expected := 5
        
        if result != expected {
            t.Errorf("Add(2, 3) = %d; want %d", result, expected)
        }
    }

Если в тесте что-то пойдёт не так, перед пушем в Git (или уже в
пайплайне) тест упадёт, подчёркивая, что применённое изменения либо
некорректно, либо бизнес-логику и сам тест стоит менять

3.  **Основное про тесты** Все файлы, которые содержат тесты в Go, имеют
    суффикс/постфикс в виде `_test.go`

    Тестовые функции всегда начинаются с префикса `Test` и принимают
    параметр `*testing.T`.

    Принято не разделять наименования тестов через `_`, а писать как
    есть: `TestAddPositive`, `TestAddOnlyNegative`

    Пакет `testing`, а значит, и сущность `T` имеют множество методов,
    которые помогают нам управлять ожиданием от тестов.

    В данном примере применяется метод `Errorf`

    **Тесы находятся в той же директории, в которой располагается
    тестируемый код!**

По сути из этого пакета нам требуется применять на первых порах
определённый набор методов

-   `t.Errorf` используется для вывода ошибки, если условие теста не
    выполнено. При этом тест **не завершается**;
-   `t.Fail` немедленно помечает тест как неудачный, но продолжает его
    выполнение;
-   `t.FailNow` немедленно завершает выполнение теста с ошибкой;
-   `t.Fatal` аналогично с t.FailNow, но ещё выводит сообщение об
    ошибке;
-   `t.Log` выводит дополнительную информацию во время выполнения теста;
-   `t.Skip` пропускает выполнения теста. Обычно используется, когда он
    не должен выполняться при определённых условиях.

Запустить тесты проще простого:

    # Запустить все тесты в текущем пакете
    go test

    # Запустить с подробным выводом (больше информации)
    go test -v

    # Запустить конкретный тест
    go test -run TestAdd

    # Запустить все тесты проекта
    go test ./...

    # Запустить тесты несколько раз, что может помочь выявить нестабильности
    go test -count=3

    # Запустить тесты с подсчётом покрытия кода
    go test -cover

    # Запуск тестов с детектором гонок данных
    go test -race

Также при необходимости получить файл покрытия для его последующего
исследования можно воспользоваться получение финального файла и его
отображением в `HTML` или в консоли:

    # Визуализация в виде HTML-страницы
    go tool cover -html=coverage.out -o coverage.html

    # Визуализация в терминале
    go tool cover -func=coverage.out

И теперь мы можем написать больше тестов для нашего калькулятора,
запуская их по разному (`Subtract`, `Multiply`, `Divide`)

    // calculator_test.go

    package main

    import (
        "testing"
        "errors"
    )

    func TestAdd(t *testing.T) {
        result := Add(2, 3)
        expected := 5
        
        if result != expected {
            t.Errorf("Add(2, 3) = %d; want %d", result, expected)
        }
    }

    func TestSubtract(t *testing.T) {
        result := Subtract(5, 3)
        expected := 2
        
        if result != expected {
            t.Errorf("Subtract(5, 3) = %d; want %d", result, expected)
        }
    }

    func TestMultiply(t *testing.T) {
        result := Multiply(4, 3)
        expected := 12
        
        if result != expected {
            t.Errorf("Multiply(4, 3) = %d; want %d", result, expected)
        }
    }

    func TestDivide(t *testing.T) {
        // Тест для успешного деления
        result, err := Divide(6, 2)
        
        if err != nil {
            t.Errorf("Divide(6, 2) вернул ошибку: %v", err)
        }
        if result != 3 {
            t.Errorf("Divide(6, 2) = %d; want 3", result)
        }
        
        // Тест для деления на ноль
        _, err = Divide(5, 0)
        if err == nil {
            t.Error("Divide(5, 0) должен был вернуть ошибку")
        }
        if !errors.Is(err, ErrDivideByZero) {
            t.Errorf("Divide(5, 0) вернул неправильную ошибку: %v", err)
        }
    }

### ААА-патерн построения тестов

`AAA` - **Arrange, Act, Assert**

-   **A (arrange)** - подготовка, то есть часть теста, в которой
    устанавливается окружение теста, включая всевозможные инициализации;
-   **A (act)** - действие, при котором вызывается проверяемый метод или
    действие;
-   **A (assert)** - проверка результата.

```{=html}
<!-- -->
```
    func TestAdd_AAA_Pattern(t *testing.T) {
        // Arrange (подготовка) — готовим всё для теста
        a := 10
        b := 20
        expected := 30
        
        // Act (действие) — выполняем тестируемую операцию
        result := Add(a, b)
        
        // Assert (проверка) — проверяем результаты
        if result != expected {
            t.Errorf("expected %v, got %v", expected, result)
        }
    }

Но и здесь можно дать несколько ценных рекомендаций:

-   **избегай множества секций**. Каждый тест должен проверять только
    одну единицу поведения. Если требуется проверить несколько
    сценариев, лучше разбить тест на отдельные части;

-   **команды if внутри тестов считаются антипаттерном**. Тесты должны
    быть линейными и проверять строго определённое поведение.

### Табличные тесты

4.  **Table Driven Test** Создаётся настоящая таблица с набором тестовых
    кейсов, а в самом коде используется цикл для выполнения каждого из
    них.

    При таком подходе код не дублируется, тесты более структурированы и
    легче добавить новые ветки сценария.

```{=html}
<!-- -->
```
    func TestAdd_TableDriven(t *testing.T) {
        // Arrange: готовим таблицу тестовых случаев
        testCases := []struct {
            name     string        // название теста
            a, b     int           // входные данные
            expected int           // ожидаемый результат
        }{
            {"простое сложение", 2, 3, 5},
            {"сложение отрицательных чисел", -2, -3, -5},
            {"сложение с нулём", 5, 0, 5},
            {"сложение двух нулей", 0, 0, 0},
            {"большие числа", 1000000, 2000000, 3000000},
            {"смешанные числа", -10, 15, 5},
        }
        
        for _, tc := range testCases {
            t.Run(tc.name, func(t *testing.T) {
                // Act: выполняем операцию
                result := Add(tc.a, tc.b)
                
                // Assert: проверяем результат
                if result != tc.expected {
                    t.Errorf("expected %v, got %v in %s", expected, result, tc.name)
                }
            })
        }
    }

Здесь `t.Run` в виде замыкания позволяет запускать отдельные тест-кейсы
внутри одной тестовой функции.

### Дополнительные инструменты

5.  **testify** - более продвинутый пакет для тестирования кода в Go.
    Установка:

```{=html}
<!-- -->
```
    `go get github.com/stretchr/testify`

-   Его использование позволяет сделать тесты еще **более лаконичным**

```{=html}
<!-- -->
```
    // calculator_test.go
    package main

    import (
        "testing"
        "github.com/stretchr/testify/assert"
    )

    func TestAdd_WithAssert(t *testing.T) {
        result := Add(2, 3)
        assert.Equal(t, 5, result, "Сложение должно работать правильно")
    }

    func TestSubtract_WithAssert(t *testing.T) {
        result := Subtract(5, 3)
        assert.Equal(t, 2, result, "Вычитание должно работать правильно")
    }

    func TestMultiply_WithAssert(t *testing.T) {
        result := Multiply(4, 3)
        assert.Equal(t, 12, result, "Умножение должно работать правильно")
    }

    func TestDivide_WithAssert(t *testing.T) {
        // Успешное деление
        result, err := Divide(6, 2)
        assert.NoError(t, err, "Деление 6 на 2 не должно возвращать ошибку")
        assert.Equal(t, 3, result, "Результат деления 6 на 2 должен быть 3")
        // Деление на ноль
        
        result, err = Divide(5, 0)
        assert.Error(t, err, "Деление на ноль должно возвращать ошибку")
        assert.Equal(t, 0, result, "При делении на ноль результат должен быть 0")
        assert.EqualError(t, err, "деление на ноль", "Текст ошибки должен быть 'деление на ноль'")
    }

6.  **Моки** Тестовые объекты, которые иммитируют поведение реальных
    компонентов системы, но не содержат их сложной логики.

    Они используются для изоляции тестируемого кода, проверки
    взаимодействия компонентов и создания заглушек (фейков) вместо
    реальных баз данных или API, что ускоряет тесты

Усложним пример с калькулятором:

    // logger.go
    package main

    import "fmt"

    // Logger — интерфейс для логирования операций
    type Logger interface {
        LogOperation(operation string, a, b, result int) error
    }

    // DatabaseLogger — реализация, которая пишет в базу данных
    type DatabaseLogger struct {
        // представим, что здесь подключение к базе данных
    }

    func (dl *DatabaseLogger) LogOperation(operation string, a, b, result int) error {
        // В реальной жизни здесь был бы код для записи в БД
        fmt.Printf("Logging: %s(%d, %d) = %d\n", operation, a, b, result)
        return nil
    }

    // EnhancedCalculator — калькулятор с логированием
    type EnhancedCalculator struct {
        logger Logger
    }

    func NewEnhancedCalculator(logger Logger) *EnhancedCalculator {
        return &EnhancedCalculator{logger: logger}
    }

    func (ec *EnhancedCalculator) AddAndLog(a, b int) (int, error) {
        result := Add(a, b)
        if err := ec.logger.LogOperation("Add", a, b, result); err != nil {
            return 0, fmt.Errorf("ошибка логирования: %w", err)
        }
        return result, nil
    }

Мы добавили в пакет с калькулятором файл с логгером и используем его
функциональность через интерфейс. И как нам это теперь тестировать? С
помощью мока

Создадим мок:

    // logger_mock_test.go
    package main

    import (
        "github.com/stretchr/testify/mock"
    )

    // MockLogger — наш мок для интерфейса Logger
    type MockLogger struct {
        mock.Mock
    }

    func (m *MockLogger) LogOperation(operation string, a, b, result int) error {
        args := m.Called(operation, a, b, result)
        return args.Error(0)
    }

Этот мок имплементирует метод интерфейса, за счёт чего мы теперь можем
сделать так:

    func TestEnhancedCalculator_AddAndLog_WithMock(t *testing.T) {
        // Arrange
        mockLogger := new(MockLogger)
        calculator := NewEnhancedCalculator(mockLogger)
        
        // Настраиваем ожидания для мока
        mockLogger.On("LogOperation", "Add", 2, 3, 5).Return(nil)
        
        // Act
        result, err := calculator.AddAndLog(2, 3)
        
        // Assert
        assert.NoError(t, err, "Операция должна выполниться без ошибок")
        assert.Equal(t, 5, result, "Результат сложения должен быть 5")
        
        // Проверяем, что все ожидания для мока выполнены
        mockLogger.AssertExpectations(t)
    }

    func TestEnhancedCalculator_AddAndLog_LoggingError(t *testing.T) {
        // Arrange
        mockLogger := new(MockLogger)
        calculator := NewEnhancedCalculator(mockLogger)
        
        // Настраиваем мок так, чтобы он возвращал ошибку
        expectedError := fmt.Errorf("база данных недоступна")
        mockLogger.On("LogOperation", "Add", 2, 3, 5).Return(expectedError)
        
        // Act
        result, err := calculator.AddAndLog(2, 3)
        
        // Assert
        assert.Error(t, err, "Должна быть ошибка логирования")
        assert.Contains(t, err.Error(), "ошибка логирования",
            "Ошибка должна содержать информацию о логировании")
        assert.Equal(t, 0, result, "При ошибке результат должен быть 0")
        
        // Проверяем, что мок был вызван правильно
        mockLogger.AssertExpectations(t)
    }

### Вспомогательные функции

В пакете `testing` есть несколько замечательных вспомогательных функций,
которые помогают дополнительно управлять жизненным циклом наших тестов

-   `t.Helper` - функция, которой мы можем маркировать вспомогательные
    для теста функции/методы, чтобы не видеть в логах указание на строку
    внутри вспомогательной функции

```{=html}
<!-- -->
```
    func assertEqual(t *testing.T, actual, expected string) {
        t.Helper() // Помечает эту функцию как хелпер
        if actual != expected {
            t.Errorf("ожидалось: %s, получено: %s", expected, actual)
        }
    }

    func TestExample(t *testing.T) {
        assertEqual(t, "a", "b") // Ошибка укажет на эту строку
    }

-   `t.Parallel` - используется в случае необходимости запуска
    нескольких тестов конкурентно.

-   `t.Skip` - для пропуска выполнения теста

### Тестирование HTTP-api

7.  **httptest**

Предположим, что у нас есть очень важная ручка, которая принимает в
query-параметре строку и меняет её регистр на верхний. С HTTP-сервером
это выглядело бы примерно так:

    // Req: http://localhost:8080/upper?word=abc
    // Res: ABC
    func upperCaseHandler(w http.ResponseWriter, r *http.Request) {
        query, err := url.ParseQuery(r.URL.RawQuery)
        if err != nil {
            w.WriteHeader(http.StatusBadRequest)
            fmt.Fprintf(w, "invalid request")
            
            return
        }
        
        word := query.Get("word")
        if len(word) == 0 {
            w.WriteHeader(http.StatusBadRequest)
            fmt.Fprintf(w, "missing word")
            
            return
        }
        
        w.WriteHeader(http.StatusOK)
        fmt.Fprintf(w, strings.ToUpper(word))
    }

    func main() {
        http.HandleFunc("/upper", upperCaseHandler)
        log.Fatal(http.ListenAndServe(":8080", nil))
    }

И с помощью `httptest` мы действительно можем замокать всё необходимое
поведение. Для этого следует использовать две сущности:

-   **\*httptest.ResponseRecorder** заменяет нам `http.ResponseWriter`
    Его можно получить через функцию `httptest.NewRecorder()` И уже со
    структурой `*httptest.ResponseRecorder` можно вытворять много
    интересного

-   **\*httptest.NewRequest()** возвращает нам тот же самый
    `http.Request`, который и используется в эндпоинте

```{=html}
<!-- -->
```
    func TestUpperCaseHandler(t *testing.T) {
        req := httptest.NewRequest(http.MethodGet, "/upper?word=abc", nil)
        w := httptest.NewRecorder()
        
        upperCaseHandler(w, req)
        
        res := w.Result()
        defer res.Body.Close() // не забываем закрывать тело ответа во избежание утечек памяти
        
        data, err := io.ReadAll(res.Body)
        if err != nil {
            t.Errorf("expected error to be nil got %v", err)
        }
        
        if string(data) != "ABC" {
            t.Errorf("expected ABC got %v", string(data))
        }
    }

Но что делать, если необходимо протестировать работу хендлера на
клиентской стороне? **Требуется замокать сервер**

    type Client struct {
        url string
    }

    func NewClient(url string) Client {
        return Client{url}
    }

    func (c Client) UpperCase(word string) (string, error) {
        res, err := http.Get(c.url + "/upper?word=" + word)
        if err != nil {
            return "", fmt.Errorf("unable to complete Get request: %w", err)
        }
        
        defer res.Body.Close()
        
        out, err := io.ReadAll(res.Body)
        if err != nil {
            return "", fmt.Errorf("unable to read response data: %w", err)
        }
        
        return string(out), nil
    }

По сути, мы делаем тут то же самое, что и в тесте на стороне сервиса.
Только на стороне клиента нам нужно протестировать именно
функцию `UpperCase`, а не сам вызов замоканного сервера. Почему? Потому
что она может вытворять с пришедшими данными что-то дополнительное.
Поэтому мы проверяем не столько сам HTTP-вызов, сколько работу с
результатами вызова.

Для этих манипуляций нам всего-навсего нужен
метод `httptest.NewServer()`, который создаст мок сервера с необходимой
нам в данном моменте логикой работы конкретного эндпоинта. Это обёртка,
которая содержит в себе много интересного из настоящего сервера.

И как проводить тестирование? Очень просто: мы прописываем хендлеру
возвращать то, что мы хотим. Вот и всё: \`\`

    func TestClientUpperCase(t *testing.T) {
        expected := "dummy data"
        svr := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            fmt.Fprintf(w, expected)
        }))
        
        defer svr.Close() // да, сервер тестовый, но и его надо бы закрыть
        
        c := NewClient(svr.URL)
        res, err := c.UpperCase("anything")
        if err != nil {
            t.Errorf("expected err to be nil got %v", err)
        }
        
        res = strings.TrimSpace(res) // избавляемся от ненужных переносов в конце строки
        if res != expected {
            t.Errorf("expected res to be %s got %s", expected, res)
        }
    }

Здесь мы точно не получим ошибку, потому что хендлер возвращает ровно
то, что получает, без использования логики прошлого примера.

### Фаззинг

8.  **testing.F** - фаззер Ещё один вид тестирования, когда функции (в
    случае unit-тестов) или программе подаются случайные, некорректные и
    неожиданные входные данные для выявления уязвимостей и багов.

    Как водится, тесты такого формата начинаются обязательно с
    префикса `Fuzz...`
