# 🧪 ANGULAR TESTING RULES (Vitest)

---

## Service Testing

### Basic Service

```typescript
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { TestBed } from '@angular/core/testing';
import { Auth } from './auth';

describe('Auth', () => {
  let service: Auth;

  beforeEach(() => {
    TestBed.configureTestingModule({ providers: [Auth] });
    service = TestBed.inject(Auth);
  });

  it('should start unauthenticated', () => {
    expect(service.isAuthenticated()).toBe(false);
  });

  it('should login and update signal state', () => {
    service.login({ id: '1', name: 'Alice' });
    expect(service.isAuthenticated()).toBe(true);
    expect(service.user()?.name).toBe('Alice');
  });

  it('should clear state on logout', () => {
    service.login({ id: '1', name: 'Alice' });
    service.logout();
    expect(service.user()).toBeNull();
  });
});
```

### Service with Dependencies (Mocking)

```typescript
describe('NotificationService', () => {
  let service: NotificationService;
  let mockHttp: { post: Mock };

  beforeEach(() => {
    mockHttp = { post: vi.fn().mockReturnValue(of({ sent: true })) };

    TestBed.configureTestingModule({
      providers: [
        NotificationService,
        { provide: HttpClient, useValue: mockHttp },
      ],
    });
    service = TestBed.inject(NotificationService);
  });

  it('should call HTTP on send', () => {
    service.send('Hello');
    expect(mockHttp.post).toHaveBeenCalledWith('/api/notify', { message: 'Hello' });
  });
});
```

### Service with Signal-Based Mock

```typescript
describe('ProfileComponent', () => {
  const mockUser = signal<User | null>(null);
  const mockAuth = {
    user: mockUser.asReadonly(),
    isAuthenticated: computed(() => mockUser() !== null),
    login: vi.fn(),
    logout: vi.fn(),
  };

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [ProfileComponent],
      providers: [{ provide: Auth, useValue: mockAuth }],
    }).compileComponents();
  });

  it('should show login when not authenticated', () => {
    mockUser.set(null);
    const fixture = TestBed.createComponent(ProfileComponent);
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('Login');
  });
});
```

---

## HTTP Testing

### HttpTestingController

```typescript
import { HttpTestingController, provideHttpClientTesting } from '@angular/common/http/testing';
import { provideHttpClient } from '@angular/common/http';

describe('UserService', () => {
  let service: UserService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      providers: [
        UserService,
        provideHttpClient(),
        provideHttpClientTesting(),
      ],
    });
    service = TestBed.inject(UserService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => httpMock.verify());

  it('should fetch users via GET', () => {
    service.getUsers().subscribe(users => {
      expect(users).toEqual([{ id: '1', name: 'Alice' }]);
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush([{ id: '1', name: 'Alice' }]);
  });

  it('should handle POST', () => {
    service.createUser({ name: 'Bob' }).subscribe();

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual({ name: 'Bob' });
    req.flush({ id: '2', name: 'Bob' });
  });

  it('should handle 404', () => {
    service.getUser('999').subscribe({
      error: (err) => expect(err.status).toBe(404),
    });

    httpMock.expectOne('/api/users/999').flush('Not found', {
      status: 404,
      statusText: 'Not Found',
    });
  });
});
```

---

## DI / Injection Token Testing

```typescript
import { InjectionToken } from '@angular/core';

describe('InjectionToken usage', () => {
  const API_URL = new InjectionToken<string>('API_URL');

  it('should provide and inject token value', () => {
    TestBed.configureTestingModule({
      providers: [{ provide: API_URL, useValue: 'https://api.test.com' }],
    });
    expect(TestBed.inject(API_URL)).toBe('https://api.test.com');
  });

  it('should override with useFactory', () => {
    TestBed.configureTestingModule({
      providers: [
        { provide: API_URL, useFactory: () => `${window.location.origin}/api` },
      ],
    });
    expect(TestBed.inject(API_URL)).toBe(`${window.location.origin}/api`);
  });
});
```

---

## Pipe Testing

```typescript
import { TruncatePipe } from './truncate.pipe';

describe('TruncatePipe', () => {
  const pipe = new TruncatePipe();

  it('should return value when under limit', () => {
    expect(pipe.transform('Hello', 10)).toBe('Hello');
  });

  it('should truncate and append ellipsis', () => {
    expect(pipe.transform('Hello World', 5)).toBe('Hello...');
  });

  it('should handle empty string', () => {
    expect(pipe.transform('', 10)).toBe('');
  });

  it('should handle null/undefined', () => {
    expect(pipe.transform(null as any, 10)).toBe('');
    expect(pipe.transform(undefined as any, 10)).toBe('');
  });
});
```

---

## Directive Testing

