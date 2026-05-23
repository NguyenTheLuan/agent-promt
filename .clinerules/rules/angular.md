# 🅰️ ANGULAR CODING RULES (v20+)

---

## API & DI

| ✅ Correct                    | ❌ Wrong                           |
| ----------------------------- | ---------------------------------- |
| `readonly x = input<T>()`     | `@Input() x: T`                    |
| `readonly x = output<T>()`    | `@Output() x = new EventEmitter()` |
| `private x = inject(Token)`   | `constructor(private x: Service)`  |
| `host: { '[attr]': 'val()' }` | `@HostBinding('attr') x`           |
| `host: { '(event)': 'fn()' }` | `@HostListener('event') fn()`      |

**Rule:** `input()`, `output()`, `inject()` ≥ Angular 17+. No decorators.

### Provider Scopes

```typescript
// Root (singleton)
@Injectable({ providedIn: 'root' })
export class Auth {}

// Component level (new instance per component)
@Component({
  providers: [EditorState],
})
export class Editor { private state = inject(EditorState); }

// Route level (shared within route tree)
{ path: 'admin', providers: [Admin], children: [...] }

// Optional injection
private analytics = inject(Analytics, { optional: true });

// Self / SkipSelf / Host
private local = inject(Local, { self: true });
private parent = inject(Parent, { skipSelf: true });
private host = inject(Host, { host: true });
```

### Injection Tokens

```typescript
export const API_URL = new InjectionToken<string>('API_URL');

// Self-providing token
export const WINDOW = new InjectionToken<Window>('Window', {
  providedIn: 'root',
  factory: () => window,
});

// Provide
{ provide: API_URL, useValue: 'https://api.example.com' }
{ provide: Logger, useClass: environment.production ? ProductionLogger : ConsoleLogger }
{ provide: AbstractLogger, useExisting: ConsoleLogger }

// Multi providers (collect array)
{ provide: VALIDATORS, useClass: RequiredValidator, multi: true }
{ provide: VALIDATORS, useClass: EmailValidator, multi: true }
```

### Provider Types

```typescript
{ provide: Token, useClass: Impl }           // Class replacement
{ provide: Token, useValue: { ... } }         // Static value
{ provide: Token, useFactory: (dep) => new X(dep), deps: [Dep] }  // Factory
{ provide: Token, useExisting: OtherToken }   // Alias
```

---

## Change Detection

```typescript
@Component({ changeDetection: ChangeDetectionStrategy.OnPush }) // REQUIRED
```

**Rule:** Always `OnPush`. No `Default` change detection.

---

## Standalone

```
✅ Default in Angular 19+ — NO flag needed
✅ NO NgModule
```

**Rule:** Every component, directive, and pipe is standalone. NgModules are legacy.

---

## Template

| ✅ Correct                              | ❌ Wrong                            |
| --------------------------------------- | ----------------------------------- |
| `@if (cond()) { }`                      | `*ngIf="cond"`                      |
| `@for (item of items(); track item.id)` | `*ngFor="let item of items"`        |
| `[class.active]="bool()"`               | `ngClass="..."`                     |
| `[style.color]="val()"`                 | `ngStyle="..."`                     |
| `computed()` in class                   | `{{ methodCall() }}` in template    |

```html
@if (isLoading()) { ... }
@else if (error()) { ... }
@else { ... }

@for (item of items(); track item.id) { ... }
@empty { <p>No items</p> }

@switch (status()) {
  @case ('pending') { ... }
  @case ('active') { ... }
  @default { ... }
}
```

**Rule:** Use built-in control flow (`@if`, `@for`, `@switch`). No structural directives.

---

## State — Signals

### Core APIs

```typescript
// Writable
const count = signal(0);
count.set(5);
count.update(c => c + 1);

// Derived (auto-updates)
const doubled = computed(() => count() * 2);

// Dependent with reset
const selected = linkedSignal(() => options()[0]);
const selectedItem = linkedSignal({
  source: () => items(),
  computation: (newItems, previous) => {
    if (previous?.value && newItems.some(i => i.id === previous.value.id)) {
      return previous.value;
    }
    return newItems[0] ?? null;
  },
});

// Side effects
effect(() => {
  console.log('Changed:', this.query());
});

// Cleanup
effect((onCleanup) => {
  const timer = setInterval(fn, 1000);
  onCleanup(() => clearInterval(timer));
});

// Untracked reads (don't depend)
const result = computed(() => a() + untracked(() => b()));

// Custom equality
const user = signal({ id: 1, name: 'Alice' }, { equal: (a, b) => a.id === b.id });
```

