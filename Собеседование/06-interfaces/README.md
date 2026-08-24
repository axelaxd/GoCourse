# Interfaces, type assertion и method set

> Быстрая шпаргалка для Go-собеседования. Источник: Неделя 3. Функции. Структуры и методы. Интерфейсы.md, Неделя 9. Тулинг в Golang.md.

## Что нужно уметь объяснить вслух

- [ ] Как тип реализует интерфейс в Go?
- [ ] Что такое пустой интерфейс / `any`?
- [ ] Что такое type assertion и type switch?
- [ ] Что такое method set?
- [ ] Чем реализация интерфейса для `T` отличается от `*T`?
- [ ] Почему интерфейс часто объявляют в месте использования?

## Ответы на вопросы

### Как тип реализует интерфейс в Go?

Неявно. Никакого `implements` нет: если method set типа содержит все методы интерфейса с нужными сигнатурами, тип удовлетворяет интерфейсу.

### Что такое пустой интерфейс / `any`?

`any` — alias для `interface{}`. Пустой интерфейс не требует методов, поэтому ему удовлетворяет любое значение. Использовать его стоит только там, где тип действительно заранее неизвестен.

### Что такое type assertion и type switch?

Assertion извлекает конкретный тип: `v, ok := x.(T)`. Type switch позволяет обработать несколько возможных динамических типов: `switch v := x.(type) { ... }`.

### Что такое method set?

Method set — набор методов, который участвует в проверке реализации интерфейсов. У `T` в method set входят методы с receiver `T`; у `*T` --- методы с receiver `T` и `*T`.

### Чем реализация интерфейса для `T` отличается от `*T`?

Если нужный метод объявлен с receiver `T`, интерфейсу обычно удовлетворяют и `T`, и `*T`. Если обязательный метод имеет receiver `*T`, то интерфейсу удовлетворяет `*T`, но не `T`.

### Почему интерфейс часто объявляют в месте использования?

Потребитель лучше знает минимальное поведение, которое ему нужно. Маленький интерфейс на стороне потребителя уменьшает связанность и позволяет поставщику не зависеть от интерфейса конкретного клиента.

------------------------ 10\. **Интерфейсы: определение поведения** **Интерфейс** в Go - это абстрактный тип, который определяет набор конкретных методов ------------------------ // Где-то в другом пакете будет не важно, что мы используем: метод напрямую или интерфейс. Мы все равно притянем пакет format

package example

import github.com/..../format

func example(f format.Formatter) { svc := format.NewService() svc.Format()

f.Format() } ------------------------

// Лучше всё-таки вынести интерфейс в место использования. Так не получим лишних импортов и полностью абстрагируем логику

package example

type Formatter interface { Format() string }

func example(f Formatter) { f.Format() } \`\`\`

---

## Конспект

### 10. **Интерфейсы: определение поведения**
**Интерфейс** в Go - это абстрактный тип, который определяет набор конкретных методов

```go
func main() {
	rect1 := Rectangle{10, 5}
	rect1.Info() // 10 5
	fmt.Println(Shape.Area(rect1)) // 50
	fmt.Println(Shape.Perimeter(rect1)) // 30
}

type Shape interface {
	Area() float64
	Perimeter() float64
}

type Rectangle struct {
	Width float64
	Height float64
}

func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

func (r Rectangle) Perimeter() float64 {
	return 2 * (r.Width + r.Height)
}

func (r Rectangle) Info() {
	fmt.Println("Width:", r.Width)
	fmt.Println("Height:", r.Height)
}
```

### 11. **Использование интерфейсов**
В примере ниже функция `printShapeInfo` принимает параметр типа интерфейс

```go
func printShapeInfo(s Shape) {
    fmt.Printf("Площадь: %.2f\n", s.Area())
    fmt.Printf("Периметр: %.2f\n", s.Perimeter())
}

func main() {
    rect := Rectangle{Width: 10, Height: 5}
    circle := Circle{Radius: 3}
    
    // Один и тот же код работает с разными типами
    fmt.Println("Прямоугольник:")
    printShapeInfo(rect)
    
    fmt.Println("\nКруг:")
    printShapeInfo(circle)
}
```

### 12. **Пустой интерфейс** 
`interface{}` не содержит ни одного метода в своей определении

Поскольку любой тип в Go может иметь ноль методов, **любой тип удовлетворяет пустому интерфейсу**. Это делает `interface{}/any` универсальным типом

### 13. **Type Assertion**
Для извлечения конкретного типа из значения интерфейса используется **type assertion**

При котором мы пытаемся привести переменную типа интерфейса к интересующему нам типу и проверяем приведение через булевое `ok`

```go
func processValue(value interface{}) {
	// безопасная проверка через идиому "comma, ok"
	if str, ok := value.(string); ok {
		fmt.Printf("Это строка: %s (длина: %d)\n", str, len(str))
	} else if num, ok := value.(int); ok {
		fmt.Printf("Это число: %d (квадрат: %d)\n", num, num*num)
	} else {
		fmt.Println("Неизвестный тип")
	}
}

  

