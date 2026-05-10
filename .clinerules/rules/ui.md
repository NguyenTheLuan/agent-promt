# 🎨 UI & STYLE RULES

## Local styles (component .less)

```less
:host {
  display: block;
}
.t-wrapper {
  display: flex;
  gap: 8px;
}
.t-content {
  padding: 16px;
}
```

## Global styles (attribute selector)

```less
[tuiFeature] {
  display: inline-flex;
  &[data-size="m"] {
    padding: 8px;
  }
  &[data-size="l"] {
    padding: 12px;
  }
  &[data-appearance="primary"] {
    background: var(--my-primary);
  }
  &[disabled] {
    opacity: 0.5;
    cursor: not-allowed;
  }
}
```

## Rules

| #   | Rule          | ✅ Đúng                                  | ❌ Sai                           |
| --- | ------------- | ---------------------------------------- | -------------------------------- |
| 1   | Variants      | `[attr.data-size]`                       | CSS class modifier `.btn--large` |
| 2   | Theme         | `var(--my-primary)`                      | Hardcode màu                     |
| 3   | Animation     | CSS Grid `grid-template-rows` transition | JS animation                     |
| 4   | Animation     | LESS mixin `.transition(~'props')`       | Plain CSS                        |
| 5   | Global CSS    | `tuiWithStyles()` với `@import`          | Duplicate CSS per instance       |
| 6   | Accessibility | Semantic HTML + ARIA                     | Chỉ dùng `<div>`                 |
| 7   | aria-label    | Để user tự set                           | Bake vào component host          |