### Component State Pattern

```typescript
@Component({...})
export class TodoList {
  todos = signal<Todo[]>([]);
  newTodo = signal('');
  filter = signal<'all' | 'active' | 'done'>('all');

  canAdd = computed(() => this.newTodo().trim().length > 0);
  filteredTodos = computed(() => { /* filter logic */ });
  remaining = computed(() => this.todos().filter(t => !t.done).length);

  addTodo() {
    this.todos.update(todos => [...todos, { id: crypto.randomUUID(), text: this.newTodo(), done: false }]);
    this.newTodo.set('');
  }
}
```

### Service State Pattern

```typescript
@Injectable({ providedIn: 'root' })
export class Auth {
  private _user = signal<User | null>(null);
  private _loading = signal(false);

  readonly user = this._user.asReadonly();
  readonly loading = this._loading.asReadonly();
  readonly isAuthenticated = computed(() => this._user() !== null);
}
```

### RxJS Interop

```typescript
// Observable → Signal
counter = toSignal(interval(1000), { initialValue: 0 });
users = toSignal(this.http.get<User[]>('/api/users'));

// Signal → Observable
results = toSignal(
  toObservable(this.query).pipe(
    debounceTime(300),
    switchMap(q => this.http.get(`/api/search?q=${q}`))
  ),
  { initialValue: [] }
);
```

**Rule:** Signals everywhere. No RxJS for component state.

---

## Styles

```typescript
@Component({
  selector: "app-card",
  styles: `
    :host {
      display: block;
      border-radius: 8px;
    }
  `,
})
export class CardComponent {}
```

**Rule:** Styles scoped via `:host` or `ViewEncapsulation`. Global styles in `styles.scss` only.

---

## hostDirectives

```typescript
@Directive({
  selector: "[appHighlight]",
  hostDirectives: [
    { directive: TooltipDirective, inputs: ["tooltip"] },
    { directive: Hoverable, outputs: ["hoverChange"] },
  ],
})
export class HighlightDirective {}
```

**Rule:** Compose behavior via `hostDirectives`. Avoid `extends`.

### Directive Composition API

```typescript
@Component({
  selector: 'app-material-button',
  hostDirectives: [
    Ripple,
    { directive: Elevation, inputs: ['elevation'] },
    { directive: Disableable, inputs: ['disabled'] },
  ],
  template: `<ng-content />`,
})
export class MaterialButton {}
```

---

## Component Selectors

```
✅ Attribute: [appButton], [appDropdown]
✅ Element: app-header, app-footer (when template needed)
✅ Prefix: app- (application), ui- (shared library)
❌ Generic names: button, card, modal (collision risk)
```

---

## File Structure

```
feature-name/
├── feature-name.component.ts
├── feature-name.component.html
├── feature-name.component.scss
├── index.ts
└── feature-name.component.spec.ts
```

**Rule:** One file per type. Tests next to source. Barrel exports via `index.ts`.

---

## HTTP & Data Fetching

### httpResource() — Signal-Based HTTP (v20+)

```typescript
@Component({
  template: `
    @if (userResource.isLoading()) { <p>Loading...</p> }
    @else if (userResource.error()) { <p>Error</p> <button (click)="userResource.reload()">Retry</button> }
    @else if (userResource.hasValue()) { <h1>{{ userResource.value().name }}</h1> }
  `,
})
export class UserProfile {
  userId = signal('123');
  userResource = httpResource<User>(() => `/api/users/${this.userId()}`);

  // With options
  usersResource = httpResource<User[]>(() => ({
    url: '/api/users',
    headers: { 'Authorization': `Bearer ${this.token()}` },
    params: { include: 'profile' },
  }), { defaultValue: [] });

  // Skip when undefined
  searchResource = httpResource<Result[]>(() => {
    return this.query() ? `/api/search?q=${this.query()}` : undefined;
  });

  // State: .value(), .hasValue(), .error(), .isLoading(), .status(), .reload(), .set(), .update()
}
```

