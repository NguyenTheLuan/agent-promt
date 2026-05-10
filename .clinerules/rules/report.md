# 📊 REPORT & CHECKLIST

## 10/10 Checklist (chấm sau mỗi feature)

```
Architecture  (1đ): Phân tầng đúng? Directive ưu tiên? hostDirectives? SOLID?
Angular       (1đ): input/output? inject? host? OnPush? standalone?
Signals       (1đ): signal/computed? Không BehaviorSubject? Không method template?
Template      (1đ): @if/@for? [class]/[style]? trackBy?
Styles        (1đ): tuiWithStyles? data-*? CSS variables?
TypeScript    (1đ): readonly? unknown > any? const > let?
Testing       (1đ): spec co-located? TuiPageObject? automation-id?
Performance   (1đ): OnPush+signals? computed memoize? lazy loading?
Accessibility (1đ): Semantic HTML? ARIA? Không bake aria-label?
Documentation (1đ): index.ts? changelog? TODO updated? REVIEW updated?

TỔNG: /10
= 10 → ✅ Qua feature tiếp
< 10 → ❌ Sửa ngay, chưa qua
```

---

## Report sau mỗi feature

```markdown
## 📊 REPORT — [Feature Name]

### Files đã tạo

- `path/file.ts` — [mô tả]

### Patterns áp dụng

- [pattern 1]
- [pattern 2]

### Self-review

| #   | Tiêu chí     | Điểm      | Ghi chú |
| --- | ------------ | --------- | ------- |
| 1   | Architecture | 1/1       | ...     |
| 2   | Angular      | 1/1       | ...     |
| ... | ...          | ...       | ...     |
|     | **TỔNG**     | **10/10** | ✅      |

### Changelog

- Added: [feature name]
```

---

## Files cần update

| File           | Khi             | Mục đích                |
| -------------- | --------------- | ----------------------- |
| `TODO.md`      | Sau mỗi feature | Tick feature hoàn thành |
| `REVIEW.md`    | Sau mỗi feature | Cập nhật điểm           |
| `CHANGELOG.md` | Sau mỗi feature | Ghi thay đổi            |
