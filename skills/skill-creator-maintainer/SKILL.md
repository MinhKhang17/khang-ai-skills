---
name: skill-creator-maintainer
description: >
  Meta-skill để tạo mới, cập nhật, deprecate, xóa và kiểm tra reusable AI skills.
  Bắt buộc phân tích impact lên SKILL.md, manifest.yaml, README.md và SKILL_INDEX.md,
  áp dụng Semantic Versioning, và trả Change Report có file, section, line range,
  version impact, dependency impact và apply order.
version: 1.0.0
language: vi
category: skill-engineering
status: stable
priority: 200
---

# Skill Creator & Maintainer

## Purpose

Dùng skill này khi người dùng yêu cầu:
- tạo skill mới;
- sửa hoặc mở rộng skill hiện có;
- thêm workflow/capability/provider;
- đổi version;
- disable/deprecate/delete skill;
- kiểm tra cấu trúc skill repository;
- tạo patch plan hoặc change report.

Mục tiêu là duy trì một skill repository nhất quán, versioned, dễ review và dễ áp dụng.

## Source of truth

Repository mặc định:

`MinhKhang17/khang-ai-skills`

Branch:

`main`

Trước mọi thay đổi, đọc theo thứ tự:

```text
manifest.yaml
→ SKILL_INDEX.md
→ README.md
→ target SKILL.md hoặc skills/_sample-skill/SKILL.md
→ references/examples/templates/scripts nếu cần
```

Không ưu tiên bản cũ trong hội thoại khi GitHub hiện tại truy cập được.

## Core files

Mọi yêu cầu tạo/sửa skill phải đánh giá 3 core files:

1. `skills/<skill-name>/SKILL.md`
2. `manifest.yaml`
3. `README.md`

Ngoài ra repository hiện sử dụng `SKILL_INDEX.md`, nên luôn đánh giá file này như **registry companion bắt buộc**.

Ngay cả file không cần đổi, Change Report vẫn phải ghi:

```text
Action: NO CHANGE
Reason: ...
```

## Optional files

Chỉ tạo khi thật sự cần:

```text
references/
examples/
templates/
scripts/
schemas/
tests/
CHANGELOG.md
MIGRATION.md
```

Không tạo file phụ chỉ để làm cấu trúc trông đầy đủ.

## Workflow

```text
UNDERSTAND
→ INSPECT REPOSITORY
→ FIND RELATED SKILLS
→ CHECK OVERLAP
→ DECIDE CREATE/UPDATE/DEPRECATE/DELETE
→ DESIGN
→ DETERMINE VERSION
→ DETERMINE FILE IMPACT
→ CREATE OR PATCH
→ VALIDATE
→ RETURN CHANGE REPORT
```

## Intent analysis

Trước khi thiết kế skill, xác định:

```yaml
goal:
inputs:
outputs:
routing_intents:
required_information:
tools:
plugins:
mutations:
dependencies:
failure_modes:
constraints:
```

Chỉ hỏi lại khi thiếu thông tin làm thay đổi kiến trúc hoặc behavior cốt lõi.

## Overlap policy

Nếu capability mới phù hợp rõ với skill hiện có:

`UPDATE EXISTING SKILL`

Không tạo skill mới chỉ vì user dùng cách gọi khác.

Tạo skill mới khi có một hoặc nhiều yếu tố:
- domain độc lập;
- workflow khác rõ;
- dependency khác;
- output contract khác;
- routing intent riêng;
- nhét vào skill cũ làm skill quá rộng.

## Naming

Folder và `name` metadata dùng `kebab-case`.

Ví dụ tốt:
- `skill-creator-maintainer`
- `student-task-calendar-orchestrator`
- `academic-research`

Tránh:
- `misc`
- `helper`
- `everything`
- `new-skill`

## Required SKILL.md sections

Skill mới tối thiểu nên có:

```text
Metadata
Purpose
When to use
When not to use
Inputs
Required information
Workflow
Tool / Plugin dependencies
Output contract
Validation
Failure handling
Constraints
Quality checklist
Examples
Version policy
```

