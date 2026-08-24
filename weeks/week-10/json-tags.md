# Быстро повторить: JSON, `Marshal`, `Unmarshal` и struct tags

[← К полной неделе](./README.md) · [↑ К содержанию курса](../README.md)

## Сериализация

Преобразование Go-значения в JSON:

```go
b, err := json.Marshal(value)
```

Результат — байты.

## Десериализация

JSON обратно в Go-структуру:

```go
err := json.Unmarshal(b, &value)
```

Обрати внимание на указатель: `Unmarshal` должен записать результат в переданное значение.

## Struct tags

```go
type User struct {
    Name     string `json:"name"`
    Age      int    `json:"age,omitempty"`
    Password string `json:"-"`
}
```

В конспекте:

- `json:"name"` задаёт имя поля;
- `omitempty` позволяет пропустить пустое/zero value;
- `json:"-"` исключает поле из JSON.

`encoding/json` работает с экспортируемыми полями структуры. Неэкспортируемое поле не сериализуется.

## JSON и YAML

В конспекте JSON предлагается для строгого машинного формата, YAML — как более удобный для редактируемых человеком конфигураций.

## Что проговорить перед собеседованием

- Что такое serialization / deserialization?
- Что возвращает `json.Marshal`?
- Почему в `json.Unmarshal` передаётся указатель?
- Что делает `omitempty`?
- Что означает `json:"-"`?
- Почему неэкспортируемое поле структуры не попадает в JSON?
