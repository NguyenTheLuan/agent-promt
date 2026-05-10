# 🅰️ ANGULAR CODING RULES

---

## API & DI

| ✅ Đúng                       | ❌ Sai                             |
| ----------------------------- | ---------------------------------- |
| `readonly x = input<T>()`     | `@Input() x: T`                    |
| `readonly x = output<T>()`    | `@Output() x = new EventEmitter()` |
| `private x = inject(Token)`   | `constructor(private x: Service)`  |
| `host: { '[attr]': 'val()' }` | `@HostBinding('attr') x`           |
| `host: { '(event)': 'fn()' }` | `@HostListener('event') fn()`      |

---

## Change Detection

```typescript
@Component({ changeDetection: ChangeDetectionStrategy.OnPush }) // BẮT BUỘC
```

---

## Standalone

```
✅ Mặc định Angular 19 — KHÔNG cần flag
✅ KHÔNG NgModule
```

---

## Template

| ✅ Đúng                                 | ❌ Sai                              |
| --------------------------------------- | ----------------------------------- |
| `@if (cond()) { }`                      | `*ngIf="cond"`                      |
| `@for (item of items(); track item.id)` | `*ngFor="let item of items"`        |
| `[class.active]="bool()"`               | `ngClass="..."`                     |
| `[style.color]="val()"`                 | `ngStyle="..."`                     |
| `computed()` trong class                | `{{ methodCall() }}` trong template |

---

## State — Signals

```
✅ signal() cho internal state
✅ computed() cho derived state (pure, memoized)
✅ .asReadonly() cho public API
✅ .update() / .set() — không mutate trực tiếp
✅ [(property)] 2-way binding với writable signals
❌ BehaviorSubject / Subject cho component state
```

---

## Styles

```typescript
// Global CSS injection — khi component dùng attribute selector
@Component({
  template: "",
  styles: [
    `
      @import "...";
    `,
  ],
})
class Styles {}

@Directive({ selector: "[tuiButton]" })
export class TuiButton {
  protected readonly nothing = tuiWithStyles(Styles);
}
```

Import: `import {tuiWithStyles} from '@my-ui/cdk/utils/miscellaneous'`

---

## hostDirectives

```typescript
@Directive({
    selector: '[tuiButton]',
    hostDirectives: [TuiWithAppearance, TuiWithIcons],
})
export class TuiButton { ... }
```

---

## File Structure

```
feature-name/
├── feature-name.component.ts
├── feature-name.component.html
├── feature-name.component.less
├── feature-name.style.less    (optional — tuiWithStyles)
├── index.ts
└── test/feature-name.spec.ts
```