### resource() — Generic Async

```typescript
searchResource = resource({
  params: () => ({ q: this.query() }),
  loader: async ({ params, abortSignal }) => {
    if (!params.q) return [];
    const res = await fetch(`/api/search?q=${params.q}`, { signal: abortSignal });
    return res.json();
  },
});
```

### HttpClient — Traditional

```typescript
private http = inject(HttpClient);
this.http.get<User[]>('/api/users');
this.http.post<User>('/api/users', data);
this.http.put<User>(`/api/users/${id}`, data);
this.http.patch<User>(`/api/users/${id}`, changes);
this.http.delete<void>(`/api/users/${id}`);
```

### Functional Interceptors

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = inject(Auth).token();
  if (token) { req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } }); }
  return next(req);
};

// app.config.ts
provideHttpClient(withInterceptors([authInterceptor, errorInterceptor, loggingInterceptor]))
```

### Loading States Pattern

```html
@switch (resource.status()) {
  @case ('idle') { <p>Enter a search term</p> }
  @case ('loading') { <app-spinner /> }
  @case ('reloading') { <app-data [data]="resource.value()" /> <app-spinner size="small" /> }
  @case ('resolved') { <app-data [data]="resource.value()" /> }
  @case ('error') { <app-error [error]="resource.error()" (retry)="resource.reload()" /> }
}
```

---

## Routing

### Basic Setup

```typescript
// app.routes.ts
export const routes: Routes = [
  { path: '', redirectTo: '/home', pathMatch: 'full' },
  { path: 'home', component: Home },
  { path: 'admin', loadChildren: () => import('./admin/admin.routes').then(m => m.adminRoutes) },
  { path: 'settings', loadComponent: () => import('./settings/settings').then(m => m.Settings) },
  { path: '**', component: NotFound },
];

// app.config.ts
provideRouter(routes, withComponentInputBinding())
```

### Route Parameters (Signal Inputs)

```typescript
// Route: /users/:id?page=1
// Enable with: withComponentInputBinding()

@Component({...})
export class UserDetail {
  id = input.required<string>();     // Path param
  page = input<string>('1');         // Query param
}
```

### Functional Guards

```typescript
export const authGuard: CanActivateFn = () => {
  const auth = inject(Auth);
  const router = inject(Router);
  return auth.isAuthenticated() ? true : router.createUrlTree(['/login']);
};

export const roleGuard = (roles: string[]): CanActivateFn => () => {
  const role = inject(Auth).currentUser()?.role;
  return role && roles.includes(role) ? true : inject(Router).createUrlTree(['/unauthorized']);
};

// Route: { path: 'admin', canActivate: [authGuard, roleGuard(['admin'])] }
```

### Resolvers

```typescript
export const userResolver: ResolveFn<User> = (route) => {
  return inject(User).getById(route.paramMap.get('id')!);
};
// Route: { path: 'users/:id', resolve: { user: userResolver } }
// Component: user = input.required<User>();
```

### Programmatic Navigation

```typescript
private router = inject(Router);
this.router.navigate(['/products', id]);
this.router.navigate(['/search'], { queryParams: { q, page: 1 } });
this.router.navigate(['edit'], { relativeTo: this.route });
```

### Nested Routes

```typescript
{ path: 'products', component: ProductsLayout,
  children: [
    { path: '', component: ProductList },
    { path: ':id', component: ProductDetail },
    { path: ':id/edit', component: ProductEdit },
  ],
}
// ProductsLayout must have <router-outlet />
```

---

## Forms — Signal Forms (v21+)

```typescript
interface LoginData { email: string; password: string; }

