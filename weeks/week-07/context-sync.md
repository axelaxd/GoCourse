# Быстро повторить: `context`, `Mutex`, `WaitGroup`

[← К полной неделе](./README.md) · [↑ К содержанию курса](../README.md)

## `context.Context`

В конспекте у контекста три основные задачи:

- передача сигнала отмены;
- дедлайны и тайм-ауты;
- передача метаданных между слоями.

Базовый контекст:

```go
ctx := context.Background()
```

Отмена вручную:

```go
ctx, cancel := context.WithCancel(ctx)
defer cancel()
```

Тайм-аут:

```go
ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
defer cancel()
```

Ожидание отмены:

```go
select {
case <-ctx.Done():
    return ctx.Err()
}
```

Если отменяется родительский context, отмена распространяется на дочерние.

В `context` не следует складывать параметры бизнес-логики; в конспекте `WithValue` рассматривается для метаданных.

## `Mutex`

`sync.Mutex` защищает участок кода/данные от одновременного конкурентного доступа:

```go
mu.Lock()
defer mu.Unlock()

m[key] = value
```

В структурах mutex обычно располагают рядом с данными, которые он защищает.

`RWMutex` дополнительно имеет `RLock` / `RUnlock` для операций чтения.

## `WaitGroup`

`WaitGroup` позволяет дождаться завершения группы горутин.

Смысл связки:

```go
wg.Add(1)

go func() {
    defer wg.Done()
    // работа
}()

wg.Wait()
```

В конспекте также отмечено: когда нужна удобная обработка ошибок от группы горутин, стоит смотреть в сторону `errgroup`.

## Что проговорить перед собеседованием

- Зачем нужен context?
- `WithCancel` vs `WithTimeout` vs `WithDeadline`?
- Что такое `ctx.Done()`?
- Как распространяется отмена?
- Что не стоит передавать через `WithValue`?
- Что защищает `Mutex`?
- Зачем `defer mu.Unlock()`?
- Для чего `WaitGroup`?
