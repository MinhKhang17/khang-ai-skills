# Skill Conventions

## 1. Naming

Folder:
`kebab-case`

Ví dụ:
- `code-review`
- `requirement-analysis`
- `spring-boot-backend`
- `academic-research`
- `rag-evaluation`

Tên Skill trong metadata nên trùng ý nghĩa với folder.

## 2. SKILL.md

Mỗi Skill phải có các phần tối thiểu:

1. Metadata
2. Purpose
3. When to use
4. Inputs
5. Workflow
6. Output contract
7. Quality checks
8. Constraints

## 3. Progressive disclosure

AI nên đọc theo thứ tự:

1. `SKILL.md`
2. Chỉ đọc `references/` khi cần kiến thức chi tiết
3. Đọc `templates/` khi cần tạo output theo format
4. Đọc `examples/` khi cần bắt chước cách làm
5. Chạy `scripts/` nếu workflow yêu cầu

## 4. References

Mỗi reference nên tập trung vào một chủ đề.

Ví dụ:

```text
references/
├── architecture-rules.md
├── naming-rules.md
├── security-checklist.md
└── api-guidelines.md
```

## 5. Versioning

Khuyến nghị đặt trong `SKILL.md`:

```yaml
version: 1.0.0
```

Quy ước:
- PATCH: sửa wording/bug nhỏ
- MINOR: thêm workflow hoặc rule mới
- MAJOR: thay đổi cách Skill hoạt động

## 6. Không nên

- Không tạo Skill quá rộng kiểu `software-engineering-everything`.
- Không copy cùng một rule vào 10 Skill.
- Không hard-code thông tin dự án riêng nếu Skill được dùng chung.
- Không viết hướng dẫn mơ hồ như “hãy làm thật tốt”.
