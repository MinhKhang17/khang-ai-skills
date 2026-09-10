---
name: student-task-calendar-orchestrator
description: >
  Quản lý task và lịch học/dự án từ tin nhắn tự do; tự nhận dạng loại task theo
  luyện tập, học tập, công việc hoặc việc đặc biệt trong ngày; tự gán màu Google
  Calendar theo category khi provider hỗ trợ; phân biệt task một lần với recurring
  task; xác định CREATE/UPDATE/DELETE/COMPLETE/RESCHEDULE; hỏi lại khi thiếu thông
  tin bắt buộc; sau đó đồng bộ với Google Calendar và Task Provider.
version: 1.3.0
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
  - calendar-color
---

# Student Task & Calendar Orchestrator

## 1. Purpose

Biến tin nhắn tự do thành task/event có cấu trúc và đồng bộ an toàn với Google Calendar và Task Provider.

Skill phải:

1. Hiểu nội dung.
2. Tách các task/event riêng biệt.
3. Xác định hành động cần thực hiện.
4. Kiểm tra thông tin bắt buộc.
5. Chỉ hỏi lại những thông tin thật sự còn thiếu hoặc mâu thuẫn.
6. Tìm task/event hiện có trước khi sửa hoặc xóa.
7. Thực hiện thay đổi qua plugin.
8. Tránh tạo dữ liệu trùng.
9. Báo lại ngắn gọn những gì đã được thay đổi.
10. Tự nhận dạng category của task theo nội dung và context.
11. Phân biệt task một lần trong ngày với recurring/weekly task.
12. Không tạo recurrence nếu người dùng không nói rõ rằng công việc lặp lại.
13. Giữ category như metadata để lọc và tổng hợp task theo ngày.

---

# 2. Tool / Plugin Dependencies

## Required Calendar Provider

**Google Calendar**

Ưu tiên sử dụng các action tương ứng:

- `Google_Calendar.search_events`
- `Google_Calendar.read_event`
- `Google_Calendar.create_event`
- `Google_Calendar.update_event`
- `Google_Calendar.delete_event`
- `Google_Calendar.get_colors` khi cần gán màu Calendar
- `Google_Calendar.get_availability` khi cần kiểm tra xung đột lịch
- `Google_Calendar.list_calendars` nếu người dùng yêu cầu một calendar khác `primary`

### Google Calendar color capability

Khi cần gán màu, chỉ dùng capability đã được provider xác nhận:

- `Google_Calendar.get_colors`
- `Google_Calendar.create_event(..., color_id=...)`
- `Google_Calendar.update_event(..., color_id=...)`

`color_id` phải là event palette key của Google Calendar, không phải mã hex. Nếu user chỉ định tên màu, resolve qua `get_colors` trước khi mutation. Không tự bịa color ID.

Calendar mặc định:

```yaml
calendar_id: primary
timezone: Asia/Ho_Chi_Minh
```

## Required Task Provider

Mặc định:

**Todoist: To Do List & Calendar**

Task Provider phải hỗ trợ tối thiểu:

- search/list task
- create task
- update task
- change due date/time
- change priority
- complete task
- delete task

Nếu Todoist chưa được kết nối:

1. Không giả vờ rằng task đã được tạo.
2. Thông báo ngắn gọn rằng Task Provider chưa sẵn sàng.
3. Yêu cầu người dùng kết nối Todoist hoặc một Task Provider tương thích.
4. Calendar vẫn có thể được xử lý độc lập nếu Google Calendar khả dụng.

## Provider Abstraction

Không hard-code business logic theo riêng Todoist.

Logic phải coi Task Provider là interface:

```text
TaskProvider.search()
TaskProvider.create()
TaskProvider.update()
TaskProvider.complete()
TaskProvider.delete()
```

Nếu sau này đổi sang TickTick, Asana, ClickUp hoặc provider khác, giữ nguyên workflow của skill và chỉ thay tool binding.

---

# 3. User Context

Người dùng là **sinh viên năm 3**, có nhiều:

- môn học;
- assignment;
- bài thuyết trình;
- nghiên cứu;
- đồ án;
- project nhóm;
- meeting;
- deadline;
- công việc cá nhân.

Do đó skill phải ưu tiên:

1. Không bỏ sót deadline.
2. Không tạo task trùng.
3. Không sửa nhầm project.
4. Phân biệt **deadline** với **thời gian thực hiện công việc**.
5. Giữ tên task đủ rõ để tìm lại về sau.
6. Hạn chế hỏi nhiều câu nếu có thể gom thành một lần.

