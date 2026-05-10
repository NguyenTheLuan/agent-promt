# 📐 TYPESCRIPT RULES

| Rule             | ✅ Đúng                         | ❌ Sai                              |
| ---------------- | ------------------------------- | ----------------------------------- |
| strict           | `tsconfig: strict: true`        | strict: false                       |
| readonly         | `readonly size = input<T>()`    | `size = input<T>()`                 |
| unknown > any    | `fn(v: unknown)`                | `fn(v: any)`                        |
| satisfies > as   | `x satisfies T`                 | `x as T`                            |
| inference        | `const items = signal<T>([])`   | `const items: WritableSignal<T>...` |
| explicit returns | `public fn(): T { return ... }` | `public fn() { return ... }`        |
| const > let      | `const x = 5`                   | `let x = 5` (không reassign)        |
| arrow callbacks  | `items.map(item => ...)`        | `items.map(function(item) {...})`   |
