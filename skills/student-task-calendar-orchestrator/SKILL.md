---
name: student-task-calendar-orchestrator
description: >
  Quản lý task và lịch học/dự án từ tin nhắn tự do; tự nhận dạng loại task theo
  luyện tập, học tập, công việc hoặc việc đặc biệt trong ngày; phân biệt task một lần
  với recurring task; xác định CREATE/UPDATE/DELETE/COMPLETE/RESCHEDULE; hỏi lại khi
  thiếu thông tin bắt buộc; sau đó đồng bộ với Google Calendar và Task Provider.
version: 1.2.0
language: vi
timezone: Asia/Ho_Chi_Minh
primary_calendar: primary
task_provider: Todoist
category: productivity
tags:
  - student
  - task-management
  - calendar
  - todoist
  - google-calendar
  - project-management
  - daily-task
  - task-classification
---

# Student Task & Calendar Orchestrator

## 1. Purpose

Biến tin nhắn tự do thành task/event có cấu trúc và đồng bộ an toàn với Google Calendar và Task Provider.

Skill phải:

1. Tách từng task/event trong một tin nhắn.
2. Xác định hành động CREATE/UPDATE/DELETE/COMPLETE/RESCHEDULE.
3. Kiểm tra thông tin bắt buộc và chỉ hỏi phần còn thiếu.
4. Tìm bản ghi hiện có trước khi sửa hoặc xóa.
5. Tránh tạo bản ghi trùng.
6. Xử lý ngày, giờ, deadline và timezone `Asia/Ho_Chi_Minh`.
7. Phát hiện xung đột lịch khi cần.
8. Xác minh kết quả sau mọi mutation.
9. Báo lỗi ngắn gọn và trung thực.
10. Tự nhận dạng category của task theo nội dung và context.
11. Phân biệt task một lần trong ngày với recurring/weekly task.
12. Không tạo recurrence nếu người dùng không nói rõ rằng công việc lặp lại.
13. Giữ category như metadata để lọc và tổng hợp task theo ngày.

## 2. Tool / Plugin Dependencies

Required: Google Calendar. Recommended: Todoist hoặc task provider tương thích.

Chỉ dùng action/provider capability đã được xác nhận. Không bịa tên API hoặc báo thành công khi provider trả lỗi.

## 3. Core Operating Principle

`parse → classify → validate → search → mutate → verify → report`.

Không mutate khi còn thiếu thông tin định danh hoặc khi có nhiều match chưa phân biệt được.

## 4. Supported Intents

`CREATE_TASK`, `CREATE_EVENT`, `CREATE_TASK_AND_EVENT`, `UPDATE_TASK`, `UPDATE_EVENT`, `RESCHEDULE_TASK`, `RESCHEDULE_EVENT`, `CHANGE_PRIORITY`, `COMPLETE_TASK`, `DELETE_TASK`, `DELETE_EVENT`, `QUERY_TASK`, `QUERY_EVENT`, `QUERY_SCHEDULE`, `BULK_IMPORT`.

## 5. Information Extraction Schema

```yaml
item:
  type: task | event | task_and_event
  title:
  task_category: training | study | work | special_day
  time_scope: one_off | recurring
  target_day:
  classification_confidence: high | medium | low
  date:
  start_time:
  end_time:
  due:
  timezone: Asia/Ho_Chi_Minh
  priority: P1 | P2 | P3 | unspecified
  status: pending | completed | cancelled
  recurrence:
  recurrence_explicit: false
  calendar_id: primary
  project_or_course:
  source:
  notes:
```

`task_category` và `time_scope` là derived fields. Skill phải suy luận trước khi tạo; không yêu cầu user cung cấp category. `target_day` là ngày tuyệt đối cho daily one-off. `recurrence_explicit` chỉ true khi user nói rõ hoặc xác nhận pattern lặp.

## 6. Daily Task Classification

Phân loại theo ý nghĩa công việc, không chỉ theo keyword:

- `training`: tập gym, cardio, chạy bộ, luyện kỹ năng, practice session.
- `study`: học môn, ôn thi, assignment, report/slide môn học, nghiên cứu học thuật, gặp giảng viên. Nếu project thuộc course/university thì ưu tiên `study`.
- `work`: internship, freelance, client work, công việc công ty, project sản phẩm cá nhân, development task ngoài môn học.
- `special_day`: việc cá nhân một lần gắn với một ngày cụ thể, như lấy giấy tờ, đóng học phí, đến ngân hàng hoặc bảo dưỡng xe. Category này phải có `target_day`.

Precedence:

```text
explicit user category → academic context → work context → training context → special-day context
```

Không chọn `special_day` chỉ vì deadline là hôm nay. Ví dụ nộp assignment MSS301 hôm nay vẫn là `study`.

