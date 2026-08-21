# Функции

### Структуры и методы. Интерфейсы

Начнем с того, что функции в Go - это объекты первого класса
	А это значит, что с функцией можно обращаться как с любой переменной

```go
func main() {
	p := prints // Не указываем аргумент функции
	fmt.Println(p) // Увидим адрес
}

func prints(str string) {
	fmt.Println(str)
}
```

1. `Анонимные функции` 
	Нужны, когда мы хотим объявить функцию в конкретном месте и ее жизнь отдельной сущностью нам не требуется

```go
f := func(a int, b int) int {
	return a + b
}

result := f(2, 3)
fmt.Println(result) // 5
```
Еще мы можем вызвать такую функцию сразу и присвоить ее результат переменной:

```go
result := func(x int) int {
	return x * x
}(4)

fmt.Println(result) // 16
```

2. `Замыкания`
	**Анонимная** (зачастую) или всё-таки **именованная функция**, которая использует переменные из **внешней области видимости** (то есть те, которые не находятся внутри самого тела функции).

	В случае замыкания, функция **не копирует значение** переменной, а захватывает ее

```go
func counter() func() int {
	i := 0
	
	return func() int {
		i++
		return i
	}
}
```

Что здесь происходит? Мы объявили анонимную функцию и вернули из нее counter; так, что анонимная функция видит переменную, объявленную в одной области видимости.

```go
c := counter()

fmt.Println(c()) // 1
fmt.Println(c()) // 2
fmt.Println(c()) // 3


d := counter()

fmt.Println(d()) // 1
```

Таким образом, здесь мы можем взаимодействовать через замыкание с данными, которые уже находятся вне нашего внимания!

Все новые замыкания будут разными, работать независимо друг от друга!

Ещё важно уточнить, что замыкания захватывают не только переменные, но и параметры функции:

```go
func main() {
	double := multiplier(2)
	triple := multiplier(3)

	fmt.Println(double(5)) // 10
	fmt.Println(triple(5)) // 15
}

func multiplier(factor int) func(int) int {
	return func(x int) int {
	return x * factor
	}
}
```

3. `Именованный возврат`
	Не обязательно создавать переменные, которые хочешь вернуть, в теле функции. Это можно сделать прямо в сигнатуре, и тогда переменные уже будут созданы на этапе компиляции. Сравним:

```go
func calc(first int, second int) (int, int) {
	sum := first + second
	product := first * second

	return sum, product
}


func calc(first int, second int) (sum int, product int) {
	sum = first + second
	product = first * second
	
	return
}
```

4. `Структуры: группировка данных и никакого ООП`
	**Структуры** - составной тип данных, который позволяет объединять связанные переменные в одну единицу данных. 
	
	Структура состоит из именованных полей одного или разных типов и помогает организовывать связанные данные в одном контейнере

```go
type Emplotee struct {
	Name      string
	Age       int
	Position  string
	Salary    int
}
```

Создавать структуры можно тоже разными способами. Например, через присваиванием переменным с чётким именованием 

(если опустить поле - оно заполнится `zero value` или же `nil`)

```go
emp1 := Employee{
	Name: "Анна Иванова",
	Age: 28,
	Position: "Разработчик",
	Salary: 120_000,
}

fmt.Println(emp1.Salary) // 120000
```

Или можно создать структуру в более кратком виде - без уточнения имени полей. Но поля должны идти по порядку:

```go
emp2 := Employee{
	"Петр Петров", 
	32, 
	"Тестировщик", 
	100_000
	}
```

И еще можно создать пустую структуру (все поля заполнятся `zero value`), а затем заполнять поля по мере надобности:

```go
var emp3 Employee

emp3.Name = "Мария Сидорова"
emp3.Age = 25
emp3.Position = "Аналитик"
emp3.Salary = 110_000

fmt.Println(emp3.Name) // Мария сидорова
```

Именно так - через точку - можно обращаться к полям для того, чтобы прочесть или записать данные.

5. `Важные особенности структур`
	Структуры, как и остальные типы, можно **передавать** в функции и возвращать их оттуда.

	Однако, изменения, выполненные над структурой, окажутся **не видны вне** тела функции. Так что, необходимо **явно возвращать ее**

