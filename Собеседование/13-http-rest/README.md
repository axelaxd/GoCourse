# HTTP, REST, client и middleware

> Быстрая шпаргалка для Go-собеседования. Источник: Неделя 11. HTTP протокол и работа с ними в REST-архитектуре.md.

## Что нужно уметь объяснить вслух

- [ ] Что такое `http.Handler` и `ServeHTTP`?
- [ ] Чем `HandlerFunc` удобен?
- [ ] Что находится в `http.Request` и `http.ResponseWriter`?
- [ ] Как читать query, headers и body?
- [ ] Почему HTTP 404/500 не обязательно даёт `err != nil` у клиента?
- [ ] Почему нужно закрывать `resp.Body`?
- [ ] Что такое middleware?
- [ ] Зачем настраивать timeout у `http.Server` / `http.Client`?

## Ответы на вопросы

### Что такое `http.Handler` и `ServeHTTP`?

`http.Handler` — интерфейс с методом `ServeHTTP(http.ResponseWriter, *http.Request)`. Любой тип с этим методом может обрабатывать HTTP-запросы.

### Чем `HandlerFunc` удобен?

`http.HandlerFunc` — адаптер типа функции. Он позволяет обычную функцию с сигнатурой `func(w http.ResponseWriter, r *http.Request)` использовать как `http.Handler`, не создавая отдельный struct с методом.

### Что находится в `http.Request` и `http.ResponseWriter`?

`Request` содержит method, URL, headers, body, context и другую информацию входящего запроса. `ResponseWriter` — интерфейс, через который handler задаёт response headers/status и пишет body.

### Как читать query, headers и body?

Query: `r.URL.Query().Get("q")`. Header: `r.Header.Get("Authorization")`. JSON body: `json.NewDecoder(r.Body).Decode(&dst)`. Body — поток; повторное чтение без специального восстановления обычно уже не вернёт исходные данные.

### Почему HTTP 404/500 не обязательно даёт `err != nil` у клиента?

Для `http.Client.Do` успешное получение HTTP-ответа — успешная транспортная операция, даже если статус 500. `err` обычно относится к созданию/отправке запроса, сети, redirect policy и подобным проблемам; статус приложения проверяется отдельно через `resp.StatusCode`.

### Почему нужно закрывать `resp.Body`?

Чтобы освободить ресурсы ответа и позволить `Transport` корректно переиспользовать соединение, когда это возможно. Обычно `defer resp.Body.Close()` ставят сразу после успешной проверки `err`.

### Что такое middleware?

Это handler-обёртка: функция обычно принимает `next http.Handler` и возвращает новый `http.Handler`, выполняющий код до/после `next.ServeHTTP`. Так выносят логирование, auth, tracing, recovery и другую сквозную логику.

### Зачем настраивать timeout у `http.Server` / `http.Client`?

Без ограничений медленный или зависший peer способен удерживать goroutine, соединения и другие ресурсы неопределённо долго. На клиенте важен общий/детализированный timeout; на сервере — подходящие read/write/idle timeouts и context deadlines с учётом характера сервиса.

---

## Конспект

### 1. **Основная библиотека** - **net/http**
Позволяет запускать HTTP-сервер за несколько строк:

```go
package main

import (
    "fmt"
    "net/http"
)

// weatherHandler обрабатывает запросы к /weather и возвращает приветственное сообщение
func weatherHandler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "Добро пожаловать в сервис погоды!")
}

func main() {
	// Регистрируем обработчик маршрута /weather
    http.HandleFunc("/weather", weatherHandler)
    fmt.Println("Сервер запущен на порту 8080...")
    
    // Запуск сервера на порту 8080
    err := http.ListenAndServe(":8080", nil)
    if err != nil {  
       fmt.Println("Ошибка запуска сервера:", err)  
    }
}
```