func main() {
	processValue("Hello")
	processValue(42)
	processValue(3.14)
}
```

### 14. **Type Switch**
Более элегантный способ работы с разными типами через **type switch**

```go
func describe(value interface{}) {
    switch v := value.(type) {
    case string:
        fmt.Printf("Строка длиной %d: %s\n", len(v), v)
    case int:
        fmt.Printf("Целое число: %d\n", v)
    case float64:
        fmt.Printf("Число с плавающей точкой: %.2f\n", v)
    case Rectangle:
        fmt.Printf("Прямоугольник: площадь = %.2f\n", v.Area())
    default:
        fmt.Printf("Неизвестный тип: %T\n", v)
    }
}
```

#### Ограничения и особенности пустого интерфейса
- Потеря статической типизации
```go
func demonstrateTypeLoss() {
    var x any = 42
    
    fmt.Println(x + 10) // Ошибка компиляции! Go не знает, что x                                                — это число
    
    // Нужно приведение типа через type assertion
    if num, ok := x.(int); ok {
        fmt.Println(num + 10) // 52
    }
}
```

- Производительность
```go
// Медленнее - нужны проверка типов и приведения
func slowSum(values []any) int {
    sum := 0
    for _, v := range values {
        if num, ok := v.(int); ok {
            sum += num
        }
    }
    return sum
}
// Быстрее - работа с конкретным типом
func fastSum(values []int) int {
    sum := 0
    for _, v := range values {
        sum += v
    }
    return sum
}
```

- Отсутствие проверки типов на этапе компиляции
```go
func riskyCode() {
    var data any = "строка"
    
    // Код ниже скомпилируется, но упадёт в runtime!
    number := data.(int) // panic: interface conversion: interface {} is string, not int
    
    // Безопасный вариант
    if number, ok := data.(int); ok {
        fmt.Println("Число:", number)
    } else {
        fmt.Println("Это не число")
    }
}
```

### 15. **Метод set и правила реализации интерфейсов**
**Method set** - это набор методов, доступных у значения определённого типа.

Понимание этого метода критически важно для работы с интерфейсами в Go

**Для типа `T` (значение)**
- Method set содержит методы с получателем (t T)
- НЕ содержит методы с получателем (t \*T)

Для типа `*T` (указатель)
- Method set содержит методы с получателем (t T) и (t \*T)

```go
type Document struct {
    title string
    content string
}

// Метод с получателем по значению
func (d Document) GetTitle() string {
    return d.title
}

// Метод с получателем по указателю
func (d *Document) SetTitle(title string) {
    d.title = title
}

// Интерфейс только с методом по значению
type Reader interface {
    GetTitle() string
}

// Интерфейс с методами по значению и указателю
type Writer interface {
    GetTitle() string
    SetTitle(string)
}

func main() {
    doc := Document{title: "Заголовок", content: "Содержимое"}
    docPtr := &Document{title: "Заголовок", content: "Содержимое"}
    
    // Document (значение) удовлетворяет Reader
    var r1 Reader = doc    // OK
    var r2 Reader = docPtr // OK (указатель содержит методы значения)
    
    // Writer требует SetTitle (указательный метод)
    var w1 Writer = doc    // Ошибка компиляции: method set T не включает SetTitle
    var w2 Writer = docPtr // OK
}
```

### 16. **Автоматическое разыменование**
Go автоматически преобразует между указателями и значениями при вызове методов

```go
type Counter struct {
    value int
}
func (c *Counter) Increment() {
    c.value++
}
func (c Counter) GetValue() int {
    return c.value
}
func main() {
    // Значение
    counter1 := Counter{value: 0}
    counter1.Increment() // Компилятор автоматически подставит (&counter1).Increment()
    
    // Указатель  
    counter2 := &Counter{value: 0}
    value := counter2.GetValue() // Компилятор автоматически подставит (*counter2).GetValue()
    
    fmt.Println(counter1.GetValue(), value)
}
```

### 17. **Композиция интерфейсов**
Интерфейсы можно комбинировать для создания более сложных контрактов, прямо как у структур

```go
type Reader interface {
    Read([]byte) (int, error)
}

type Writer interface {
    Write([]byte) (int, error)
}

type Closer interface {
    Close() error
}

// Композиция интерфейсов
type ReadWriter interface {
    Reader
    Writer
}

type ReadWriteCloser interface {
    Reader
    Writer
    Closer
}
```