Confidence `high` khi domain rõ, `medium` khi có đủ context để best-fit, `low` khi thiếu context đáng kể. Chỉ hỏi category khi confidence thấp và category làm thay đổi provider, workflow hoặc hành động quan trọng.

## 7. One-off Daily Scope vs Recurring Scope

Default:

```yaml
time_scope: one_off
recurrence_explicit: false
```

Không suy diễn recurrence từ routine cũ, lịch tuần liên quan, category `training`/`study`, hoặc một weekday duy nhất. “Hôm nay 19h tập gym” là one-off của hôm nay.

Chỉ bật recurring khi có tín hiệu rõ như “mỗi ngày”, “hàng tuần”, “mỗi thứ 4”, “2 tuần một lần”, hoặc user xác nhận recurrence. Nếu pattern mơ hồ, hỏi lại. Không lấy daily task ghi thành weekly item từ Weekly Master Schedule.

## 8. Required Information Policy

Create task cần title và đủ thông tin định danh thời điểm theo provider. Create event cần title, date và start time; end time nếu provider yêu cầu. Không hỏi priority nếu user không cung cấp, lưu `unspecified`.

Category/scope không phải required input. Derived fields phải được suy luận trước khi tạo, với default `one_off` và `recurrence_explicit: false`.

## 9. Date, Time and Priority

Resolve “hôm nay”, “mai” và ngày tương đối theo `Asia/Ho_Chi_Minh`, sau đó lưu ngày tuyệt đối. Hỏi lại khi ngày hoặc giờ có nhiều cách hiểu. Không tự bịa deadline, priority, duration hoặc end time.

## 10. Create Workflow

1. Parse và tách item.
2. Classify category/scope.
3. Validate required fields.
4. Search duplicate theo title, project/course và date window.
5. Nếu một match rõ, update record đó khi user đang bổ sung thông tin; nếu nhiều match, hỏi chọn record.
6. Gọi đúng provider.
7. Verify bằng cách đọc lại kết quả rồi report.

## 11. Update, Reschedule, Delete and Complete

Các từ “dời”, “đổi”, “chuyển”, “sửa”, “update”, “reschedule” map sang update, không create mới khi target rõ. Delete dùng `search → identify → delete → verify`; recurring/bulk hoặc nhiều match cần xác nhận. “Xong rồi”, “đã nộp”, “done” map sang `COMPLETE_TASK`, không xóa task.

## 12. Naming and Metadata

Giữ title sạch và dễ search, ví dụ `[MSS301] Nộp report`. Không bắt buộc đưa category vào title. Lưu category dưới dạng provider metadata/label/project khi capability đã verify; nếu không hỗ trợ, thêm vào note:

```text
Category: STUDY
Scope: ONE_OFF
```

## 13. Natural-language Command Mapping

Các câu “hôm nay tôi cần làm gì”, “task hôm nay”, “việc đặc biệt hôm nay”, “lịch học hôm nay”, “việc tập luyện hôm nay”, “công việc hôm nay” map sang `QUERY / FILTER BY target_day + task_category`.

Daily summary nhóm theo thứ tự `SPECIAL_DAY`, `STUDY`, `WORK`, `TRAINING`; trong mỗi nhóm sort theo giờ explicit, rồi priority, rồi deadline. Không đổi thành weekly summary nếu user không yêu cầu week/weekly.

## 14. Conflict and Clarification Policy

Kiểm tra conflict khi tạo/sửa event có thời gian. Hỏi một câu gộp các trường còn thiếu hoặc các ambiguity quan trọng. Không hỏi để hoàn thiện metadata nếu đã có best-fit và không ảnh hưởng hành động.

## 15. Output Contract

Sau mutation, báo action, title, thời gian tuyệt đối, category/scope và provider result. Nếu partial failure, nêu rõ item nào thành công/thất bại và không che giấu lỗi.

## 16. Quality Checklist

- [ ] Đã parse đủ item và action chưa?
- [ ] Task category đã được nhận dạng theo semantics chưa?
- [ ] `target_day` đã thành ngày tuyệt đối cho daily task chưa?
- [ ] Default `time_scope` có là `one_off` không?
- [ ] `recurrence_explicit` chỉ true khi user nói/xác nhận recurrence chưa?
- [ ] Có tránh biến task trong ngày thành weekly recurring task không?
- [ ] Daily query có bị biến thành weekly summary không?
- [ ] Đã search duplicate trước CREATE chưa?
- [ ] Đã verify sau mutation chưa?

## 17. Final Behavioral Rule

Ưu tiên hiểu đúng ý định và phạm vi ngày của user. Daily task là one-off mặc định; recurring là opt-in rõ ràng. Không phỏng đoán khi phỏng đoán có thể tạo lịch hoặc task sai.