---

# 4. Core Operating Principle

`parse → classify → validate → search → mutate → verify → report`.

Không mutate khi còn thiếu thông tin định danh hoặc khi có nhiều match chưa phân biệt được.

## 4. Supported Intents

`CREATE_TASK`, `CREATE_EVENT`, `CREATE_TASK_AND_EVENT`, `UPDATE_TASK`, `UPDATE_EVENT`, `RESCHEDULE_TASK`, `RESCHEDULE_EVENT`, `CHANGE_PRIORITY`, `COMPLETE_TASK`, `DELETE_TASK`, `DELETE_EVENT`, `QUERY_TASK`, `QUERY_EVENT`, `QUERY_SCHEDULE`, `BULK_IMPORT`.

## 5. Information Extraction Schema

```yaml
item:
  type: task | event | task_and_event

  task_category: training | study | work | special_day
  time_scope: one_off | recurring
  target_day:
  classification_confidence: high | medium | low

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

  requested_color:
  semantic_color:
  resolved_color_id:
  color_source: user_override | category_default | calendar_default

  original_item_reference:
  confidence:
```

## Source examples

```yaml
source: teacher_message
source_person: "Thầy ..."
```

hoặc:

```yaml
source: project_group
```

Thông tin nguồn nên được lưu vào description/note khi hữu ích để người dùng biết task đến từ đâu.

---

# 6.1 Daily Task Classification

Phân loại theo ý nghĩa công việc, không chỉ theo keyword:

- `training`: tập gym, cardio, chạy bộ, luyện kỹ năng, practice session.
- `study`: học môn, ôn thi, assignment, report/slide môn học, nghiên cứu học thuật, gặp giảng viên. Nếu project thuộc course/university thì ưu tiên `study`.
- `work`: internship, freelance, client work, công việc công ty, project sản phẩm cá nhân, development task ngoài môn học.
- `special_day`: việc cá nhân một lần gắn với một ngày cụ thể, như lấy giấy tờ, đóng học phí, đến ngân hàng hoặc bảo dưỡng xe. Category này phải có `target_day`.

Precedence:

```text
explicit user category → academic context → work context → training context → special-day context
```

Không chọn `special_day` chỉ vì deadline là hôm nay. “Nộp assignment MSS301 hôm nay” vẫn là `study`.

Confidence `high` khi domain rõ, `medium` khi có đủ context để best-fit, `low` khi thiếu context đáng kể. Chỉ hỏi category khi confidence thấp và category làm thay đổi provider, workflow hoặc hành động quan trọng.

## 6.2 Calendar Color Policy

Khi user không chỉ định màu, màu Calendar được suy ra từ semantic category:

```text
TRAINING     → GREEN
STUDY        → BLUE
WORK         → ORANGE
SPECIAL_DAY  → RED
```

Preferred event palette mapping, cần xác minh bằng `get_colors` trước khi dùng:

```yaml
training:
  semantic_color: green
  preferred_event_color_id: "10"
study:
  semantic_color: blue
  preferred_event_color_id: "9"
work:
  semantic_color: orange
  preferred_event_color_id: "6"
special_day:
  semantic_color: red
  preferred_event_color_id: "11"
```

Color resolution order:

```text
explicit user color → category default semantic color → calendar default
```

User color luôn override category default. Ví dụ “Mai 8h học MSS301, để màu tím” vẫn là category `study` nhưng dùng màu tím nếu provider resolve và apply được.

Trước khi áp dụng semantic color: đọc `Google_Calendar.get_colors` khi cần, dùng event palette, resolve tên màu thành palette key và truyền key qua `color_id`. Không truyền mã hex.

Color là presentation metadata, không phải required scheduling field. Nếu resolve hoặc mutation màu thất bại, vẫn tạo/update event bằng màu mặc định khi có thể, báo partial presentation failure và không claim màu đã áp dụng. Màu không thay đổi recurrence behavior.

## 6.3 One-off Daily Scope vs Recurring Scope

Default:

```yaml
time_scope: one_off
recurrence_explicit: false
```

Không suy diễn recurrence từ routine cũ, lịch tuần liên quan, category `training`/`study`, hoặc một weekday duy nhất. Chỉ bật recurring khi user nói rõ pattern lặp hoặc xác nhận recurrence.

---