```go
func main() {
	emp1 := Employee{
		Name: "Кристина",
		Salary: 100_000,
	}

	increaseSalary(&emp1, 5000)

	fmt.Println(emp1)
}

type Employee struct {
	Name string
	Salary int
}

// Эта функция НЕ изменит оригинальную структуру!
func tryToIncreaseSalary(emp Employee, amount int) {
	emp.Salary += amount // Изменяем только копию
}

func increaseSalary(emp *Employee, amount int) {
	emp.Salary += amount // Изменяем оригиналь через указатель
}
```

6. `Вложение структур друг в друга`
```go
type Address struct {
	Street string
	City string
	Zip string
}

type Person struct {
	Name string
	Age int
	Address Address // Вложенная структура
}
```

Таким образом можно составлять иерархию структур, данных, характеристик

Частным случаем композиции является **встраивание (embedding)**, когда поле не имеет имени. В таком случае **все поля и методы встроенной структуры автоматически подтягиваются** к родительской структуре

```go
type Contact struct {
	Email string
	Phone string
}

type Employee struct {
	Name string
	Position string
	Contact // Встроенная структура (анонимное поле)
}

func main() {
	emp := Employee{
	Name: "Анна Иванова",
	Position: "Разработчик",
	Contact: Contact{
		Email: "anna@company.com",
		Phone: "+7-123-456-7890",
	},
}

fmt.Println(emp.Email) // anna@company.com
fmt.Println(emp.Phone) // +7-123-456-7890

fmt.Println(emp.Contact.Email) // также
fmt.Println(emp.Contact.Phone) // работает
}
```

**Про конфликт имён:**

- Имена полей, объявленные в самой структуре, имеют приоритет над именами полей встраиваемых структур;

- Если одинаковое имя встречается только у встроенных структур, выбирается поле/метод, ближней к внешней структура

- Если одинаковые имена приходят из нескольких встраиваний по одной глубине, обращение не по полному пути приведет к ошибке компиляции

7. **Кастомные типы**
```go
type Stringz []string
type Address string
type FullAddress Address
type EveryDay int
```

8. `Методы структур`
	`Метод` - это функция, которая принадлежит конкретному типу и реализует логику, связанную только с ним
```go
type Status string // Объявим кастомный тип

// Добавим к типу поведение - возвращать представление типа в виде базового типа

func (s Status) String() string {
	return string(s) // Приведение кастомного типа к базовому
}
```

Или вот пример со структурой:

```go
// Добавим к типу поведение - возвращать представление типа в виде базового типа

type Rectangle struct {
	Width float64
	Height float64
}

// Метод с получателем по значению
func (r Rectangle) Area() float64 {
	return r.Width * r.Height
}

// Метод с получателем по указателю
func (r *Rectangle) Scale(factor float64) {
	r.Width *= factor
	r.Height *= factor
}

func main() {
	rect := Rectangle{Width: 10, Height: 5}
	
	fmt.Println("Площадь:", rect.Area()) // 50
	fmt.Println(rect) // Структура до вызова Scale // {10 5}
	
	rect.Scale(2)
	
	fmt.Println("После масштабирования:", rect) // {20 10}
	fmt.Println("Новая площадь:", rect.Area()) // 200
}
```

9. `Публичные vs приватные` методы структур
	**Публичные (экспортируемые)** - имя метода начинается с заглавной буквы

	**Приватные (неэкспортируемые)** - имя начинается с маленькой буквы

Практическая польза

- Экспортируемые методы формируют публичный API твоего типа - то, как другие пакеты могут взаимодействовать с твоей структурой;

- Неэкспортируемые методы инкапсулируют внутреннюю логику, которую не нужно выставлять наружу

- Это позволяет изменять внутреннюю реализацию, не ломая код, который использует экспортируемые методы структуры.

10. `Интерфейсы: определение поведения`
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

11. `Использование интерфейсов`
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

12. `Пустой интерфейс` 
	`interface{}` не содержит ни одного метода в своей определении

	Поскольку любой тип в Go может иметь ноль методов, **любой тип удовлетворяет пустому интерфейсу**. Это делает `interface{}/any` универсальным типом

13. `Type Assertion`
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

14. `Type Switch`
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

15. `Метод set и правила реализации интерфейсов`
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

16. `Автоматическое разыменование`
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

17. `Композиция интерфейсов`
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