## Metadata contract

```yaml
---
name: <kebab-case-name>
description: >
  <routing-focused description>
version: 1.0.0
language: vi
category: <category>
status: stable
priority: <integer>
---
```

Description phải mô tả WHAT + WHEN + primary intent/input.

## Semantic Versioning

Dùng:

`MAJOR.MINOR.PATCH`

### New skill
`NEW → 1.0.0`

### PATCH
Ví dụ `1.1.0 → 1.1.1`

Dùng khi:
- typo;
- wording;
- example không đổi behavior;
- bugfix nhỏ không đổi contract.

### MINOR
Ví dụ `1.1.0 → 1.2.0`

Dùng khi:
- thêm intent;
- thêm workflow;
- thêm capability;
- thêm supported provider;
- thêm optional behavior tương thích ngược.

### MAJOR
Ví dụ `1.2.0 → 2.0.0`

Dùng khi:
- đổi required input;
- đổi output contract không tương thích;
- đổi default behavior đáng kể;
- đổi schema;
- xóa capability đang dùng;
- breaking mutation policy.

## Library version

Phân biệt:
- skill version;
- `skill_library.version`.

Không tăng library version cho mọi thay đổi skill.

Chỉ tăng library version khi:
- registry schema đổi;
- global loading convention đổi;
- global policy đổi;
- release nhiều skill như một library milestone.

## manifest.yaml rules

Khi tạo skill mới, thêm entry dưới `skills:`:

```yaml
<skill-name>:
  path: skills/<skill-name>/SKILL.md
  version: 1.0.0
  enabled: true
  status: stable
  category: <category>
  priority: <priority>
  intents:
    - <INTENT>
```

Nếu có dependency:

```yaml
dependencies:
  required:
    - <provider>
  recommended:
    - <provider>
```

Khi version skill đổi, version trong manifest phải đổi theo.

## README.md rules

README là repository-level documentation.

Chỉ update khi thay đổi ảnh hưởng:
- repository structure;
- loading policy;
- skill creation convention;
- global provider/tool policy;
- high-level capability summary.

Nếu tạo skill mới nhưng README không liệt kê từng skill, có thể:

`README.md — NO CHANGE`

Nhưng vẫn phải report.

## SKILL_INDEX.md rules

Phải update khi:
- tạo skill;
- đổi tên/path;
- đổi version;
- đổi enabled/status/category;
- deprecate/delete skill;
- thay đổi routing summary đáng kể.

## Line number policy

Người dùng cần biết file + section + line.

Luôn trả cả:
- `Section/Anchor`
- `Line range`

Ví dụ:

```text
File: manifest.yaml
Section: skills.student-task-calendar-orchestrator
Lines: 42–78
Action: UPDATE
```

File mới:

```text
Lines: NEW FILE
```

Nếu provider không cung cấp line chính xác:

```text
Lines: unavailable
Anchor: ## Version Policy
```

Không bịa line number.

## Mandatory Change Report

Mọi yêu cầu tạo/sửa/xóa skill phải kết thúc bằng report sau.

### A. Decision

```text
Action: CREATE | UPDATE | DISABLE | DEPRECATE | DELETE
Skill: ...
Reason: ...
```

### B. Version impact

```text
Skill version: old → new
Library version: old → new | unchanged
SemVer reason: PATCH | MINOR | MAJOR | NEW
```

### C. Core file changes

| File | Action | Section/Anchor | Lines | Version impact | Reason |
|---|---|---|---|---|---|

Luôn có:
- target `SKILL.md`;
- `manifest.yaml`;
- `README.md`.

### D. Registry companion

Luôn report:
- `SKILL_INDEX.md`.

### E. Optional files

Chỉ liệt kê file thật sự cần.

### F. Dependency changes

```text
Added:
Removed:
Unchanged:
```

### G. Validation

