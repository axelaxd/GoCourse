# Go Interview Notes

Навигация по конспектам, переразложенная под техническое собеседование.

## Приоритет

- 🔴 — повторить обязательно перед собеседованием.
- 🟠 — уверенно объяснять концептуально.
- 🟡 — достаточно быстро освежить.

## Темы

- 🟠 [Strings, bytes и runes](./01-strings-runes/README.md)
- 🔴 [Arrays и slices](./02-slices-arrays/README.md)
- 🔴 [Maps](./03-maps/README.md)
- 🔴 [Pointers и value semantics](./04-pointers/README.md)
- 🔴 [Structs, methods и embedding](./05-structs-methods/README.md)
- 🔴 [Interfaces и method set](./06-interfaces/README.md)
- 🔴 [Errors / defer / panic / recover](./07-errors-defer-panic/README.md)
- 🟠 [Generics](./08-generics/README.md)
- 🔴 [Goroutines / channels / select / race](./09-concurrency/README.md)
- 🔴 [Context / synchronization / graceful shutdown](./10-context-sync/README.md)
- 🟠 [Packages / modules / project / tooling](./11-project-modules-tooling/README.md)
- 🟡 [I/O / JSON / YAML / config](./12-io-json-config/README.md)
- 🔴 [HTTP / REST / middleware](./13-http-rest/README.md)
- 🟠 [Testing / mocks / httptest / fuzzing](./14-testing/README.md)

## Маршрут на последний вечер

1. Slices → Maps → Pointers.
2. Structs → Interfaces → Errors.
3. Goroutines → Channels → Select → Race.
4. Context → Mutex/WaitGroup → Graceful shutdown.
5. HTTP → Testing.
6. Если осталось время: Generics → Modules/Tooling → JSON/I/O.

## Как повторять

В начале каждого README есть блок **«Что нужно уметь объяснить вслух»**. Сначала попробуй ответить на эти вопросы без подсказки, затем читай выдержку из конспекта.