```typescript
describe('HighlightDirective', () => {
  @Component({
    standalone: true,
    imports: [HighlightDirective],
    template: `<p [appHighlight]="color">Text</p>`,
  })
  class TestHost {
    color = 'yellow';
  }

  it('should apply background color on mouseenter', () => {
    const fixture = TestBed.createComponent(TestHost);
    fixture.detectChanges();
    const el = fixture.nativeElement.querySelector('p');
    el.dispatchEvent(new MouseEvent('mouseenter'));
    fixture.detectChanges();
    expect(el.style.backgroundColor).toBe('yellow');
  });
});
```

---

## Component Testing

### Basic Rendering

```typescript
describe('Counter', () => {
  let fixture: ComponentFixture<Counter>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({ imports: [Counter] }).compileComponents();
    fixture = TestBed.createComponent(Counter);
    fixture.detectChanges();
  });

  it('should render initial count', () => {
    expect(fixture.nativeElement.textContent).toContain('0');
  });

  it('should increment on button click', () => {
    const button = fixture.nativeElement.querySelector('button');
    button.click();
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('1');
  });
});
```

### OnPush with Signal Inputs

```typescript
describe('UserCard (OnPush)', () => {
  let fixture: ComponentFixture<UserCard>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({ imports: [UserCard] }).compileComponents();
    fixture = TestBed.createComponent(UserCard);
  });

  it('should update on input change via setInput', () => {
    fixture.componentRef.setInput('user', { name: 'Alice' });
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('Alice');

    fixture.componentRef.setInput('user', { name: 'Bob' });
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('Bob');
  });
});
```

### ng-content / Content Projection

```typescript
describe('Card (with content projection)', () => {
  @Component({
    standalone: true,
    imports: [CardComponent],
    template: `<app-card>Projected Content</app-card>`,
  })
  class TestHost {}

  it('should project content', () => {
    const fixture = TestBed.createComponent(TestHost);
    fixture.detectChanges();
    expect(fixture.nativeElement.textContent).toContain('Projected Content');
  });
});
```

---

## Async Testing

### fakeAsync / tick

```typescript
import { fakeAsync, tick, flush } from '@angular/core/testing';

describe('async operations', () => {
  it('should debounce input', fakeAsync(() => {
    const fixture = TestBed.createComponent(SearchComponent);
    fixture.componentInstance.query.set('test');
    tick(300);  // Advance debounce timer
    fixture.detectChanges();
    expect(fixture.componentInstance.results()).toBeDefined();
    flush();
  }));

  it('should handle setTimeout', fakeAsync(() => {
    let called = false;
    setTimeout(() => (called = true), 1000);
    expect(called).toBe(false);
    tick(500);
    expect(called).toBe(false);
    tick(500);
    expect(called).toBe(true);
  }));
});
```

### waitForAsync

```typescript
import { waitForAsync } from '@angular/core/testing';

it('should load async data', waitForAsync(() => {
  const fixture = TestBed.createComponent(DataComponent);
  fixture.whenStable().then(() => {
    expect(fixture.componentInstance.data()).toBeDefined();
  });
}));
```

### Promises & Observables

```typescript
it('should resolve promise', async () => {
  const fixture = TestBed.createComponent(PromiseComponent);
  await fixture.whenStable();
  expect(fixture.componentInstance.result()).toBe('resolved');
});

it('should handle observable with firstValueFrom', async () => {
  const service = TestBed.inject(DataService);
  const result = await firstValueFrom(service.getData());
  expect(result).toEqual([{ id: '1' }]);
});
```

---

## Testing Effect and Computed

```typescript
import { signal, computed, effect } from '@angular/core';

describe('Signals in tests', () => {
  it('should recompute derived signal', () => {
    const items = signal([1, 2, 3]);
    const count = computed(() => items().length);
    expect(count()).toBe(3);

    items.update(arr => [...arr, 4]);
    expect(count()).toBe(4);
  });

  it('should track effect runs', () => {
    const source = signal(0);
    const spy = vi.fn();
    const testEffect = effect(() => { source(); spy(); });
    TestBed.flushEffects();

    expect(spy).toHaveBeenCalledTimes(1);
    source.set(1);
    TestBed.flushEffects();
    expect(spy).toHaveBeenCalledTimes(2);
  });
});
```

---

## Testing Rules

| #   | Rule |
| --- | ---- |
| 1   | Use `TestBed.configureTestingModule` — no manual DI wiring |
| 2   | Mock dependencies with `useValue` / `useFactory`, not spies on real services |
| 3   | Use `TestBed.inject()` instead of `new Service()` |
| 4   | Test pipes as pure functions — instantiate with `new Pipe()` |
| 5   | Test services as isolated units — mock their dependencies |
| 6   | `fixture.detectChanges()` after every state mutation |
| 7   | Use `componentRef.setInput()` for OnPush signal input components |
| 8   | `httpMock.verify()` in `afterEach` to catch pending requests |
| 9   | Use `fakeAsync` + `tick` for debounce/timer operations |
| 10  | Co-locate test files: `feature-name.spec.ts` or `feature-name/__tests__/` |
| 11  | Each `describe` block tests ONE unit (service, pipe, directive, component) |
| 12  | Test public API only — avoid testing private methods |