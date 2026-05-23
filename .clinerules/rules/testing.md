# 🧪 TESTING RULES

## Test Structure

```typescript
describe("TuiFeature", () => {
  @Component({
    standalone: true,
    imports: [TuiFeature],
    template: `<div tuiFeature automation-id="tui-feature__input"></div>`,
  })
  class Test {}

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

  it("renders", () => {
    expect(pageObject.getByAutomationId("tui-feature__input")).toBeTruthy();
  });
});
```

## Rules

| #   | Rule                                                      |
| --- | --------------------------------------------------------- |
| 1   | Inline host component — NO separate file                  |
| 2   | `TuiPageObject` queries DOM                               |
| 3   | `automation-id` EXACT, no prefix, no transform            |
| 4   | `fixture.detectChanges()` after each action               |
| 5   | Test file co-located: `test/feature-name.spec.ts`         |
| 6   | Test 3 things: render, action, state                      |
| 7   | Each public component has a ComponentHarness              |