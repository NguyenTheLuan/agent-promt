# 🏗️ ARCHITECTURE RULES — Design Patterns & OOP

---

## COMPONENT DESIGN

### Rule: Ưu tiên directive > component

- Attribute selector `[tuiButton]` **luôn ưu tiên** hơn `tui-button`
- Directive khi behavior áp dụng lên nhiều element
- Component khi cần template riêng

### Rule: Phân tầng dependency

```
cdk/  → KHÔNG import từ package khác
core/ → CHỈ import từ cdk/
kit/  → Import từ cdk/ + core/
```

### Rule: Container + Presenter

- **Container**: quản lý state, logic
- **Presenter**: render UI, emit events, KHÔNG business logic

### Rule: Component size

- .ts ≤ 200 dòng, .html ≤ 100 dòng → vượt thì tách

### Rule: hostDirectives > extends

- Compose behavior qua `hostDirectives`, KHÔNG `extends`

### Rule: DI token cho global config

- Default options → `InjectionToken<T>` + helper provider function

### Rule: CSS injection

- Attribute selector → `tuiWithStyles(Styles)`

---

## SOLID PRINCIPLES

| Principle                 | Áp dụng                                                              |
| ------------------------- | -------------------------------------------------------------------- |
| **S**ingle Responsibility | 1 component = 1 việc. Tách logic ra service/directive                |
| **O**pen/Closed           | Mở cho extension: `input()`, `hostDirectives`. Đóng cho modification |
| **L**iskov Substitution   | Directive có thể thay thế nhau không gây lỗi                         |
| **I**nterface Segregation | Token nhỏ, không 1 token khổng lồ                                    |
| **D**ependency Inversion  | Phụ thuộc vào abstraction: `inject(Token)`, không `new Service()`    |

---

## DESIGN PATTERNS

### Pattern 1: Decorator (Attribute Directive)

```typescript
// Button behavior decorate lên native element
@Directive({ selector: '[tuiButton]' })
export class TuiButton { ... }
```

### Pattern 2: Composite (hostDirectives)

```typescript
@Directive({
    selector: '[tuiButton]',
    hostDirectives: [TuiWithAppearance, TuiWithIcons], // Compose behaviors
})
```

### Pattern 3: Strategy (Token Providers)

```typescript
// Cấu hình khác nhau cho cùng 1 component
{ provide: TUI_BUTTON_OPTIONS, useValue: { size: 'l' } }
```

### Pattern 4: Observer (Signals)

```typescript
// Signals thay thế Observable cho state
readonly count = computed(() => this.items().length);
```

### Pattern 5: Factory (InjectionToken factory)

```typescript
export const TUI_BUTTON_OPTIONS = new InjectionToken("...", {
  factory: () => ({ size: "m", appearance: "primary" }),
});
```

### Pattern 6: Singleton (Service)

```typescript
@Injectable({ providedIn: 'root' })
export class TuiNotificationService { ... }
```

### Pattern 7: Template Method (Base Directive)

```typescript
@Directive()
export abstract class TuiAbstractDropdown {
    protected abstract getContent(): TemplateRef<unknown>;
    open(): void { ... } // Template method
}
```

### Pattern 8: Adapter (ControlValueAccessor)

```typescript
// Adapt Angular Forms vào component
export class TuiInput implements ControlValueAccessor { ... }
```

### Pattern 9: Facade (Service layer)

```typescript
// Service ẩn complexity bên trong
export class TuiDialogService {
  open(component: Type<unknown>): void {
    /* portal + overlay logic */
  }
}
```