**Главная функция здесь** `http.ListenAndServe()`, которая принимает порт интерфейса сервера и обработчик запросов. В контексте Go обработчик (или handler) — объект с методом `ServeHTTP(w http.ResponseWriter, r *http.Request)`, который реагирует на входящие HTTP-запросы

Теперь, если перейти в браузере по адресу `http://localhost:8080/weather`, сервер вернёт строку


Обрати внимание, что функция-обработчик `weatherHandler` принимает **два параметра**:

- `http.ResponseWriter` — это интерфейс для отправки ответа клиенту (через него мы пишем данные и заголовки);
- `http.Request` содержит информацию о запросе (URL, метод, тело).

### 2. **Другой вариант создания сервера** - **switch**
Зачастую нам всё же хочется настраивать сервер под себя

И его можно создать таким образом:
```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"
)

func handler(w http.ResponseWriter, r *http.Request) {
	switch r.URL.Path {
	case "/":
		fmt.Fprintln(w, "главная")
	case "/about":
		fmt.Fprintln(w, "о проекте")
	default:
		http.NotFound(w, r)
	}
	
}
func main() {
	srv := &http.Server{
		Addr:           ":8080",
		Handler:        http.HandlerFunc(handler),
		ReadTimeout:    5 * time.Second,
		WriteTimeout:   10 * time.Second,
	}
	
	log.Fatal(srv.ListenAndServe())
}
```

### 3. **Еще более кастомный** - **tcp**
```go
package main

import (
	"fmt"
	"log"
	"net"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "Hello via Serve(listener)")
}

func main() {
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		log.Fatal(err)
	}
	
	srv := &http.Server{Handler: http.HandlerFunc(handler)}
	log.Fatal(srv.Serve(ln))
}
```

### 4. **Пример обединения логики с единой структурой**:
```go
package main

import (
	"fmt"
	"log"
	"net/http"
)

type App struct{}

func (a *App) home(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "главная")
}

func (a *App) about(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintln(w, "о проекте")
}

func (a *App) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	switch r.URL.Path {
	case "/":
		a.home(w, r)
	case "/about":
		a.about(w, r)
	default:
		http.NotFound(w, r)
	}
}

func main() {
	app := &App{}
	srv := &http.Server{Addr: ":8080", Handler: app}
	
	log.Fatal(srv.ListenAndServe())
}
```

Благодаря методу `ServeHTTP` наш `App` теперь может участвовать в обработке запросов, регистрируясь в нашем сервере.

#### Еще про обработчики и маршрутизацию
Для более сложных API можно использовать несколько обработчиков для разных путей.

Например, как в остальных примерах, или можно написать такое для трекера задач:

```go
http.HandleFunc("/tasks", tasksHandler)    // Вывод списка задач
http.HandleFunc("/add", addTaskHandler)    // Добавление новой
```

Если теперь сервер получит запрос к `/tasks`, вызовется функция `tasksHandler`, а к `/add` — `addTaskHandler`. `net/http` сам сравнивает URL запроса с зарегистрированными маршрутами и вызывает нужную функцию.


#### Разбор запросов к серверу

Когда клиент делает HTTP-запрос, он отправляет метод (`GET`, `POST`, другие), URL, заголовки и опционально тело (body). В обработчике через объект `*http.Request` можно получить всю эту информацию:

- _Метод запроса_. Доступен как `r.Method` (строка, например `GET` или `POST`). Часто используют условные операторы, чтобы различать логику для разных методов на одном пути.

- _Путь и строка запроса_. `r.URL.Path` содержит путь (например, `/tasks`), а `r.URL.Query()` — объект с query-параметрами (часть URL после `?`). Например, для запроса `/tasks?done=true` вызов `r.URL.Query().Get("done")` вернёт строку `true`. У query-параметров всегда тип string, но при необходимости их нужно конвертировать.

- _Заголовки_ (headers). Хранятся в `r.Header` (`map[string][]string`). Можно получить конкретный заголовок, например `r.Header.Get("Content-Type")`.