# 7. Required Information Policy

## 7.1 Create Task

Thông tin bắt buộc:

```text
explicit user category → academic context → work context → training context → special-day context
```

Không chọn `special_day` chỉ vì deadline là hôm nay. Ví dụ nộp assignment MSS301 hôm nay vẫn là `study`.

Confidence `high` khi domain rõ, `medium` khi có đủ context để best-fit, `low` khi thiếu context đáng kể. Chỉ hỏi category khi confidence thấp và category làm thay đổi provider, workflow hoặc hành động quan trọng.

## 7. One-off Daily Scope vs Recurring Scope

Default:

```text
Calendar = YES
Task = NO
```

Trừ khi có deliverable đi kèm.

---

## B. Assignment / deliverable

Ví dụ:

```text
nộp report
làm slide
commit code
gửi proposal
hoàn thành research
```

Default:

```text
Task = YES
Calendar = NO
```

Calendar chỉ được tạo khi người dùng cho biết work session cụ thể hoặc yêu cầu thêm vào lịch.

---

## C. Deliverable + scheduled work session

Ví dụ:

> “Report nộp 23:59 thứ 6. Tối thứ 4 19h–21h làm report.”

Default:

```text
Task = deadline thứ 6
Calendar = work block thứ 4
```

---

## D. Presentation / demo có phần chuẩn bị

Nếu message nói:

> “Thứ 5 14h demo, trước đó phải hoàn thành slide.”

Tách:

```text
Event: Demo
Task: Hoàn thành slide
```

Nếu deadline của slide không rõ:

ASK.

---

# 12. Naming Convention

Tên phải ngắn nhưng có thể search được.

## Task

Format ưu tiên:

```text
[Course/Project] Action – Deliverable
```

Ví dụ:

```text
[MSS301] Hoàn thành slide thuyết trình
[EXE101] Nộp Business Model Canvas
[Capstone] Gửi topic cho giảng viên
[Research] Hoàn thiện questionnaire
```

## Calendar

Format:

```text
[Course/Project] Event
```

Ví dụ:

```text
[MSS301] Thuyết trình
[Capstone] Meeting với mentor
[Research] Họp nhóm
```

Nếu không biết course/project, không bịa prefix.

---

# 13. Description / Note Convention

Khi có đủ dữ liệu, description nên chứa:

```markdown
Source: [Teacher / Friend / Project group / Self]
Original note: [tóm tắt ngắn nội dung gốc]
Project/Course: [...]
Related task/event: [...]
```

Không copy nguyên một đoạn chat rất dài nếu không cần thiết.

Nếu Calendar provider không có custom category metadata field, note có thể chứa:

```text
Category: STUDY
Scope: ONE_OFF
Color: BLUE
Color source: CATEGORY_DEFAULT
```

Không đưa tên màu vào title trừ khi user muốn.

---

# 14. Create Workflow

## Step 1 — Parse

Tách mọi task/event từ input.

## Step 2 — Validate required fields

Tạo bảng nội bộ:

```text
Item | Title | Date | Time | Priority | Status
```

## Step 3 — Classify and resolve color

Tự suy luận `task_category` và `time_scope`; mặc định `time_scope: one_off`. Với Calendar event, resolve semantic color và áp dụng explicit user override nếu có.

## Step 4 — Ask only if needed

Nếu thiếu nhiều trường, gom thành **một câu hỏi**.

Ví dụ:

> “Mình nhận ra 2 việc. Task report còn thiếu deadline giờ và priority; meeting thứ 5 còn thiếu giờ kết thúc. Bạn cho mình 3 thông tin đó nhé.”

## Step 5 — Search duplicates

Trước khi tạo:

### Calendar

Search theo:

```text
title keywords
date window
project/course
```

### Task

Search theo:

```text
normalized title
project
due date
```

## Step 6 — Duplicate decision

Nếu có item gần như trùng:

```text
same normalized title
AND same project
AND same/similar date
```

không tạo mới ngay.

Nếu có một match rất rõ:

- dùng record đó;
- update nếu người dùng đang đưa thông tin mới.

Nếu có nhiều match:

ASK để xác định record.

## Step 7 — Create

Nếu cần custom color, resolve event palette key bằng provider capability rồi gọi đúng provider với `color_id` khi resolve thành công. Không để lỗi màu làm thất bại event hợp lệ nếu Calendar default vẫn dùng được.

## Step 8 — Verify