@Component({
  template: `
    <form (submit)="onSubmit($event)">
      <input [formField]="loginForm.email" />
      @if (loginForm.email().touched() && loginForm.email().invalid()) {
        <p class="error">{{ loginForm.email().errors()[0].message }}</p>
      }
      <button [disabled]="loginForm().invalid()">Submit</button>
    </form>
  `,
})
export class Login {
  loginModel = signal<LoginData>({ email: '', password: '' });
  loginForm = form(this.loginModel, (schemaPath) => {
    required(schemaPath.email, { message: 'Email is required' });
    email(schemaPath.email, { message: 'Invalid email' });
    required(schemaPath.password, { message: 'Password required' });
  });

  onSubmit(event: Event) {
    event.preventDefault();
    submit(this.loginForm, async () => {
      await this.auth.login(this.loginModel());
    });
  }
}
```

### Field State

```typescript
field().valid() / invalid() / errors() / pending()
field().touched() / dirty()
field().disabled() / hidden() / readonly()
field().value()
```

### Conditional Fields

```typescript
hidden(schemaPath.couponField, ({ valueOf }) => valueOf(schemaPath.total) < 50);
disabled(schemaPath.field, condition);
readonly(schemaPath.field);
```

### Custom & Cross-Field Validation

```typescript
validate(schemaPath.username, ({ value }) =>
  value().includes(' ') ? { kind: 'noSpaces', message: 'No spaces' } : null
);

validate(schemaPath.confirmPassword, ({ value, valueOf }) =>
  value() !== valueOf(schemaPath.password) ? { kind: 'mismatch', message: 'Passwords must match' } : null
);
```

### Dynamic Fields (Arrays)

```typescript
orderModel = signal({ items: [{ product: '', quantity: 1 }] });
orderForm = form(this.orderModel, (schemaPath) => {
  applyEach(schemaPath.items, (item) => {
    required(item.product);
    min(item.quantity, 1);
  });
});

addItem() { this.orderModel.update(m => ({ ...m, items: [...m.items, { product: '', quantity: 1 }] })); }
removeItem(i: number) { this.orderModel.update(m => ({ ...m, items: m.items.filter((_, j) => j !== i) })); }
```

---

## Directives

### Attribute Directive

```typescript
@Directive({
  selector: '[appHighlight]',
  host: { '(mouseenter)': 'show()', '(mouseleave)': 'hide()', '[class.highlight]': 'active()' },
})
export class Highlight {
  color = input('yellow', { alias: 'appHighlight' });
  active = signal(false);
  private el = inject(ElementRef<HTMLElement>);

  show() { this.el.nativeElement.style.backgroundColor = this.color(); }
  hide() { this.el.nativeElement.style.backgroundColor = ''; }
}
```

### host Property (in @Directive/@Component)

```typescript
host: {
  'role': 'button',                                    // Static
  '[class.active]': 'isActive()',                      // Dynamic class
  '[attr.aria-disabled]': 'disabled()',                // Dynamic attr
  '[style.--btn-color]': 'color()',                    // CSS variable
  '(click)': 'onClick($event)',                        // Event
  '(keydown.enter)': 'onClick($event)',                // Keyboard
}
```

### Structural Directive (Portal)

```typescript
@Directive({ selector: '[appPortal]' })
export class Portal implements OnInit, OnDestroy {
  target = input<string | HTMLElement>('body', { alias: 'appPortal' });
  // Renders templateRef into target container
}
```

---

## SSR (Server-Side Rendering)

### Setup

```bash
ng add @angular/ssr
```

### Render Modes

```typescript
export const serverRoutes: ServerRoute[] = [
  { path: '', renderMode: RenderMode.Prerender },       // Static at build
  { path: 'products/:id', renderMode: RenderMode.Server },  // Dynamic SSR
  { path: 'dashboard', renderMode: RenderMode.Client }, // SPA only
];
```

### Hydration

```typescript
// Default (enabled)
provideClientHydration()

// Event replay
provideClientHydration(withEventReplay())

// Incremental hydration
@defer (hydrate on viewport) { <app-comments /> }
@defer (hydrate on interaction) { <app-chart /> }
@defer (hydrate on idle) { <app-recommendations /> }
@defer (hydrate never) { <app-static-footer /> }
```

### Browser-Only Code

```typescript
// Platform detection
if (isPlatformBrowser(inject(PLATFORM_ID))) { /* browser only */ }

// After render (SSR-safe)
afterNextRender(() => this.initChart());
afterRender(() => this.updateChart());

