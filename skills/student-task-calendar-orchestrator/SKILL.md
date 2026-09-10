# Student Task & Calendar Orchestrator — v1.2.0 Update Patch

Target repository:

`MinhKhang17/khang-ai-skills`

Target file:

`skills/student-task-calendar-orchestrator/SKILL.md`

Current version:

`1.1.0`

Target version:

`1.2.0`

SemVer:

`MINOR`

Reason: thêm capability phân loại task theo ngữ nghĩa và daily-scope policy nhưng không phá contract cũ.

---

# PATCH 1 — Metadata

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

YAML frontmatter

Current location:

approximately lines `1–20`

## Replace description with

```yaml
description: >
  Quản lý task và lịch học/dự án từ tin nhắn tự do; tự nhận dạng loại task theo
  luyện tập, học tập, công việc hoặc việc đặc biệt trong ngày; phân biệt task một lần
  với recurring task; xác định CREATE/UPDATE/DELETE/COMPLETE/RESCHEDULE; hỏi lại khi
  thiếu thông tin bắt buộc; sau đó đồng bộ với Google Calendar và Task Provider.
```

## Replace version

```yaml
version: 1.2.0
```

## Add tags

```yaml
  - daily-task
  - task-classification
```

---

# PATCH 2 — Purpose

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`## 1. Purpose`

Current inspection window:

lines `22–50`

Add to the `Skill phải:` list:

```markdown
10. Tự nhận dạng category của task theo nội dung và context.
11. Phân biệt task một lần trong ngày với recurring/weekly task.
12. Không tạo recurrence nếu người dùng không nói rõ rằng công việc lặp lại.
13. Giữ category như metadata để lọc và tổng hợp task theo ngày.
```

---

# PATCH 3 — Information Extraction Schema

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`# 6. Information Extraction Schema`

Current inspection window:

lines `221–290`

Inside `item:`, immediately after:

```yaml
type: task | event | task_and_event
```

add:

```yaml
  task_category: training | study | work | special_day
  time_scope: one_off | recurring
  target_day:
  classification_confidence:
```

Immediately after:

```yaml
recurrence:
```

add:

```yaml
  recurrence_explicit: false
```

Meaning:

- `task_category`: semantic task group.
- `time_scope`: one-off daily item vs recurring item.
- `target_day`: absolute calendar date for one-off daily tasks.
- `classification_confidence`: confidence in inferred category.
- `recurrence_explicit`: true only when recurrence was explicitly stated or explicitly confirmed.

---

# PATCH 4 — Add Daily Task Classification Section

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Insert after:

`# 6. Information Extraction Schema`

and its source examples, before:

`# 7. Required Information Policy`

Current insertion window:

lines `221–300`

Insert:

```markdown
# 6.1 Daily Task Classification

Mỗi task phải được phân loại theo **ý nghĩa công việc**, không chỉ theo từ khóa.

Canonical categories:

```text
TRAINING
STUDY
WORK
SPECIAL_DAY
```

Normalize vào schema:

```yaml
task_category:
  training | study | work | special_day