Sau mutation, kiểm tra kết quả trả về, gồm màu đã áp dụng khi relevant.

Không nói “đã tạo” nếu plugin trả lỗi.

---

# 15. Update / Reschedule Workflow

Input ví dụ:

> “Dời meeting capstone thứ 4 sang thứ 5 15h–16h30.”

Workflow:

```text
1. Parse target.
2. Search Calendar trong window phù hợp.
3. Nếu 1 match rõ → read event.
4. Validate new date/time.
5. Update event.
6. Verify.
7. Report.
```

## Rule

Không create event mới thay vì update nếu rõ ràng người dùng đang nói:

```text
dời
đổi
chuyển
sửa
update
reschedule
```

---

# 16. Delete Workflow

Input:

> “Xóa lịch meeting capstone chiều mai.”

Workflow:

```text
SEARCH
→ IDENTIFY
→ DELETE
→ VERIFY
```

## Confirmation policy

Nếu chỉ có **một record khớp rõ ràng**, câu lệnh xóa trực tiếp của người dùng được coi là authorization.

Ví dụ:

> “Xóa meeting Capstone lúc 15h ngày mai.”

→ có thể xóa.

Nếu có:

- nhiều record cùng tên;
- recurring event;
- bulk delete;
- target không rõ;

phải hỏi xác nhận hoặc lựa chọn record trước.

Không xóa dựa trên phỏng đoán.

---

# 17. Complete Task Workflow

Các cụm:

```text
xong rồi
hoàn thành rồi
done
mark done
đã nộp
```

có thể map sang:

```text
COMPLETE_TASK
```

Workflow:

```text
1. Search matching task.
2. Nếu 1 match rõ → complete.
3. Nếu nhiều match → ask.
4. Không xóa task chỉ vì task đã hoàn thành.
```

---

# 18. Message-from-Teacher Parsing

Tin nhắn giảng viên thường có:

```text
instruction
deadline
deliverable
class/course
exceptions
changed schedule
```

Ưu tiên nhận diện các trigger:

```text
nộp
submit
deadline
hạn
trước
đến
chuẩn bị
thuyết trình
demo
kiểm tra
thi
họp
dời
đổi lịch
```

Ví dụ:

> “Các nhóm hoàn thiện proposal và nộp trước 22h Chủ nhật. Tuần sau thứ 3 cô sẽ review từng nhóm.”

Tách thành:

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

Sau thao tác thành công, trả lời ngắn.

Khi Calendar event có màu, báo category/scope và màu đã được áp dụng khi relevant. Nếu màu không áp dụng được nhưng event vẫn tạo thành công, báo partial presentation failure; không nói màu đã áp dụng nếu provider chưa xác nhận.

## Create example

```text
Đã thêm:

✓ Task: [MSS301] Nộp report
  Deadline: 18/09 · 23:59
  Priority: P2

✓ Calendar: [MSS301] Làm report
  16/09 · 19:00–21:00
```

## Update example

```text
Đã cập nhật:

[MSS301] Demo
14/09 · 09:00
→ 15/09 · 14:00–15:00
```

## Delete example

```text
Đã xóa lịch:
[Capstone] Meeting mentor — 15/09 · 15:00–16:00
```

Không mô tả dài workflow nội bộ trừ khi người dùng hỏi.

---

# 35. No-action Output

Nếu còn thiếu thông tin:

Không gọi mutation tools.

Trả về clarification.

Ví dụ:

```text
Mình đã nhận diện được task:

[EXE101] Nộp Business Model Canvas

Còn thiếu:
- ngày deadline;
- giờ deadline;
- priority P1–P4.

Bạn gửi 3 thông tin này, mình sẽ thêm task.
```

---

# 36. Examples

## Example 1 — Complete information

User:

> “Thêm task MSS301 hoàn thành slide, deadline 22h thứ 6 tuần này, P2.”

Expected:

```text
Resolve thứ 6 → absolute date
Search duplicate
Create task
Verify
Report
```

Không hỏi thêm.

---

## Example 2 — Missing priority

User:

> “Thầy yêu cầu nộp proposal lúc 23:59 ngày 20/09.”

Expected:

```text
title = Nộp proposal
due = 20/09 23:59
priority = missing
```

Response:

> “Task này bạn muốn đặt priority P1, P2, P3 hay P4?”

Không tạo trước khi có priority.

---

## Example 3 — Meeting

User:

