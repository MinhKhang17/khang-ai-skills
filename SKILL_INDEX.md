# Skill Index

Registry để xác định skill phù hợp trước khi thực hiện yêu cầu.

| Skill | Path | Category | Status | Version | Enabled |
|---|---|---|---|---|---|
| Sample Skill | `skills/_sample-skill/SKILL.md` | Template | Template | 1.0.0 | No |
| Code Review | `skills/code-review/SKILL.md` | Software Engineering | Stable | 1.0.0 | Yes |
| Student Task & Calendar Orchestrator | `skills/student-task-calendar-orchestrator/SKILL.md` | Productivity | Stable | 1.2.0 | Yes |
| Skill Creator & Maintainer | `skills/skill-creator-maintainer/SKILL.md` | Skill Engineering | Stable | 1.0.0 | Yes |

## Routing policy

1. Đọc `manifest.yaml`.
2. Chỉ xét skill có `enabled: true`.
3. Match theo semantic intent, không chỉ theo từ khóa.
4. Ưu tiên skill cụ thể hơn.
5. Nếu nhiều skill cùng phù hợp, xét `priority`.
6. Chỉ compose nhiều skill khi nhiệm vụ thật sự cần.
7. Đọc `SKILL.md` tương ứng sau khi chọn skill.

## Skill summaries

### Code Review

Entrypoint: `skills/code-review/SKILL.md`

Dùng để review code, tìm bug, đánh giá correctness/security/maintainability và đề xuất refactor.

### Student Task & Calendar Orchestrator

Entrypoint: `skills/student-task-calendar-orchestrator/SKILL.md`

Dùng để tạo/sửa/xóa task và lịch, xử lý deadline, dời lịch, đánh dấu task hoàn thành, và chuyển tin nhắn tự nhiên thành task/calendar item.
- tự nhận dạng task theo `Training`, `Study`, `Work`, `Special Day`;
- mặc định task theo ngày là one-off;
- chỉ tạo recurring/weekly task khi recurrence được user nói rõ.

Dependencies:
- Google Calendar: required
- Todoist: recommended

### Skill Creator & Maintainer

Entrypoint: 'skills/skill-creator-maintainer/SKILL.md'

Dùng khi người dùng yêu cầu:

tạo skill mới;

sửa hoặc mở rộng skill;

đổi version;

kiểm tra cấu trúc skill;

thêm hoặc xóa dependency;

disable/deprecate/delete skill;

tạo Change Report;

xác định file, section và line cần cập nhật.
## Maintenance checklist

- [ ] Version trong `manifest.yaml` khớp `SKILL.md`.
- [ ] Path chính xác.
- [ ] `SKILL_INDEX.md` đã cập nhật.
- [ ] `enabled/status/priority` hợp lý.
- [ ] Dependency mới đã được khai báo.