```

## A. TRAINING — Luyện tập

Dùng cho hoạt động luyện tập thể chất hoặc luyện kỹ năng.

Ví dụ:

```text
tập gym
cardio
chạy bộ
đi bộ theo mục tiêu
tập lái xe
luyện kỹ năng
practice session
```

Ví dụ normalized:

```yaml
task_category: training
time_scope: one_off
```

Không mặc định biến một buổi tập được nhắc tới hôm nay thành lịch tập hàng tuần.

## B. STUDY — Học tập

Dùng cho hoạt động gắn với việc học, môn học, nghiên cứu học thuật hoặc deliverable ở trường.

Ví dụ:

```text
học MSS301
ôn thi
làm assignment
nộp report
làm slide môn học
đọc tài liệu
research cho môn
gặp giảng viên về bài học
```

Nếu project thuộc môn học hoặc university course, ưu tiên `study` thay vì `work`.

## C. WORK — Công việc

Dùng cho công việc ngoài hoạt động học tập trực tiếp.

Ví dụ:

```text
internship
freelance
client work
công việc công ty
project cá nhân mang tính sản phẩm
development task không thuộc môn học
việc nhóm ngoài trường
```

Nếu không rõ một project là project môn học hay công việc:

- dùng context hiện có nếu đủ;
- nếu classification không ảnh hưởng scheduling, chọn best-fit và không hỏi;
- nếu classification ảnh hưởng provider/project destination, hỏi lại.

## D. SPECIAL_DAY — Việc đặc biệt trong ngày

Dùng cho việc một lần có ý nghĩa riêng trong một ngày cụ thể và không nên biến thành thói quen hàng tuần.

Ví dụ:

```text
hôm nay đi lấy giấy tờ
đóng học phí hôm nay
gọi cho mentor trong ngày
mua đồ cần thiết trước tối nay
đến ngân hàng
đưa xe đi bảo dưỡng
việc cá nhân bắt buộc hoàn thành hôm nay
```

`SPECIAL_DAY` phải có `target_day`.

Nếu người dùng nói:

```text
hôm nay
```

resolve thành ngày tuyệt đối theo timezone:

```text
Asia/Ho_Chi_Minh
```

Nếu người dùng nói một ngày cụ thể khác, vẫn có thể dùng `special_day` với `target_day` tương ứng.

## Classification precedence

Ưu tiên theo domain semantics:

```text
explicit category from user
→ course/academic context
→ work/employment context
→ training/practice context
→ special one-off daily context
```

Không phân loại `SPECIAL_DAY` chỉ vì task có deadline hôm nay.

Ví dụ:

```text
"Nộp assignment MSS301 hôm nay"
```

phải là:

```yaml
task_category: study
time_scope: one_off
target_day: <today>
```

không phải `special_day`.

`SPECIAL_DAY` dùng khi **bản chất task là việc đặc biệt/personal one-off**, không phải chỉ vì due date gần.

## Classification confidence

```yaml
classification_confidence:
  high | medium | low
```

- `high`: domain rõ.
- `medium`: có đủ context để best-fit.
- `low`: thiếu context đáng kể.

Classification không phải required input do user cung cấp.

Skill phải tự nhận dạng trước.

Chỉ hỏi category khi:
- confidence thấp;
- và category làm thay đổi nơi lưu, workflow hoặc hành động quan trọng.

Không hỏi category chỉ để hoàn thiện metadata.

---

# 6.2 One-off Daily Scope vs Recurring Scope

Default policy:

```yaml
time_scope: one_off
recurrence_explicit: false
```

Một task/event **không được coi là recurring** chỉ vì:

- nó giống một routine;
- trước đây user từng làm hoạt động tương tự;
- tồn tại lịch tuần liên quan;
- thuộc category `training`;
- thuộc category `study`;
- user nói một weekday duy nhất.

Ví dụ:

```text
"Hôm nay 7h tối tập gym"
```

→ one-off task/event của ngày hôm nay.

Không:

```text
→ every Thursday 19:00
```

## Recurrence chỉ được bật khi có tín hiệu rõ

Ví dụ:

```text
mỗi ngày
hàng ngày
mỗi thứ 2
hàng tuần
thứ 4 hàng tuần
mỗi cuối tuần
2 tuần một lần
lặp lại mỗi tháng
```

Khi đó:

```yaml
time_scope: recurring
recurrence_explicit: true
```

Nếu recurrence có thể hiểu nhiều cách, ASK.

## Daily-first principle

Đối với tin nhắn thông thường, ưu tiên hiểu là **công việc của ngày cụ thể được nói đến**.

```text
date-specific message
→ one_off
```

chỉ chuyển thành recurring khi user nói rõ recurrence.

## Weekly schedule separation

Không tự lấy một daily task rồi ghi nó thành weekly recurring item.

Weekly routines chỉ được:
- đọc để kiểm tra conflict/context nếu skill được phép;
- tạo hoặc sửa khi user yêu cầu lịch lặp;
- không được dùng để suy diễn rằng task mới cũng lặp hàng tuần.
```

---

# PATCH 5 — Create Task Required Information

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`## 7.1 Create Task`

Current inspection window:

lines `275–350`

Do **not** make `task_category` a required user-provided field.

Add after the required-information block:

