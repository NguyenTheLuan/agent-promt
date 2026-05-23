# 📊 REPORT & CHECKLIST

## 10/10 Checklist (score after each feature)

```
Architecture  (1pt): Correct layering? Directive preferred? hostDirectives? SOLID?
Angular       (1pt): input/output? inject? host? OnPush? standalone?
Signals       (1pt): signal/computed? No BehaviorSubject? No method in template?
Template      (1pt): @if/@for? [class]/[style]? trackBy?
Styles        (1pt): tuiWithStyles? data-*? CSS variables?
TypeScript    (1pt): readonly? unknown > any? const > let?
Testing       (1pt): spec co-located? TuiPageObject? automation-id?
Performance   (1pt): OnPush+signals? computed memoize? lazy loading?
Accessibility (1pt): Semantic HTML? ARIA? Don't bake aria-label?
Documentation (1pt): index.ts? changelog? TODO updated? REVIEW updated?

TOTAL: /10
= 10 → ✅ Proceed to next feature
< 10 → ❌ Fix immediately, not yet passed
```

---

## Report after each feature

```markdown
## 📊 REPORT — [Feature Name]

### Files created

- `path/file.ts` — [description]

### Patterns applied

- [pattern 1]
- [pattern 2]

### Self-review

| #   | Criteria     | Score     | Notes |
| --- | ------------ | --------- | ----- |
| 1   | Architecture | 1/1       | ...   |
| 2   | Angular      | 1/1       | ...   |
| ... | ...          | ...       | ...   |
|     | **TOTAL**    | **10/10** | ✅    |

### Changelog

- Added: [feature name]
```

---

## Files to update

| File           | When              | Purpose                  |
| -------------- | ----------------- | ------------------------ |
| `TODO.md`      | After each feature | Check off completed feature |
| `REVIEW.md`    | After each feature | Update score             |
| `CHANGELOG.md` | After each feature | Log changes              |