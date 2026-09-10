# ChatGPT Skill Library

Bộ khung này dùng để tổ chức các **Skill** theo dạng module độc lập, dễ đọc bởi AI và dễ mở rộng về sau.

## Cấu trúc tổng quát

```text
chatgpt-skill-library/
├── README.md
├── SKILL_INDEX.md
├── CONVENTIONS.md
└── skills/
    ├── _sample-skill/
    │   ├── SKILL.md
    │   ├── references/
    │   ├── examples/
    │   ├── templates/
    │   └── scripts/
    └── code-review/
        ├── SKILL.md
        ├── references/
        ├── examples/
        └── templates/
```

## Cách tạo một Skill mới

1. Copy folder `skills/_sample-skill/`.
2. Đổi tên folder thành dạng `kebab-case`, ví dụ `spring-boot-api-review`.
3. Sửa phần metadata và nội dung trong `SKILL.md`.
4. Chỉ thêm các thư mục cần thiết:
   - `references/`: tiêu chuẩn, guideline, kiến thức nền.
   - `examples/`: input/output mẫu.
   - `templates/`: mẫu đầu ra tái sử dụng.
   - `scripts/`: script hỗ trợ nếu skill cần thao tác tự động.
5. Cập nhật `SKILL_INDEX.md`.

## Nguyên tắc thiết kế

- Một Skill nên giải quyết **một nhóm nhiệm vụ rõ ràng**.
- `SKILL.md` là file bắt buộc và là nguồn hướng dẫn chính.
- Đưa thông tin dài, bảng tiêu chuẩn và tài liệu tra cứu sang `references/`.
- Đưa format đầu ra cố định sang `templates/`.
- Đưa ví dụ few-shot sang `examples/`.
- Không nhồi toàn bộ kiến thức vào `SKILL.md`; giữ file này ngắn, có tính điều phối.
- Viết chỉ dẫn dưới dạng hành động cụ thể: `Do`, `Check`, `Return`, `Avoid`.
