---
name: sample-skill
description: Template chuẩn để tạo một skill mới. Copy folder này và thay nội dung theo domain cụ thể.
version: 1.0.0
---

# Sample Skill

## Purpose

Mô tả ngắn gọn Skill này giúp AI làm việc gì và kết quả cuối cùng mong muốn là gì.

## When to use

Dùng Skill này khi:
- Người dùng yêu cầu ...
- Nhiệm vụ có ...
- Cần tuân theo ...

Không dùng khi:
- Nhiệm vụ chỉ là ...
- Có một Skill chuyên biệt hơn.

## Inputs

Xác định dữ liệu cần có trước khi thực hiện.

Ví dụ:
- Mục tiêu
- File hoặc đoạn nội dung đầu vào
- Constraint
- Technology/domain
- Desired output

Nếu thiếu thông tin không quan trọng, dùng best effort.
Chỉ hỏi lại khi thiếu dữ liệu làm thay đổi hoàn toàn kết quả.

## Workflow

### Step 1 — Understand
- Xác định mục tiêu cuối.
- Xác định constraint.
- Phân loại loại nhiệm vụ.

### Step 2 — Inspect
- Đọc input.
- Phát hiện dữ liệu thiếu, contradiction hoặc risk.
- Khi cần, đọc file trong `references/`.

### Step 3 — Execute
- Thực hiện nhiệm vụ theo domain.
- Dùng template trong `templates/` nếu phù hợp.
- Dùng examples chỉ để học pattern, không copy máy móc.

### Step 4 — Validate
- Kiểm tra tính đúng.
- Kiểm tra tính đầy đủ.
- Kiểm tra format.
- Kiểm tra constraint.

### Step 5 — Return
- Trả kết quả trực tiếp.
- Nêu limitation nếu có.
- Không thêm nội dung không phục vụ mục tiêu.

## Output contract

Kết quả nên có:

```text
[Primary result]

[Important assumptions, nếu cần]

[Next action, nếu thật sự hữu ích]
```

## Quality checks

Trước khi hoàn tất, xác nhận:
- [ ] Đã trả đúng yêu cầu chính.
- [ ] Không tự thêm assumption quan trọng.
- [ ] Không bỏ qua constraint.
- [ ] Format dễ tái sử dụng.
- [ ] References được dùng đúng mục đích.

## Constraints

- Không bịa dữ liệu.
- Không biến example thành rule bắt buộc.
- Ưu tiên hướng dẫn có thể kiểm chứng.
- Giữ Skill đủ hẹp để có thể tái sử dụng ổn định.

## Resources

Khi cần đọc thêm:
- `references/reference-sample.md`
- `examples/example-input.md`
- `examples/example-output.md`
- `templates/output-template.md`
