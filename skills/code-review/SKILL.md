---
name: code-review
description: Review code có hệ thống, ưu tiên correctness, security, maintainability và khả năng test.
version: 1.0.0
---

# Code Review

## Purpose

Review code như một senior engineer: tìm lỗi thực tế trước, sau đó mới đến style và optimization.

## When to use

Dùng khi người dùng yêu cầu:
- review source code
- tìm bug
- đánh giá pull request
- đánh giá architecture ở phạm vi code
- đề xuất refactor

## Inputs

Ưu tiên xác định:
- Ngôn ngữ/framework
- Code hoặc file cần review
- Expected behavior
- Constraint của project
- Có được phép sửa code hay chỉ review

## Workflow

### 1. Establish intent
Xác định code đang cố làm gì trước khi đánh giá implementation.

### 2. Review in priority order

1. Correctness
2. Data loss / side effects
3. Security
4. Concurrency / state
5. Error handling
6. API contract
7. Performance
8. Maintainability
9. Testability
10. Style

### 3. Every important finding must include

- Location
- Problem
- Why it matters
- Reproduction or failure scenario khi có thể
- Concrete fix

### 4. Avoid noise

Không report:
- preference thuần cá nhân
- formatting nếu formatter có thể xử lý
- micro-optimization không có impact
- hypothetical issue không có failure path hợp lý

### 5. Validate

Đọc `references/review-checklist.md` trước khi kết luận nếu review có phạm vi lớn.

## Output contract

Sắp xếp finding theo severity:

- Critical
- High
- Medium
- Low

Mỗi finding:

```text
[Severity] Title

Location:
Problem:
Impact:
Recommendation:
```

Nếu không tìm thấy bug đáng kể, nói rõ điều đó thay vì tạo finding giả.

## Constraints

- Không suy đoán behavior của dependency nếu chưa có căn cứ.
- Phân biệt bug với recommendation.
- Không yêu cầu rewrite toàn hệ thống nếu có fix nhỏ hơn.