```text
[ ] Path correct
[ ] Metadata name matches folder
[ ] SKILL.md version matches manifest
[ ] SKILL_INDEX version matches
[ ] enabled/status valid
[ ] intents registered
[ ] dependencies registered
[ ] README evaluated
[ ] no unnecessary duplicate skill
```

### H. Apply order

Ví dụ:

```text
1. CREATE skills/foo/SKILL.md
2. UPDATE manifest.yaml
3. UPDATE SKILL_INDEX.md
4. UPDATE README.md if required
5. ADD optional files
6. Re-read repository and verify
```

## Patch detail

Khi sửa skill hiện có, report phải chỉ rõ:

```text
File: skills/foo/SKILL.md
Section: ## Required Information
Lines: 120–148
Change:
- ADD ...
- REPLACE ...
- REMOVE ...
```

Không chỉ ghi chung chung `Update SKILL.md`.

Nếu change nhỏ, có thể thêm Before/After.

Nếu change lớn:
- cung cấp complete updated file riêng;
- Change Report chỉ tóm tắt section/line ranges.

## GitHub mutation policy

Nếu connector có write access và user yêu cầu ghi trực tiếp:

1. fetch file hiện tại;
2. lấy SHA;
3. validate change;
4. create/update/delete;
5. không write song song cùng path;
6. verify provider result;
7. re-fetch khi cần;
8. báo commit/result.

Nếu connector chỉ read:
- không nói đã commit;
- tạo patch/files để user áp dụng;
- báo rõ write permission unavailable.

## Disable / Deprecate / Delete

Phân biệt:

### DISABLE
`enabled: false`

Giữ file.

### DEPRECATE
- `enabled: false`
- `status: deprecated`
- ghi replacement/migration nếu có.

### DELETE
- xóa folder;
- xóa manifest entry;
- xóa index entry;
- kiểm tra README references;
- kiểm tra cross-reference từ skill khác.

Trước DELETE phải kiểm tra dependency/cross-reference.

## Dependency impact

Khi thêm/xóa plugin/tool, report:

```text
Provider:
Capability:
Required/Recommended:
Files affected:
Fallback behavior:
```

Nếu tool/provider chưa verify:

`Status: unresolved provider binding`

Không bịa action/API.

## Example — Create

User:
`Tạo skill quản lý email giảng viên.`

Expected:

```text
Decision: CREATE teacher-email-manager
Skill version: NEW → 1.0.0
Library version: unchanged

CREATE skills/teacher-email-manager/SKILL.md
UPDATE manifest.yaml
UPDATE SKILL_INDEX.md
EVALUATE README.md
ADD Gmail dependency if verified
```

## Example — Add capability

User:
`Skill calendar thêm reminder trước 1 ngày và 1 tiếng.`

Expected:

```text
Decision: UPDATE student-task-calendar-orchestrator
Version: 1.1.0 → 1.2.0

SKILL.md: UPDATE
manifest.yaml: UPDATE version
SKILL_INDEX.md: UPDATE version
README.md: NO CHANGE unless global policy changes
```

## Quality gate

Trước khi hoàn tất:

```text
[ ] Đọc source-of-truth mới nhất?
[ ] Kiểm tra overlap?
[ ] CREATE/UPDATE decision hợp lý?
[ ] SemVer đúng?
[ ] 3 core files đều được đánh giá?
[ ] SKILL_INDEX.md được đánh giá?
[ ] Optional files có lý do?
[ ] Line/section chính xác hoặc ghi unavailable?
[ ] Versions đồng bộ?
[ ] Dependencies hợp lệ?
[ ] Có apply order?
[ ] Mutation ngoài đã verify?
```

## Final rule

Mọi output phải giúp người dùng trả lời ngay:

```text
Tạo file nào?
Sửa file nào?
Sửa section nào?
Line nào?
Version nào tăng?
PATCH/MINOR/MAJOR?
Manifest đổi gì?
README đổi không?
SKILL_INDEX đổi không?
Dependency/plugin đổi không?
Áp dụng theo thứ tự nào?
```

Nếu chưa trả lời được các câu trên, task chưa hoàn chỉnh.
