# khang-ai-skills

A reusable collection of personal AI skills for different tasks, projects, and workflows.

## Purpose

Repository này là **source of truth** cho bộ skill cá nhân của tôi. Mỗi skill là một module độc lập, có thể tái sử dụng, cập nhật và mở rộng mà không phụ thuộc vào một cuộc hội thoại cụ thể.

## Repository structure

```text
khang-ai-skills/
├── README.md
├── manifest.yaml
├── manifest.json
├── SKILL_INDEX.md
├── CONVENTIONS.md
└── skills/
    ├── _sample-skill/
    ├── code-review/
    └── student-task-calendar-orchestrator/
```

## Skill loading policy

1. Đọc `manifest.yaml` trên branch `main`.
2. Đọc `SKILL_INDEX.md` để xác định skill phù hợp.
3. Chỉ load `SKILL.md` của skill được chọn.
4. Chỉ đọc `references/`, `examples/`, `templates/`, `scripts/` khi cần.
5. Luôn ưu tiên phiên bản hiện tại trên GitHub thay vì bản cũ trong hội thoại.

```text
User request
→ manifest.yaml
→ SKILL_INDEX.md
→ matched SKILL.md
→ required resources
→ required tools/plugins
→ execute
→ verify
```

## Source of truth

```yaml
provider: github
repository: MinhKhang17/khang-ai-skills
branch: main
manifest: manifest.yaml
index: SKILL_INDEX.md
```

`manifest.yaml` là registry machine-readable chính. `SKILL_INDEX.md` là registry human/AI-readable. `manifest.json` chỉ được giữ để tương thích với cấu trúc cũ.

## Creating a new skill

1. Copy `skills/_sample-skill/`.
2. Đổi tên folder theo `kebab-case`.
3. Viết hoặc cập nhật `SKILL.md`.
4. Chỉ thêm resource folder khi cần.
5. Đăng ký skill trong `manifest.yaml`.
6. Cập nhật `SKILL_INDEX.md`.
7. Tăng version theo Semantic Versioning khi behavior thay đổi.
