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

| #   | Rule                                                     |
| --- | -------------------------------------------------------- |
| 1   | Inline host component — KHÔNG file riêng                 |
| 2   | `TuiPageObject` query DOM                                |
| 3   | `automation-id` CHÍNH XÁC, không prefix, không transform |
| 4   | `fixture.detectChanges()` sau mỗi action                 |
| 5   | Test file co-located: `test/feature-name.spec.ts`        |
| 6   | Test 3 thứ: render, action, state                        |
| 7   | Mỗi component public có ComponentHarness                 |