- _Тело запроса_ (body). Для методов вроде `POST` или `PUT` клиент может отправлять данные в теле. В Go тело запроса доступно как `r.Body` (`io.ReadCloser`). Его можно прочитать, например через `io.ReadAll(r.Body)`, или использовать `json.NewDecoder(r.Body)` для разбора `JSON`. Важно: `r.Body` можно читать только один раз. После чтения (или если не предполагается чтение) его обычно закрывают: `defer r.Body.Close()`.

Рассмотрим обработчик, который разбирает параметры запроса. Допустим, для маршрута `/weather` мы хотим запросить погоду по названию города, принимая название как query-параметр:

```go
package main  
  
import (  
    "encoding/json"  
    "fmt"    
    "net/http"
)
  
// weatherHandler теперь анализирует переданный город из query-параметра
func weatherHandler(w http.ResponseWriter, r *http.Request) {
    city := r.URL.Query().Get("city") // Данные из параметра ?city
    if city == "" {
        http.Error(w, "Не указан город", http.StatusBadRequest)
        return
    }
    fmt.Fprintf(w, "Запрошена погода для города: %s", city)
}
  
func main() {  
    http.HandleFunc("/weather", weatherHandler)
  
    fmt.Println("Сервер запущен на порту 8080...")  
    err := http.ListenAndServe(":8080", nil)
    if err != nil {  
       fmt.Println("Ошибка запуска сервера:", err)  
    }  
}
```

### 6. **Как прописывать ошибки** - **http.Error**

Обрати внимание на `http.Error(w, msg, code)`. Это удобная функция, которая комбинирует установку статуса ответа и тело с сообщением об ошибке.

В последних версиях Go стали поддерживаться и path-параметры

```go
http.HandleFunc("GET /users/{id}", func(w http.ResponseWriter, r *http.Request) {
		id := r.PathValue("id") // Строка из сегмента пути
		if id == "" {
			http.Error(w, "missing id", http.StatusBadRequest)
			return
		}
		// ...
	})
```


Как же читать тела `POST`-запросов?

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
)

type createUserReq struct {
	Name  string `json:"name"`
	Email string `json:"email"`
}

func createUser(w http.ResponseWriter, r *http.Request) {
	if r.Method != http.MethodPost {
		http.Error(w, "method not allowed", http.StatusMethodNotAllowed)
		return
	}
	defer r.Body.Close()
	
	var req createUserReq
	dec := json.NewDecoder(r.Body)
	dec.DisallowUnknownFields() // Опционально: лишние поля в JSON → ошибка
	
	if err := dec.Decode(&req); err != nil {
		http.Error(w, "invalid json", http.StatusBadRequest)
		return
	}
	
	// Пустое тело: Decode вернёт EOF — при желании отдельно проверяйте Content-Length
	
	w.Header().Set("Content-Type", "application/json")
	_ = json.NewEncoder(w).Encode(map[string]string{
		"ok":    "true",
		"saved": req.Name,
	})
}

func main() {
	http.HandleFunc("/users", createUser)
	log.Fatal(http.ListenAndServe(":8080", nil))
}
```

#### Маршрутизация запросов

### 4. **Маршрутизация** - **mux**
Создаётся при помощи http.NewServeMux()

В Go существует маршрутизатор по умолчанию. Это тип `*http.ServeMux` (multiplexer), который реализует интерфейс `http.Handler`. Этот тип умеет соотнести URL запроса с зарегистрированными шаблонами

```go
func main() {
	mux := http.NewServeMux()
	
	// Регистрируем обработчик в маршрутизаторе
	mux.HanldeFunc("/weather", weatherHandler)
	
	fmt.Println("Сервер запущен на 8080...")
	http.ListenAndServe(":8080", mux)
}
```

Или для более сложного случая:

```go
mux := http.NewServeMux()

