# 🛠️ SKILLS — Kỹ năng coding

> Code mẫu, patterns, tips.

---

## 1. Directive-based component

```typescript
@Component({
  template: "",
  styles: [
    `
      @import "...";
    `,
  ],
})
class Styles {}

@Directive({
  selector: "button[tuiButton], a[tuiButton], [tuiButton]",
  host: {
    "[attr.data-size]": "size()",
    "[attr.data-appearance]": "appearance()",
    "(click)": "onClick()",
  },
})
export class TuiButton {
  readonly size = input<"m" | "l">("m");
  protected readonly nothing = tuiWithStyles(Styles);
}
```

**Tips:** Selector format `element[tuiName], [tuiName]`, `hostDirectives`, `tuiWithStyles`.

---

## 2. Element component với signal state

```typescript
@Component({
  selector: "tui-expand",
  templateUrl: "./expand.component.html",
  changeDetection: ChangeDetectionStrategy.OnPush,
  host: { "[class._expanded]": "expanded()" },
})
export class TuiExpand {
  readonly expanded = input(false);
  readonly expandedChange = output<boolean>();
  private readonly animating = signal(false);
}
```

```html
@if (expanded() || animating()) {
<div class="t-wrapper"><ng-content /></div>
}
```

**Tips:** `computed()` cho derived state, `.asReadonly()` expose API.

---

## 3. tuiWithStyles — inject CSS global

```typescript
export function tuiWithStyles(component: Type<unknown>): undefined {
  const map = inject(MAP);
  const env = inject(EnvironmentInjector);
  if (!map.has(component)) {
    map.set(
      component,
      createComponent(component, { environmentInjector: env })
    );
  }
  return;
}
```

Map cache → 1 component ảo duy nhất → tất cả instances chia sẻ.

---

## 4. Service với signal state

```typescript
@Injectable({ providedIn: "root" })
export class TuiNotificationService {
  private readonly _items = signal<T[]>([]);
  readonly items = this._items.asReadonly();
  readonly count = computed(() => this._items().length);

  show(item: T): void {
    this._items.update((list) => [...list, item]);
  }
  clear(): void {
    this._items.set([]);
  }
}
```

`_items` private → `items` public `.asReadonly()`. Không BehaviorSubject.

---

## 5. Pure pipe

```typescript
@Pipe({ name: "tuiFilter" })
export class TuiFilterPipe implements PipeTransform {
  public transform<T>(
    items: readonly T[],
    matcher: (item: T, ...args: unknown[]) => boolean,
    ...args: unknown[]
  ): T[] {
    return items.filter((item) => matcher(item, ...args));
  }
}
```

---

## 6. Injection token

```typescript
export const TUI_BUTTON_OPTIONS = new InjectionToken<TuiButtonOptions>(
  ngDevMode ? "TUI_BUTTON_OPTIONS" : "",
  { factory: () => ({ size: "m", appearance: "primary" }) }
);
```

`TUI_` + UPPER_SNAKE, `ngDevMode ? 'NAME' : ''`.

---

## 7. hostDirective (compose behavior)

```typescript
@Directive({
  selector: "[tuiAppearance]",
  host: { "[class._hover]": "hover()", "[class._active]": "active()" },
})
export class TuiAppearance {
  readonly hover = signal(false);
  readonly active = signal(false);
}

@Directive({ selector: "[tuiButton]", hostDirectives: [TuiAppearance] })
export class TuiButton {
  private readonly appearance = inject(TuiAppearance);
}
```

---

## 8. Unit test với TuiPageObject

```typescript
describe("TuiButton", () => {
  @Component({
    standalone: true,
    imports: [TuiButton],
    template: `<button
      tuiButton
      automation-id="tui-button__native"
      (click)="onClick()"
    >
      Click
    </button>`,
  })
  class Test {
    clicked = false;
    onClick() {
      this.clicked = true;
    }
  }

  let fixture: ComponentFixture<Test>;
  let pageObject: TuiPageObject<Test>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [Test],
    }).compileComponents();
    fixture = TestBed.createComponent(Test);
    pageObject = new TuiPageObject(fixture);
    fixture.detectChanges();
  });

  it("click", () => {
    pageObject.getByAutomationId("tui-button__native")?.nativeElement.click();
    fixture.detectChanges();
    expect(fixture.componentInstance.clicked).toBeTrue();
  });
});
```

---

## 9. Barrel export

```typescript
export * from "./constants";
export * from "./tokens";
export * from "./directives";
export * from "./pipes";
export * from "./button.directive";
```

---

## 10. Form control

```typescript
@Directive({
  selector: "input[tuiInput]",
  providers: [
    { provide: NG_VALUE_ACCESSOR, multi: true, useExisting: TuiInput },
  ],
  hostDirectives: [TuiControl],
})
export class TuiInput implements ControlValueAccessor {
  private readonly ctrl = inject(TuiControl);
  writeValue(v: string) {
    this.ctrl.value.set(v);
  }
  registerOnChange(fn: Function) {
    this.ctrl.onChange = fn;
  }
  registerOnTouched(fn: Function) {
    this.ctrl.onTouched = fn;
  }
  setDisabledState(d: boolean) {
    this.ctrl.disabled.set(d);
  }
}
```

---

## 11. CSS Grid animation

```less
:host {
  display: block;
  overflow: hidden;
  transition: grid-template-rows var(--tui-duration, 300ms);
  grid-template-rows: 0fr;
  &._expanded {
    grid-template-rows: 1fr;
  }
}
.t-wrapper {
  overflow: hidden;
}
```

GPU accelerated, không cần JS.

---

## 12. Dropdown/overlay

```typescript
@Directive({
  selector: "[tuiDropdown]",
  hostDirectives: [TuiDropdownDirective],
})
export class TuiDropdown {
  readonly dropdown = input<TuiDropdownContent | null>(null, {
    alias: "tuiDropdown",
  });
  readonly open = input(false, { alias: "tuiDropdownOpen" });
}
```
