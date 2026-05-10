# 🔄 WORKFLOW — Quy trình từng bước

---

## Tổng quan

```
PHASE 0: KHỞI TẠO    → Hỏi user → Confirm
PHASE 1: LẬP KẾ HOẠCH → Tạo TODO.md + REVIEW.md
PHASE 2: CODE (lặp)    → Feature 1 → 10/10? → Feature 2 → ...
PHASE 3: KẾT THÚC     → Báo cáo tổng kết
```

---

## PHASE 0: Khởi tạo

### Hỏi user từng câu, chờ trả lời:

```
1. "Dự án này để làm gì?"
2. "Công nghệ? (Angular version, LESS/SCSS, Nx?)"
3. "Cấu trúc thư mục? (monorepo? tên packages?)"
4. "Các features cần tạo? (liệt kê)"
5. "Testing? CI/CD? Deploy?"
6. "Ai maintain?"
```

Sau đó confirm lại trước khi qua Phase 1.

---

## PHASE 1: Lập kế hoạch

### Tạo TODO.md:

```markdown
- [ ] Feature 1: [tên] — Files: [...], Pattern: [...], Deps: [...]
- [ ] Feature 2: ...
```

### Tạo REVIEW.md:

```markdown
Architecture: ⏳ | Angular: ⏳ | Signals: ⏳ | Template: ⏳
Styles: ⏳ | TS: ⏳ | Testing: ⏳ | Perf: ⏳ | A11y: ⏳ | Docs: ⏳
Score: 0/10
```

---

## PHASE 2: Code (vòng lặp)

### Cho MỖI feature:

```
1. Chọn feature từ TODO (chưa tick)
2. Phân tích: files? pattern? cần hỏi user?
3. Code — tạo files, kiểm tra từng file
4. Kiểm tra: compile (nx build/tsc), lint (eslint) → lỗi thì fix
5. Tự chấm 10/10 (checklist trong rules.md)
6. = 10/10? → Tick TODO, update CHANGELOG
7. < 10/10? → Quay lại bước 3 sửa
8. Còn feature? → Lặp
```

### ⚠️ KHÔNG:

- Code feature mới khi feature cũ chưa 10/10
- Sửa feature đã hoàn thành
- Code nhiều features cùng lúc

---

## PHASE 3: Kết thúc

- Update TODO.md, REVIEW.md, CHANGELOG.md
- Báo cáo user: features đã xong, score cuối cùng

---

## Flowchart

```
User yêu cầu → Hỏi → Confirm
→ Tạo TODO + REVIEW
→ [Chọn feature → Code → Kiểm tra → Chấm → <10? sửa → =10? tick]
→ Báo cáo
```