```markdown
`task_category` và `time_scope` là derived fields.

Skill phải tự suy luận hai field này từ nội dung trước khi tạo task.

Không hỏi user category nếu có thể phân loại với confidence `high` hoặc `medium`.

Default:

```yaml
time_scope: one_off
recurrence_explicit: false
```
```

---

# PATCH 6 — Recurring Events

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`# 22. Recurring Events`

Current inspection window:

lines `920–1005`

Insert immediately after heading:

```markdown
## Explicit recurrence rule

Recurrence là **opt-in**, không phải default.

Nếu message chỉ đề cập:
- hôm nay;
- ngày mai;
- một ngày cụ thể;
- một thứ cụ thể trong tuần;
- một buổi luyện tập;
- một buổi học;
- một task công việc;

thì default:

```yaml
time_scope: one_off
recurrence_explicit: false
```

Chỉ tạo RRULE/recurring task khi user nói rõ pattern lặp hoặc xác nhận recurrence.

Không suy diễn recurrence từ Weekly Master Schedule hoặc routine đã biết.
```

Keep the existing recurrence examples after this new rule.

---

# PATCH 7 — Naming / Metadata Strategy

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`# 12. Naming Convention`

Current inspection window:

approximately lines `580–650`

Add:

```markdown
## Category metadata

Không bắt buộc đưa category vào title.

Ưu tiên title sạch và dễ search:

```text
[MSS301] Nộp report
Tập gym — Pull
Gọi mentor
```

Category nên được lưu dưới dạng provider metadata/label/project khi provider hỗ trợ.

Nếu provider không hỗ trợ category field, thêm vào description/note:

```text
Category: STUDY
Scope: ONE_OFF
```

Không bịa label API nếu provider chưa verify capability.
```

---

# PATCH 8 — Daily Query Behavior

File:

`skills/student-task-calendar-orchestrator/SKILL.md`

Anchor:

`# 25. Natural-language Command Mapping`

Current inspection window:

approximately lines `1025–1090`

Add mappings:

```text
"hôm nay tôi cần làm gì"
"task hôm nay"
"việc đặc biệt hôm nay"
"lịch học hôm nay"
"việc tập luyện hôm nay"
"công việc hôm nay"
→ QUERY / FILTER BY target_day + task_category
```

For daily summaries, group result in this order:

```text
1. SPECIAL_DAY
2. STUDY
3. WORK
4. TRAINING
```

Within each category:
- sort by explicit time when available;
- otherwise sort by priority;
- then deadline.

Do not convert this query into a weekly summary unless user explicitly asks for week/weekly.
```

---

# Expected behavior examples

## Example 1 — Training one-off

User:

> Hôm nay 7h tối tôi tập gym 2 tiếng.

Expected:

```yaml
task_category: training
time_scope: one_off
recurrence_explicit: false
target_day: <today>
```

Do not create a weekly gym recurrence.

## Example 2 — Study

User:

> Tối nay làm report MSS301 trước 10h, P2.

Expected:

```yaml
task_category: study
time_scope: one_off
target_day: <today>
priority: P2
```

## Example 3 — Work

User:

> Mai 9h sửa API cho project cá nhân, xong trước 11h.

Expected:

```yaml
task_category: work
time_scope: one_off
target_day: <tomorrow>
```

## Example 4 — Special day

User:

> Hôm nay nhớ đi lấy giấy tờ trước 4h chiều, P1.

Expected:

```yaml
task_category: special_day
time_scope: one_off
target_day: <today>
priority: P1
```

## Example 5 — Explicit weekly recurrence

User:

> Mỗi thứ 4 lúc 7h tối tập gym 2 tiếng.

Expected:

```yaml
task_category: training
time_scope: recurring
recurrence_explicit: true
```

Only this case should create recurring scheduling.

---

# Validation additions

Add to the existing Quality Checklist:

```text
[ ] Task category đã được nhận dạng?
[ ] Category dựa trên semantics chứ không chỉ keyword?
[ ] target_day đã resolve thành ngày tuyệt đối nếu là one-off daily task?
[ ] Default time_scope là one_off?
[ ] recurrence_explicit chỉ true khi user nói/xác nhận recurrence?
[ ] Không biến task trong ngày thành weekly recurring task?
[ ] Daily query không bị biến thành weekly summary?
```