mux.HandleFunc("/tasks", tasksHandler) // Список задач
mux.HandleFunc("/tasks/add", addTaskHandler) // Создание новой
mux.HandleFunc("/tasks/", taskNotFoundHandler) // По умолчанию

http.ListenAndServe(":8080", mux)
```

В этом примере мы создали новый маршрутизатор `mux` и зарегистрировали на нём три обработчика. В `ListenAndServe` мы передаём mux вместо `nil`, и сервер будет использовать наш локальный маршрутизатор.

### HTTP-клиенты на Go

Проще всего сделать запросы с помощью высокоуровневых функций:
- `http.Get`
- `http.Post`
- `http.Head`

Еще можно создавать свой объект `http.Client` для настроек или даже низкоуровневые запросы через `http.NewRequest` + `Client.Do`

Рассмотрим пример `GET`-запроса к внешнему сервису:

```go
import (
    "encoding/json"
    "fmt"
    "net/http"
)

type WeatherResponse struct {  
    CurrentWeather struct {  
       Temperature float64 `json:"temperature"`  
    } `json:"current_weather"`  
}  
  
type Coordinates struct {  
    Lat  float64  
    Long float64  
}  
  
var cityCoordinates = map[string]Coordinates{  
    "Москва": {Lat: 55.7558, Long: 37.6173},  
    "Лондон": {Lat: 51.5074, Long: -0.1278},  
    "Нью-Йорк": {Lat: 40.7128, Long: -74.0060},  
}  
  
func getWeather(city string) (float64, error) {  
    coords, exists := cityCoordinates[city]  
    if !exists {  
       return 0, fmt.Errorf("город не найден")  
    }  
  
    baseURL, err := url.Parse("https://api.open-meteo.com/v1/forecast")  
    if err != nil {  
       return 0, err  
    }  
  
    queryParams := url.Values{}  
    queryParams.Set("latitude", fmt.Sprintf("%f", coords.Lat))  
    queryParams.Set("longitude", fmt.Sprintf("%f", coords.Long))  
    queryParams.Set("current_weather", "true")  
    baseURL.RawQuery = queryParams.Encode()  
  
    resp, err := http.Get(baseURL.String())  
    if err != nil {  
       return 0, err  
    }  
    defer resp.Body.Close()  
  
    var data WeatherResponse  
    if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {  
       return 0, err  
    }  
    return data.CurrentWeather.Temperature, nil  
}
```

Ключевые моменты при использовании HTTP-клиента:
- Функция `http.Get(url)` возвращает (`*http.Response`, error). Ошибка `err` будет не `nil`, только если запрос не смог вообще выполниться (нет сети, DNS, тайм-аут). Если же удалённый сервер ответил с кодом ошибки (404, 500 и так далее), `err` всё равно будет `nil`. Поэтому проверяй `resp.StatusCode` отдельно.

- Всегда закрывай `resp.Body` (через `defer` сразу после проверки ошибки). Иначе соединение может зависнуть и не вернуться в пул, особенно если ответ дочитан не до конца.

- Чтение ответа можно делать через `io.ReadAll`, как выше, или построчно, или сразу парсить JSON: `json.NewDecoder(resp.Body).Decode(&obj)`, если ожидается JSON.

- Если нужен POST-запрос, можно использовать `http.Post(url, contentType, bodyReader)`. Например:

```go
resp, err := http.Post("https://httpbin.org/post", "application/json", strings.NewReader(`{"x": 5}`))
```

Это отправит JSON {"x":5} на указанный URL. Но часто удобнее вручную собрать запрос, особенно если нужны нестандартные заголовки.

- Для гибкости можно создать запрос вручную:
```go
req, _ := http.NewRequest("PUT", "http://example.com/resource/42", bodyReader)
req.Header.Set("Authorization", "Bearer TOKEN")
resp, err := http.DefaultClient.Do(req)
```

Здесь мы установили заголовок `Authorization` и отправили запрос через `http.DefaultClient` (стандартный клиент). Напомним, что `DefaultClient` — это готовый `&http.Client{}`. Можно создать и свой, особенно если нужны настройки (тайм-ауты, прокси, кастомный `Transport`).

#### Middlewares

При работе с веб-серверами часто возникает потребность выполнять некоторый код до или после основного обработчика маршрута — например, логировать запросы, проверять авторизацию, обрабатывать ошибки единообразно или изменять выходные данные (сжатие, кеширование). Такой шаблон называется **middleware**(промежуточное ПО).

В Go нет специального типа, но его легко реализовать самостоятельно, ведь обработчики - это интерфейсы и функции.

Самый распространённый подход - написать функцию, которая принимает один `http.Handler` и возвращает другой `http.Handler`, оборачивающий логику вокруг исходного вызова

Например, middleware для логирования запросов:
```go
func loggingMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        fmt.Printf("=> %s %s\n", r.Method, r.URL.Path)  // Логируем метод и путь
        next.ServeHTTP(w, r)                            // Вызываем следующий обработчик
        fmt.Println("<= отправлен ответ клиенту")
    })
}
```

Здесь `loggingMiddleware` — функция, возвращающая новый обработчик. При каждом запросе он сначала пишет в лог информацию о запросе, затем вызывает `next.ServeHTTP(w, r)`, а после этого может выполнить ещё какие-то действия (в данном случае просто пишет, что ответ отправлен). Когда обработчик вызывает `next.ServeHTTP(w, r)`, он передаёт обработку реальному маршруту.

Мы можем применять эту функцию к нашим маршрутизаторам или отдельным обработчикам. Например, чтобы логировать абсолютно все запросы на сервере, можно обернуть весь `ServeMux`:

```go
mux := http.NewServeMux()
// ... регистрация маршрутов ...
loggedMux := loggingMiddleware(mux)
http.ListenAndServe(":8080", loggedMux)
```

Теперь каждый запрос будет проходить через `loggingMiddleware` перед тем, как попасть к реальным отработчикам. В консоли будут выводиться строки вида:

```text
=> GET /weather
<= отправлен ответ клиенту
```

Например, проверку авторизации: middleware читает заголовок `Authorization` или `cookie`, решает, можно ли пускать дальше, и либо возвращает 401, либо вызывает `next`.

Другая распространённая прослойка — установка общих заголовков ответа (например, `Content-Security-Policy`) для всех запросов. Это тоже можно сделать через middleware, вызывая `w.Header().Set(...)` до `next.ServeHTTP`.

```go
package main