// Safe browser APIs
export const WINDOW = new InjectionToken<Window>('Window', {
  providedIn: 'root',
  factory: () => isPlatformBrowser(inject(PLATFORM_ID)) ? window : null,
});
```

### TransferState (Prevent Double Fetch)

```typescript
import { TransferState, makeStateKey } from '@angular/core';
const DATA_KEY = makeStateKey<Data[]>('data');

// Check transfer cache first, fallback to HTTP
if (this.transferState.hasKey(DATA_KEY)) {
  return of(this.transferState.get(DATA_KEY, []));
}
return this.http.get<Data[]>('/api/data').pipe(
  tap(data => { if (isPlatformServer(this.platformId)) this.transferState.set(DATA_KEY, data); })
);
```

---

## Testing

### Basic Component Test (Vitest)

```typescript
import { describe, it, expect, beforeEach } from 'vitest';

describe('Counter', () => {
  let fixture: ComponentFixture<Counter>;
  beforeEach(async () => {
    await TestBed.configureTestingModule({ imports: [Counter] }).compileComponents();
    fixture = TestBed.createComponent(Counter);
    fixture.detectChanges();
  });

  it('should increment count', () => {
    expect(fixture.componentInstance.count()).toBe(0);
    fixture.componentInstance.increment();
    expect(fixture.componentInstance.count()).toBe(1);
  });
});
```

### Testing OnPush with Signal Inputs

```typescript
it('should update on input change', () => {
  const fixture = TestBed.createComponent(OnPushCmpt);
  fixture.componentRef.setInput('data', { name: 'Test' });
  fixture.detectChanges();
  expect(fixture.nativeElement.textContent).toContain('Test');
});
```

### Testing HTTP

```typescript
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';

beforeEach(async () => {
  await TestBed.configureTestingModule({
    providers: [provideHttpClient(), provideHttpClientTesting()],
  }).compileComponents();
  httpMock = TestBed.inject(HttpTestingController);
});

afterEach(() => httpMock.verify());

it('should fetch data', () => {
  const req = httpMock.expectOne('/api/users/1');
  expect(req.request.method).toBe('GET');
  req.flush({ id: '1', name: 'Test' });
  fixture.detectChanges();
});
```

### Mocking Services

```typescript
const mockAuth = {
  user: signal<User | null>(null),
  isAuthenticated: computed(() => mockAuth.user() !== null),
  login: vi.fn(),
};
{ provide: Auth, useValue: mockAuth }
```

### Async Testing

```typescript
it('should debounce', fakeAsync(() => {
  fixture.componentInstance.query.set('test');
  tick(300);
  fixture.detectChanges();
  expect(fixture.componentInstance.results().length).toBeGreaterThan(0);
  flush();
}));

it('should load', waitForAsync(() => {
  fixture.whenStable().then(() => {
    expect(fixture.componentInstance.data()).toBeDefined();
  });
}));
```

---

## Tooling

```bash
ng new my-app --style=scss --routing

ng g c features/profile
ng g s services/auth
ng g d directives/highlight
ng g p pipes/truncate
ng g guard guards/auth    # Functional by default
ng g interceptor interceptors/auth

ng serve --port 4201 --open
ng build -c production
ng test --watch=false --browsers=ChromeHeadless
ng lint --fix
ng update @angular/core @angular/cli
ng add @angular/material
ng add @angular/ssr
```

---

## PATTERNS

### Directive-based Button

```typescript
@Directive({
  selector: "button[appButton], a[appButton], [appButton]",
  host: {
    "[attr.data-size]": "size()",
    "[attr.data-variant]": "variant()",
    "[attr.disabled]": "disabled() || null",
    "[class.disabled]": "disabled()",
  },
})
export class AppButton {
  readonly size = input<"sm" | "md" | "lg">("md");
  readonly variant = input<"primary" | "secondary">("primary");
  readonly disabled = input(false, { transform: booleanAttribute });
}
```

### Expand Component with CSS Grid Animation

```typescript
@Component({
  selector: "app-expand",
  template: `@if (expanded() || animating()) { <div class="wrapper"><ng-content /></div> }`,
  changeDetection: ChangeDetectionStrategy.OnPush,
  host: { "[class._expanded]": "expanded()" },
  styles: `
    :host { display: block; overflow: hidden; transition: grid-template-rows var(--duration, 300ms); grid-template-rows: 0fr; }
    :host._expanded { grid-template-rows: 1fr; }
    .wrapper { overflow: hidden; }
  `,
})
export class AppExpand {
  readonly expanded = input(false);
  readonly expandedChange = output<boolean>();
  private readonly animating = signal(false);
}
```

### Service with Signal State

```typescript
@Injectable({ providedIn: "root" })
export class NotificationService {
  private readonly _items = signal<Notification[]>([]);
  readonly items = this._items.asReadonly();
  readonly count = computed(() => this._items().length);

