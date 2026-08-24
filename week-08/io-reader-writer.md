# Быстро повторить: `io.Reader`, `io.Writer` и буферизация

[← К полной неделе](./README.md) · [↑ К содержанию курса](../README.md)

## Основные интерфейсы `io`

В конспекте выделены маленькие интерфейсы стандартной библиотеки:

```go
io.Reader
```

Читающий тип предоставляет:

```go
Read(p []byte) (int, error)
```

Записывающий:

```go
io.Writer
```

с методом:

```go
Write(p []byte) (int, error)
```

Освобождение ресурса:

```go
io.Closer
```

с методом:

```go
Close() error
```

Есть составные интерфейсы: `io.ReadWriter`, `io.ReadCloser`, `io.WriteCloser`.

## Работа с файлом

```go
file, err := os.Open("input.txt")
if err != nil {
    return err
}
defer file.Close()
```

`os.Open` открывает файл для чтения.

Для чтения целого небольшого файла есть:

```go
data, err := os.ReadFile("input.txt")
```

Но в конспекте отмечено, что для больших файлов лучше работать потоково.

## `bufio`

Для последовательного текстового чтения:

```go
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println(scanner.Text())
}
```

Для более управляемого чтения и длинных строк можно использовать:

```go
reader := bufio.NewReader(file)
```

## EOF

`io.EOF` сигнализирует о достижении конца потока.

## Что проговорить перед собеседованием

- Что такое `io.Reader`?
- Почему `os.File` можно передавать туда, где нужен `io.Reader`?
- Чем `os.ReadFile` отличается от потокового чтения?
- Для чего `bufio.Scanner`?
- Что означает `io.EOF`?
- Почему открытый файл нужно закрывать?