import (
	"log"
	"net/http"
	"time"
	"github.com/google/uuid"
)

func withRequestID(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		id := uuid.NewString()
		w.Header().Set("X-Request-ID", id)
		next.ServeHTTP(w, r)
	})
}

func withLogging(next http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		start := time.Now()
		next.ServeHTTP(w, r)
		log.Printf("%s %s (%s)", r.Method, r.URL.Path, time.Since(start))
	})
}

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("OK"))
}

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", hello)
	h := withRequestID(withLogging(http.HandlerFunc(hello)))
    http.ListenAndServe(":8080", h)
}
```

Здесь мы передаём `hello` как объект, который удовлетворяет интерфейсу, а не как функцию. И в этом случае порядок объявления играет роль. Сначала при входящем запросе сработает функция `withRequestID`, потом `withLogging` и только затем основная функция `hello`.

Как всегда, такую запись можно сделать красивее:

```go
package main

import (
	"log"
	"net/http"
)

// chain склеивает middleware слева направо: сначала outer, потом inner, потом h
func chain(h http.Handler, middlewares ...func(http.Handler) http.Handler) http.Handler {
	for i := len(middlewares) - 1; i >= 0; i-- {
		h = middlewares[i](h)
	}
	return h
}

// ...

func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/", hello)
	root := chain(mux, withLogging, withRequestID)
	log.Fatal(http.ListenAndServe(":8080", root))
}
```