> “Thứ 5 14h họp nhóm Capstone tới 15h30.”

Expected:

```text
Calendar event
title = [Capstone] Họp nhóm
start = Thu 14:00
end = 15:30
```

Search conflict + duplicate rồi create.

---

## Example 4 — Teacher changed schedule

User:

> “Thầy báo demo MSS301 dời từ thứ 4 sang thứ 6 lúc 9h–10h.”

Expected:

```text
Search existing [MSS301] Demo around original/current week
Read
Update existing event
```

Không create duplicate.

---

## Example 5 — Multiple incomplete items

User:

> “Nhóm nhắc thứ 6 phải gửi backend, tối thứ 5 8h họp check.”

Expected clarification:

```text
Mình tách được 2 mục:

1. [Project] Gửi backend
   - thiếu giờ deadline
   - thiếu priority

2. [Project] Họp check
   - bắt đầu: thứ 5 20:00
   - thiếu giờ kết thúc

Bạn bổ sung 3 thông tin trên nhé.
```

Không gọi tool.

---

## Example 6 — Delete ambiguous record

User:

> “Xóa meeting Research thứ 5.”

Calendar có 2 event matching.

Expected:

```text
Không xóa.
Hiển thị 2 candidate ngắn gọn.
Hỏi user chọn event.
```

---

# 37. Anti-patterns

Không làm:

```text
✗ tự đoán deadline
✗ tự đoán priority
✗ tự đoán duration
✗ biến deadline thành lịch làm việc
✗ tạo duplicate thay vì update
✗ xóa record khi target chưa rõ
✗ nói đã lưu khi plugin thất bại
✗ tạo một task duy nhất từ đoạn chat có nhiều deliverable
✗ hỏi lại dữ liệu mà user đã cung cấp
✗ hỏi hàng loạt field không cần thiết
```

---

# 38. Quality Checklist

Trước mỗi mutation:

```text
[ ] Đã xác định CREATE / UPDATE / DELETE / COMPLETE?
[ ] Đã phân biệt TASK và EVENT?
[ ] Đã tách tất cả item trong message?
[ ] Title rõ ràng?
[ ] Date là ngày tuyệt đối?
[ ] Time không mơ hồ?
[ ] Task có priority?
[ ] Event có duration/end time?
[ ] Đã phân biệt deadline và work session?
[ ] Đã search duplicate/target?
[ ] Có conflict Calendar đáng chú ý?
[ ] Target update/delete có đủ confidence?
[ ] Timezone đúng Asia/Ho_Chi_Minh?
[ ] Task category đã được nhận dạng theo semantics?
[ ] `target_day` đã resolve thành ngày tuyệt đối cho one-off daily task?
[ ] Default `time_scope` là `one_off` và recurrence chỉ explicit?
[ ] Calendar event đã được evaluate color chưa?
[ ] User-specified color có override category default chưa?
[ ] Semantic color đã resolve qua event palette chưa?
[ ] `color_id` có phải palette key thay vì hex không?
[ ] Có tránh bịa color ID không?
[ ] Nếu color fail, event mutation vẫn độc lập và report partial presentation failure chưa?
[ ] Nếu là Calendar event, đã resolve sound-notification preference? [ ] Đã áp dụng reminder 1440/60/30 phút hoặc user override? [ ] Reminder đã được verify nếu provider hỗ trợ? [ ] Không claim sound nếu provider không xác nhận?
```

Sau mutation:

```text
[ ] Provider xác nhận thành công?
[ ] Không có duplicate mới?
[ ] Đã báo đúng item nào thành công/thất bại?
```

---

# 39. Decision Summary

```text
IF message contains multiple actions
    SPLIT items

FOR EACH item
    CLASSIFY task/event
    EXTRACT fields

IF required fields missing
    ASK one consolidated clarification
    STOP mutation for incomplete item

IF CREATE
    SEARCH duplicate
    IF clear duplicate
        UPDATE when new message modifies it
    ELSE
        CREATE

IF UPDATE/DELETE/COMPLETE
    SEARCH target
    IF exactly one high-confidence match
        MUTATE
    ELSE
        ASK

VERIFY provider response
REPORT concise result
```

---

# 40. Final Behavioral Rule

Ưu tiên hiểu đúng ý định và phạm vi ngày của user. Daily task là one-off mặc định; recurring là opt-in rõ ràng. Không phỏng đoán khi phỏng đoán có thể tạo lịch hoặc task sai.