  show(item: Notification): void { this._items.update(list => [...list, item]); }
  clear(): void { this._items.set([]); }
}
```

### Pure Pipe

```typescript
@Pipe({ name: "appFilter", standalone: true })
export class AppFilterPipe implements PipeTransform {
  transform<T>(items: readonly T[], matcher: (item: T, ...args: unknown[]) => boolean, ...args: unknown[]): T[] {
    return items.filter(item => matcher(item, ...args));
  }
}
```

### Injection Token

```typescript
export const APP_BUTTON_OPTIONS = new InjectionToken<AppButtonOptions>(
  ngDevMode ? "APP_BUTTON_OPTIONS" : "",
  { factory: () => ({ size: "md", variant: "primary" }) }
);
```

### hostDirectives Composition

```typescript
@Directive({ selector: "[appAppearance]", host: { "[class._hover]": "hover()", "[class._active]": "active()" } })
export class AppAppearance { readonly hover = signal(false); readonly active = signal(false); }

@Directive({ selector: "[appButton]", hostDirectives: [AppAppearance] })
export class AppButton { private readonly appearance = inject(AppAppearance); }
```

### Form Control (ControlValueAccessor)

```typescript
@Directive({
  selector: "input[appInput]",
  providers: [{ provide: NG_VALUE_ACCESSOR, multi: true, useExisting: AppInput }],
  hostDirectives: [AppControl],
})
export class AppInput implements ControlValueAccessor {
  private readonly ctrl = inject(AppControl);

  writeValue(v: string): void { this.ctrl.value.set(v); }
  registerOnChange(fn: Function): void { this.ctrl.onChange = fn; }
  registerOnTouched(fn: Function): void { this.ctrl.onTouched = fn; }
  setDisabledState(d: boolean): void { this.ctrl.disabled.set(d); }
}
```

### Dropdown / Overlay

```typescript
@Directive({
  selector: "[appDropdown]",
  hostDirectives: [AppDropdownTrigger],
})
export class AppDropdown {
  readonly dropdown = input<TemplateRef<unknown> | null>(null, { alias: "appDropdown" });
  readonly open = input(false, { alias: "appDropdownOpen" });
}
```

### Test with Page Object / Host Component

```typescript
describe("AppButton", () => {
  @Component({
    standalone: true,
    imports: [AppButton],
    template: `<button appButton data-testid="app-button" (click)="onClick()">Click</button>`,
  })
  class Test { clicked = false; onClick() { this.clicked = true; } }

  it("handles click", () => {
    const fixture = TestBed.createComponent(Test);
    fixture.detectChanges();
    fixture.nativeElement.querySelector("[data-testid='app-button']").click();
    fixture.detectChanges();
    expect(fixture.componentInstance.clicked).toBeTrue();
  });
});
```

### Barrel Export

```typescript
export * from "./constants";
export * from "./tokens";
export * from "./directives";
export * from "./pipes";
export * from "./button.directive";
```

### Accessibility Requirements

Components MUST:
- Pass AXE checks, meet WCAG AA
- Include ARIA attributes for interactive elements
- Support keyboard navigation
- Maintain visible focus indicators

```typescript
@Component({
  selector: 'app-toggle',
  host: {
    'role': 'switch',
    '[attr.aria-checked]': 'checked()',
    'tabindex': '0',
    '(keydown.space)': 'toggle(); $event.preventDefault()',
  },
})
export class Toggle { ... }