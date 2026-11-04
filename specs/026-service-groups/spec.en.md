# Feature Specification: Service Groups Management (basic data, identity, eligibility rules, revisions)

> **Language version: English (translation).** The Arabic [`spec.md`](spec.md) is the source of truth; on any difference, the Arabic version prevails. Both versions use the same IDs (FR-, SC-, G-), and every change must be applied to both files.

**Feature Branch**: `feature/authentication-authorization-specs`

**Created**: 2026-10-28

**Status**: Draft

**Input**: User description: a "Service groups management" specification, same approach (DC §7 – §9, RSD, POC, Figma). User decision 2026-10-28: split into two specifications — **026** (this one): the group, its basic data, identity, goals, eligibility rules, their testing and revisions; and **027**: the "workflow stages" using the **POC workflow tree** (stages, branches, medication administration, linked persons, follow-up, and a path per follow-up visit type).

**Sources**:

| Source | Sections used | Status |
|---|---|---|
| RSD "Eligibility Form and Portal Management" v0.3 — **RSD** | Story «إدارة مجموعات الخدمات» (scenarios 1 – 7) and «نموذج إدارة مجموعات الخدمات» («اسم مجموعة الخدمات» unique, «الوصف» 250, «أهداف مجموعة الخدمات» multiple choice); «إدارة اهداف مجموعات الخدمات»; «إدارة حالات مجموعات الخدمات»; §2.2 (System Admin) | Draft — not yet approved by the client |
| Dynamic configuration document `docs/RSDs/dynamic-configuration-specs.docx` — **DC** | §7 (scenarios 1 – 15, «البيانات الأهلية» replacing «الفئة المستهدفة» and «بيانات أهلية إضافية», AND between rules, the rejection message, saving together, «قواعد الأهلية الحالية لمجموعات الخدمات»); §8 «اختبار قاعدة الأهلية»; §9 «إصدارات إعدادات مجموعات الخدمات»; §0.3 (three-valued logic); §11 points 3, 5 and 6 | Draft |
| Figma `VtUVDWHAZ0O6q3l40Wvq5N`, page "Admin — Service Management" (`726:371`) | "Service Group List Screen" (`812:11113`), "Service Group Details Screen" (`812:11288`) with the tabs «البيانات الأساسية / مراحل سير العمل / شروط الأهلية» and the «هوية الخدمة» section, `896:3203` / `896:3767` / `896:3992` (colours and errors), "Service Identity Developer Notes" (`898:3574`), "Eligibility Rules Screen" (`796:1707`) with «جرّب على مستفيد», "Eligibility Rule Editor Drawer" (`798:18184`), "Eligibility Rule States Drawer" (`800:2603`) | Reviewed 2026-10-28 |
| Prototype (POC) `C:\Projects\MOHP\RuleBuilder`, branch `master` (`03de267`; first written from `b36b2c5`, updated 2026-11-01 from `1a31c4e` and 2026-11-02 from `03de267`) — **POC** | `ServiceGroup` (code, name, description, the five goals, identity: logo, hash and two colours); `EligibilityRule` (a stable rule identity, a version per edit); `ServiceGroupConfigurationRevision` (an immutable full revision per save, v1 on creation, a save with no change skipped); `HealthUnitServiceGroup`; `VisibilityGatedBag` (a field hidden by its visibility condition counts as answered-empty — plan M.39) | Behavioural reference — not an approved source |
| [`001-condition-builder`](../001-condition-builder/spec.en.md), [`002-eligibility-form-fields`](../002-eligibility-form-fields/spec.en.md), [`009-health-units`](../009-health-units/spec.en.md), [`019-authorization`](../019-authorization/spec.en.md), [`025-reference-lists-seed`](../025-reference-lists-seed/spec.en.md) | 001: the condition builder and three-valued logic; 002: fields available in conditions and rules; 009: «مجموعات الخدمات المتاحة بالوحدة»; 019: P-24 `CanManageServiceGroups`; 025: «حالات مجموعات الخدمات» (formerly «حالات الطلب» — G-03) | Draft |

> Per Principle I of the constitution, this specification is derived exclusively from the sources above and the user's decisions. What the sources do not define is listed as **PROVISIONAL** and its results are not a release reference, and every unsettled value is **PENDING** in the Gap Register (constitution, Principle III).

**Tags**: **[FE]** frontend, **[BE]** server and business rules, **[FE+BE]** both, **the server being the final authority**.

## Clarifications

### Session 2026-10-28

- Q: Which workflow model? → A: **the POC workflow tree**. (Input)
- Q: One specification or two? → A: **two**: 026 for these parts, and 027 for the workflow stages. (Input)
- Q: How are service-group changes saved: one save for everything (DC) or «حفظ القاعدة» per rule (Figma)? → A: **one save for the whole group** (basic data, rules and the 027 stages), all or nothing. (G-01 — FR-035)
- Q: Is a saved rule stopped or deleted? → A: **deleted**; it stays in earlier revisions. (G-02 — FR-038)
- Q: What is the name of the list 025 called «حالات الطلب», which the RSD calls «حالات مجموعات الخدمات»? → A: **«حالات مجموعات الخدمات»**, and 025 is updated. (G-03)
- Q: Which groups is a visit bound to and evaluated against? → A: **every service group active in the system**; those available at the visit's unit are selected by default, and those not available are shown unselected and disabled with the «غير متاحة بالوحدة» hint for the nurse, and no case is opened for them (the user revised the first answer "only those available at the unit"). (G-16 — FR-062)
- Q: Which groups are seeded in every environment? → A: **the four groups with their rules from the DC table** (the dynamic configuration document), without «نصائح غذائية للأم». (G-06 — FR-090)
- Q: Are the five goals seeded? → A: **yes**. (G-05 — FR-080)
- Q: Are these limits right: code as in 020, names ≤ 100, rejection message ≤ 250? → A: **yes**. (G-04)
- Q: Does each rule need an English rejection message? → A: **yes, required**. (G-13 — FR-030)
- Q: Is there a maximum number of rules per group? → A: **no**. (G-14 — FR-030)
- Q: Is the new women's-health group name from RSD 0.4 adopted? → A: **yes — «مجموعة خدمات دعم صحة المرأة»**. (G-17)
- Q: Does the seeded chronic rule add BMI ≥ 30 and average BP > 139/89 from Eligibility RSD 0.4? → A: **yes, with «OR»** beside the chronic history. (G-18)
- Q: Which name for the pregnancy group? → A: **«مجموعة خدمات العناية بصحة الأم والجنين»** (Eligibility RSD 0.4). (G-19)

### Session 2026-11-01

- Q: A rule reads a field the form hid by its own visibility condition (e.g. «موضع الورم» when the personal history has no tumours, or «الحمل» for a male), so the group stays "incomplete data" over a field nobody can fill. How is it evaluated? → A: **as in the POC (M.39): the hidden field counts as answered-empty** — a condition on it is not met; if its visibility condition is itself still unknown, the field stays unknown. Documented in 026 only, with a reference to 001 / 004. (G-20 — FR-039)

## Scope

**In scope**:

1. The service-groups list, with activation and deactivation.
2. The «البيانات الأساسية» tab: code, name, description, goals, and «هوية الخدمة» (logo and two colours).
3. The «أهداف مجموعات الخدمات» list (RSD).
4. The «شروط الأهلية» tab: rules with the condition builder, the rejection message, AND between rules, and trying them on a hypothetical beneficiary.
5. Group configuration revisions and binding visits and cases to them (DC §9).
6. Initial data: the four service groups and their rules from the DC.

**Out of scope**:

- The «مراحل سير العمل» tab and all its content — [`027`] (the next specification); 027 adds its content to the revision (FR-060).
- Evaluating eligibility while the eligibility form is filled and showing it to the nurse — DC §10 and the eligibility-form specifications (013).
- Linking the group to health units and the referral hospital — [`009-health-units`](../009-health-units/spec.en.md) FR-046.
- Deleting a service group (not in the RSD).

## Reference Design (Figma)

- **FIG-1 — List** (`812:11113`): «مجموعات الخدمات» as the first item of the System Admin side menu; search «ابحث باسم المجموعة…»; a «قائمة المجموعات» table (count) with group, goals (chips), status («مفعلة» switch), «عرض المزيد»; and a «مجموعة خدمات جديدة» button.
- **FIG-2 — Details** (`812:11288`): a header with the group name and «الهدف: …»; three tabs: «البيانات الأساسية», «مراحل سير العمل» (027), «شروط الأهلية»; and «حفظ» and «إلغاء».
- **FIG-3 — Service identity** (`812:11288`, `896:3203`, `896:3767`, `896:3992`, `898:3574`): «شعار الخدمة (اختياري)» by upload or drag, «مربع 1:1 · PNG أو SVG أو JPG · حتى 2 ميجابايت», 1:1 crop to 512×512, extracting two colours (primary and background) with manual HEX editing and a contrast warning; "shown on the service card the nurse and doctor see".
- **FIG-4 — Eligibility rules** (`796:1707`): "a beneficiary is eligible for this service only when all the following rules hold together"; a numbered card per rule with «تعديل القاعدة», its paths joined by «أو», and «رسالة الرفض للممرض»; «يجب أن تتحقق أيضًا» between rules; «إضافة قاعدة (يجب أن تتحقق مع القواعد السابقة)»; and a side panel «جرّب على مستفيد» with a result per rule.
- **FIG-5 — Rule editor** (`798:18184`) and **its states** (`800:2603`): rule name, rejection message, the condition with «كل الشروط (و) / أي منها (أو)» groups and «استبعاد (NOT)», «اختبار القاعدة» with three results («تحقق القاعدة», «لم تتحقق القاعدة», «بيانات ناقصة»), and the messages «لا يمكن الحفظ بعد…» and «تعذّر الحفظ: …»; Figma's «حفظ القاعدة» button is **not adopted**: there is one save for the whole group (FR-035 — G-01).

## User Scenarios & Testing *(mandatory)*

Single actor: **System Admin**.

### User Story 1 — Groups list and basic data (Priority: P1)

I want to add service groups, edit their basic data and goals, and activate and deactivate them.

**Why P1**: no eligibility evaluation and no cases without a service group.

**Independent Test**: on its screen alone: add a group, edit, deactivate, reactivate and search for it.

**Acceptance Scenarios**:

1. **Given** I am the System Admin, **When** I open «مجموعات الخدمات», **Then** I see the groups list (name, goals, status) with search by name and paging; with no results, «لا توجد مجموعات خدمات مطابقة لبحثك.» (DC). (FR-001, FR-010)
2. **Given** I am adding a group, **When** I enter the code and both names, pick at least one goal and save, **Then** the group is created active with its first revision, and «تم تسجيل خدمة جديدة بنجاح» is shown (DC). (FR-011 – FR-015, FR-061)
3. **Given** a group name already in use, **When** I save, **Then** the system refuses with the duplicate message. (FR-012)
4. **Given** a group not linked to an active health unit, **When** I deactivate then reactivate it, **Then** each change is saved; if it is linked, **Then** the system refuses with «لا يمكن تعطيل مجموعة الخدمات، لأنها مستخدمة بوحدة صحية مفعلة» (RSD). (FR-016, FR-017)
5. **Given** I am in edit mode with no change, **When** I look at the save button, **Then** it is disabled with «لا يوجد تغيير غير محفوظ» (DC). (FR-018)

---

### User Story 2 — Service identity (Priority: P3)

I want to upload a logo for the service, have two colours extracted from it and adjust them, so the service card the nurse and doctor see carries its identity.

**Why P3**: a visual improvement (Figma); the group works without it.

**Independent Test**: upload a square logo, a non-square one and one with an unaccepted format or size, and edit a colour with a valid and an invalid HEX.

**Acceptance Scenarios**:

1. **Given** a square 1 MB PNG logo, **When** I upload it, **Then** it is accepted and a primary and a background colour are extracted and shown in the preview. (FR-020, FR-021)
2. **Given** a file of another format or larger than 2 MB, **When** I upload it, **Then** it is refused and the error state is shown. (FR-020)
3. **Given** a non-square image, **When** I upload it, **Then** a 1:1 crop window opens. (FR-020)
4. **Given** I am editing a colour, **When** I enter a value that is not a 6-digit HEX, **Then** an error is shown and it is not saved; and when the contrast ratio between the two colours is below 3:1, **Then** a warning is shown that does not block saving. (FR-022, FR-023)

---

### User Story 3 — Eligibility rules (Priority: P1)

I want to define the group's eligibility rules with the condition builder, each rule with a name and a rejection message, a beneficiary being eligible only when all rules hold.

**Why P1**: they decide who enters each service group (DC §7).

**Independent Test**: with an existing group: add two rules and edit one, and check the save and the rule version.

**Acceptance Scenarios**:

1. **Given** the «شروط الأهلية» tab, **When** I add a rule, **Then** a card shows the condition builder, the rule name, the rejection message and «اختبار هذه القاعدة», and only fields available in conditions and rules are offered. (FR-030, FR-031)
2. **Given** two rules, **When** I look at the tab, **Then** it states above the rules that a beneficiary is eligible only when all of them hold, with «يجب أن تتحقق أيضًا» between the cards. (FR-032)
3. **Given** a new or edited rule with no name, no rejection message, or whose test was not run after its last edit, **When** I save, **Then** the system refuses with «أكمل اسم كل قاعدة ورسالة الرفض وشغّل الاختبار عليها قبل حفظ الخدمة.» (DC). (FR-034)
4. **Given** a saved rule, **When** I edit and save it, **Then** it is saved as a new version, the previous one is kept, and «قاعدة #رقم — النسخة الحالية: v(رقم)» is shown above its card (DC). (FR-036)
5. **Given** a group with no rules, **When** a beneficiary is evaluated, **Then** they are eligible for it (DC). (FR-033)
6. **Given** a saved rule in revision v3, **When** I delete it, confirm and save, **Then** v4 is created without it, and it stays in v3 and the visits and cases bound to it. (FR-038)

---

### User Story 4 — Rule testing (Priority: P2)

I want to try a rule on a hypothetical beneficiary's data before saving, to make sure it works as I intend.

**Why P2**: saving requires it (DC §7 scenario twelve), but it is a checking tool that changes no data.

**Independent Test**: with one rule: enter data that satisfies the rule, data that does not, and incomplete data.

**Acceptance Scenarios**:

1. **Given** I open «اختبار هذه القاعدة», **When** the test is shown, **Then** I see an input for each field used in the conditions in its eligibility-form shape, and the inputs update when a condition is added or removed (DC §8). (FR-050)
2. **Given** a rule with no conditions, **When** I open the test, **Then** «أضف شروطاً للقاعدة أولاً» is shown and «تشغيل الاختبار» is disabled (DC). (FR-051)
3. **Given** data that satisfies the rule, **When** I run the test, **Then** «تحققت القاعدة» is shown; if it does not satisfy it, **Then** «لم تتحقق القاعدة» with the rejection message; if a field is missing, **Then** «بيانات ناقصة» with the field name. (FR-052)
4. **Given** I ran the test and then edited the conditions, **When** I save, **Then** the test counts as invalid and saving is refused until I run it again (DC). (FR-053)
5. **Given** any test, **When** it runs, **Then** nothing is saved and no beneficiary is affected, and calculated fields (such as age) are computed from the hypothetical data (DC). (FR-054)

---

### User Story 5 — Revisions (Priority: P1)

I want my changes to eligibility and the workflow to apply to new visits only, without changing the requirements of ongoing visits and cases.

**Why P1**: it stops a beneficiary's requirements changing mid-journey (DC §9), and 020 – 025 depend on it.

**Independent Test**: open a visit, then edit a rule, then complete the visit and check it is evaluated with the old rule.

**Acceptance Scenarios**:

1. **Given** a group at revision v2, **When** I edit an eligibility rule and save, **Then** revision v3 is created, and v2 can be neither edited nor deleted. (FR-060)
2. **Given** I edit only the name, description, goals or identity, or save with no actual change, **When** I save, **Then** no new revision is created. (FR-061)
3. **Given** a visit opened on v2, **When** v3 is created before the nurse completes stage two, **Then** its eligibility is evaluated on v2. (FR-062)
4. **Given** open visits and ongoing cases on an earlier revision, **When** I save a change that created a revision, **Then** «تم الحفظ. ستُطبَّق التعديلات على الزيارات الجديدة فقط — هناك (عدد) زيارات مفتوحة و(عدد) حالات جارية ستستمر على الإعدادات السابقة.» is shown (DC). (FR-064)
5. **Given** another admin saved a change to the group after I opened it, **When** I save, **Then** «قام شخص آخر بتعديل إعدادات هذه الخدمة بعد فتحك للصفحة — أعد تحميل الصفحة ثم أعد تعديلاتك» is shown and nothing is saved (DC). (FR-065)

---

### Edge Cases

- **A completely empty new rule card**: ignored on save (DC) (FR-035).
- **Rules not edited**: not saved again and no version created (DC) (FR-035, FR-036).
- **A field in an eligibility rule later deactivated or changed in type**: the field is "used in a condition" under the 002 rules, so the change is refused (FR-037).
- **Deleting a saved rule**: allowed; it leaves the next revision and stays in earlier revisions (FR-038 — G-02).
- **Everything saved together**: either all of it is saved or nothing (DC) (FR-035).
- **Evaluating eligibility with missing data**: "incomplete" under three-valued logic (001; DC §0.3), not a rejection (FR-033).
- **A rule on a field hidden by its visibility condition**: the field counts as answered-empty, so the rule is not met instead of "incomplete data"; while the visibility condition is still unknown (e.g. «النوع» not answered), the group stays "incomplete data" (FR-039 — G-20).

## Requirements *(mandatory)*

### Functional Requirements — Menu and permission

- **FR-001** [FE+BE]: The System Admin side menu MUST show a «مجموعات الخدمات» item (FIG-1), hidden from anyone without the service-groups management permission, and the server refuses any direct access.
- **FR-002** [BE]: The screen and its operations MUST be protected by the **`CanManageServiceGroups`** permission (019 P-24) for the System Admin only; P-24 becomes defined by this specification.

### Functional Requirements — Basic data (Story 1)

- **FR-010** [FE+BE]: The list MUST show the groups (name, goals as chips, status as a switch, «عرض المزيد») with their count, search by group name (ignoring letter case and surrounding spaces), and paging; no-result message «لا توجد مجموعات خدمات مطابقة لبحثك.» (DC; FIG-1).
- **FR-011** [FE+BE]: The **group code** is required, unique, unchangeable after creation, in the 020 lab-test code format (POC — G-04); it is used in revisions and cases.
- **FR-012** [FE+BE]: The Arabic **service group name** is required and unique ignoring letter case and surrounding spaces (RSD), up to 100 characters (G-04); the **English name** is required, up to 100 characters (007); duplicate message «اسم مجموعة الخدمات مستخدم بالفعل» (provisional text).
- **FR-013** [FE+BE]: The **service group description** is optional, up to 250 letters, digits and special characters (RSD), with a «0/250» counter (FIG-2).
- **FR-014** [FE+BE]: The **service group goals** are required, multiple choice from the **active** goals of the «أهداف مجموعات الخدمات» list (FR-080) (RSD).
- **FR-015** [FE+BE]: A group is created **active** together with its first revision v1 (POC; FR-061), showing «تم تسجيل خدمة جديدة بنجاح», and on edit «تم تعديل الخدمة بنجاح» (DC).
- **FR-016** [FE+BE]: The status switch in the list MUST deactivate and reactivate the group, each change saved immediately; a deactivated group is not offered in new choices (e.g. «مجموعات الخدمات المتاحة بالوحدة» in 009) and is not evaluated in new visits, while ongoing cases on it continue on their revision.
- **FR-017** [BE]: Deactivating a group linked to an active health unit (009 FR-046) is refused with «لا يمكن تعطيل مجموعة الخدمات، لأنها مستخدمة بوحدة صحية مفعلة» (RSD).
- **FR-018** [FE]: In edit mode the save button is disabled when nothing changed, with «لا يوجد تغيير غير محفوظ» (DC).

### Functional Requirements — Service identity (Story 2)

- **FR-020** [FE+BE]: The **service logo** is optional: PNG, SVG or JPG up to 2 MB; a non-square image is cropped 1:1 and the result saved at 512×512; anything else is an error state (FIG-3).
- **FR-021** [FE]: On upload a **primary colour** and a **background colour** are extracted from the logo (ignoring transparent pixels and those close to white or black), and uploading a new logo re-extracts them (FIG-3, `898:3574`).
- **FR-022** [FE+BE]: Each colour can be edited manually with a 6-digit HEX value only (`#RRGGBB`), anything else being an error that is not saved; each colour shows its source («افتراضي», «مستخرج من الشعار», «مولّد من اللون الأساسي», «معدّل يدويًا») with «استعادة اللون المستخرج» (FIG-3).
- **FR-023** [FE]: The contrast ratio (WCAG) between the two colours is computed; below 3:1 is a warning that **does not block saving** (FIG-3).
- **FR-024** [FE]: With no logo and no colours the service card shows its default look; the identity appears on the service card the nurse and doctor see, the display itself being in their specifications (FIG-3).

### Functional Requirements — Eligibility rules (Story 3)

- **FR-030** [FE+BE]: Each group has zero or more eligibility rules **with no maximum** (G-14); each rule has: **conditions** required, with the condition builder (001; «كل الشروط (و) / أي منها (أو)» groups and «استبعاد (NOT)»), a required **rule name** up to 100 characters, and **Arabic and English rejection messages**, both required, up to 250 characters each (RSD «رسالة الرفض بالعربي», the English one by user decision — G-13), shown to the nurse in the interface language as the reason for non-eligibility (DC; FIG-4; lengths — G-04).
- **FR-031** [FE+BE]: Only field-registry fields with "available in conditions and rules" enabled are offered in rule conditions (002; DC).
- **FR-032** [BE]: Rules are joined by **AND**: a beneficiary is eligible only when all rules hold, and when a rule does not hold its rejection message is shown as the reason; an "or" between two conditions goes inside the same rule (DC); the explanation is shown above the rules (FIG-4).
- **FR-033** [BE]: A group with no rules makes **every beneficiary eligible** (DC); the evaluation result is three-valued under the 001 logic (DC §0.3): "eligible", "not eligible" (with the rejection messages of the rules that do not hold), "incomplete data" (with the missing fields); the display is in the eligibility form (DC §10); a field hidden by its visibility condition is not "missing" (FR-039).
- **FR-034** [FE+BE]: Saving is refused when a new or edited rule has no name, no rejection message or no condition, or its test was not run after the last edit of its conditions, with «أكمل اسم كل قاعدة ورسالة الرفض وشغّل الاختبار عليها قبل حفظ الخدمة.» (DC); each card shows its state: «جاهزة — ستُحفظ هذه القاعدة تلقائيًا عند الضغط على "تعديل وحفظ".», «أكمل اسم القاعدة ورسالة الرفض وشغّل الاختبار — …», «لا يوجد تغيير في هذه القاعدة.» (DC §8).
- **FR-035** [FE+BE]: **One save for the whole group** (G-01; DC §7 scenario thirteen): the «حفظ» (or «تعديل وحفظ») button saves together the basic data, the identity, every new, edited and deleted rule, and the workflow stages (027) — all of it or nothing; there is no separate save per rule (Figma's «حفظ القاعدة» in `800:2603` is not adopted); moving between tabs keeps unsaved changes; a completely empty new rule card is ignored and an unchanged rule is not saved again (DC).
- **FR-036** [BE]: Editing a saved rule saves a **new version** of it under the same rule identity, keeping the previous versions, and «قاعدة #رقم — النسخة الحالية: v(رقم)» is shown above the card (DC §9 scenario six; POC).
- **FR-037** [BE]: A field used in an eligibility-rule condition counts as "used in a condition" under the 002 rules (deactivating it, changing its type or removing its availability in conditions).
- **FR-038** [FE+BE]: A **saved rule may be deleted** from the group (G-02) after confirming «هل تريد حذف القاعدة؟ ستُطبَّق المجموعة بدونها على الزيارات الجديدة فقط.» (provisional text); the deletion is carried out with the single save (FR-035), so the rule leaves the next revision and stays in earlier revisions and the visits and cases bound to them (FR-060); deleting the last rule makes every beneficiary eligible (FR-033).
- **FR-039** [BE]: In evaluating a group's eligibility, a field **without an answer whose visibility condition (004) definitely does not hold** counts as **answered-empty**, not unknown: a condition on it is **not met** (and under «NOT» it holds), and a comparison with it as the other field is not met; if its visibility condition is itself still unknown (e.g. «النوع» not answered), the field stays unknown and the group "incomplete data", since it may still appear; a field hidden behind another hidden field is hidden too. It applies to the evaluation recorded on the visit, to the beneficiary classifications evaluated on the visit from the same data (not those at a case's close), and to the live preview in the eligibility form, so both always agree; the rule tester (FR-052) is not affected (POC M.39 — G-20). The general three-valued logic stays in [`001-condition-builder`](../001-condition-builder/spec.en.md) FR-041 and the chained conditions in [`004-field-conditions`](../004-field-conditions/spec.en.md), unchanged by this update.

### Functional Requirements — Rule testing (Story 4)

- **FR-050** [FE]: «اختبار هذه القاعدة» inside its card shows an input for each field used in its conditions in its eligibility-form shape, and the inputs update when a condition is added or removed (DC §8); a «جرّب على مستفيد» panel at tab level tests all rules together and shows each rule's result (FIG-4).
- **FR-051** [FE]: A rule with no conditions shows «أضف شروطاً للقاعدة أولاً» and «تشغيل الاختبار» is disabled (DC).
- **FR-052** [FE+BE]: The test result is **three-valued**: «تحققت القاعدة» (with the path that holds), «لم تتحقق القاعدة» (with the rejection message), «بيانات ناقصة» (with the missing field) — as in the real evaluation (FIG-5 `800:2603`; DC §11 point 5; instead of "success / failure" in DC §8), except that a field missing from the sample stays «بيانات ناقصة» even when its visibility condition does not hold (FR-039 does not apply to the tester — POC M.39).
- **FR-053** [FE]: Editing a rule's conditions after the test invalidates it, and it must be run again before saving (DC).
- **FR-054** [BE]: The test **saves no data** and affects no beneficiary, and calculated fields (such as age) are computed from the hypothetical data (DC).

### Functional Requirements — Revisions (Story 5)

- **FR-060** [BE]: Every save that changes the **eligibility rules**, the **workflow stages** (027), or the selected lab tests, exams and questionnaires and their applicability conditions creates a full **new revision** of the group's configuration; earlier revisions **are never edited or deleted** (DC §9; POC). Catalog edits (020 – 024) that touch what is in the group also produce a new revision for it (POC "CatalogChange"; closes the 020 – 024 "revision mechanism").
- **FR-061** [BE]: Editing only the name, description, goals or identity, or saving with no actual change, **creates no revision** (DC); creating a group creates v1 (POC).
- **FR-062** [FE+BE]: When a new visit is opened (stage one — data entry) it is bound to the latest revision of every service group **active in the system** (not only those available at the unit), and its eligibility for all these groups is computed after stage two (the nurse) on that same revision even if the configuration changed between the stages (DC §9). An eligible group **available at the visit's unit** (009 FR-046) is shown **selected by default**; one **not available at the unit** is shown **unselected and disabled** with the hint «غير متاحة بالوحدة» for the nurse (provisional text), and no case is opened for it at this unit; the display itself is in the eligibility form (DC §10 / 013) (G-16).
- **FR-063** [BE]: When a case is opened in a service group it is bound to the revision of the visit it was opened from, and all its stages are completed on that revision until it closes (DC §9).
- **FR-064** [FE+BE]: After a save that created a revision, with open visits or ongoing cases on an earlier revision, «تم الحفظ. ستُطبَّق التعديلات على الزيارات الجديدة فقط — هناك (عدد) زيارات مفتوحة و(عدد) حالات جارية ستستمر على الإعدادات السابقة.» is shown (DC).
- **FR-065** [BE]: If another admin saved a change to the group after I opened it, my whole save is refused with «قام شخص آخر بتعديل إعدادات هذه الخدمة بعد فتحك للصفحة — أعد تحميل الصفحة ثم أعد تعديلاتك» (DC §9 scenario seven).

### Functional Requirements — Service group goals

- **FR-080** [FE+BE]: A **«أهداف مجموعات الخدمات»** list in «خيارات القوائم» (007; RSD «إدارة اهداف مجموعات الخدمات»: the goal name unique), seeded in every environment with (Figma; POC): «صحة الأم والطفل», «السيطرة على الأمراض المزمنة», «الكشف المبكر», «الرعاية الصحية لكبار السن», «الوقاية من الأمراض المعدية» (G-05); deactivating a goal used by an active service group is refused with «لا يمكن تعطيل الهدف، لأنه مستخدم بمجموعة خدمات مفعلة» (RSD).

### Functional Requirements — Initial data

- **FR-090** [BE]: Every environment is seeded with the four service groups and their eligibility rules as in the DC table «قواعد الأهلية الحالية لمجموعات الخدمات» (G-06), and «نصائح غذائية للأم» (Figma) is not seeded; the codes, English names and English rejection messages are in the table after it (G-13):
  - «مجموعة خدمات الأمراض المزمنة» (chronic-disease control): «تاريخ مرضي أو عائلي بمرض مزمن» **or** BMI ≥ 30 **or** average blood pressure > 139/89 (one rule with an OR group — Eligibility RSD 0.4, G-18) — message «لا يوجد تاريخ مرضي شخصي أو عائلي بأمراض مزمنة مؤهلة»;
  - «مجموعة خدمات كبار السن» (healthcare for the elderly): «الفئة العمرية المستهدفة (كبار السن)» age between 65 and 120 — «خارج الفئة العمرية المستهدفة (65-120 سنة)»;
  - «مجموعة خدمات العناية بصحة الأم والجنين» (mother and child health): «الفئة المستهدفة (الأم والجنين)» female and age between 15 and 50 — «خارج الفئة المستهدفة (إناث 15-50 سنة)»; and «الحمل» = «حامل» — «غير حامل»;
  - «مجموعة خدمات دعم صحة المرأة» (early detection): «الفئة المستهدفة (صحة المرأة)» female and age between 18 and 120 — «خارج الفئة المستهدفة (إناث 18 سنة فأكثر)»; and «معايير الكشف المبكر عن سرطان الثدي» (the DC's four paths) — «لا تنطبق معايير الكشف المبكر عن سرطان الثدي».

  | Code | Group (English) | Goal | English rejection messages |
  |---|---|---|---|
  | `CHRONIC` | Chronic Diseases service group | السيطرة على الأمراض المزمنة | No qualifying personal or family history of chronic disease |
  | `ELDERLY` | Elderly service group | الرعاية الصحية لكبار السن | Outside the target age group (65–120 years) |
  | `MATERNAL` | Mother and Child Health Care service group | صحة الأم والطفل | Outside the target group (females aged 15–50); Not pregnant |
  | `BREAST_CANCER` | Women's Health Support service group | الكشف المبكر | Outside the target group (females aged 18 and over); Breast cancer early-detection criteria do not apply |

### Functional Requirements — Shared

- **FR-100** [BE]: Every creation and edit (with the changed fields and their old and new values, including rule versions), deactivation and reactivation of a group, and every revision created, MUST be audited, with the actor and time.
- **FR-101** [FE]: The screens MUST be available in Arabic (default, RTL) and English (LTR), WCAG 2.1 AA compliant, working on desktop, tablet and phone in Chrome, Firefox and Edge (constitution, Principle V); QA testing is in Arabic only.

### Key Entities

- **ServiceGroup**: code (unique, fixed), both names (Arabic unique), description, goals, identity (logo, primary colour, background colour), status.
- **EligibilityRule**: a stable identity, a rule number, numbered versions; each version with name, condition, rejection message, effective time.
- **ServiceGroupConfigurationRevision**: a number ordered per group, an immutable full copy of the rules, workflow stages, selected items and their conditions, the source, creator and time.
- **Service group goal**: an option of «أهداف مجموعات الخدمات» (007).

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: No group or rule breaking any rule of FR-011 – FR-014, FR-030 and FR-034 is saved (0 false accepts in a sample covering every rule).
- **SC-002**: Every visit is evaluated on the revision it was bound to when opened, and every case on its visit's revision (100% in a sample where the configuration is edited during the visit and the case).
- **SC-003**: The «اختبار القاعدة» result matches the real evaluation of the same data in a sample covering the three results (100%).
- **SC-004**: The System Admin adds a group with two rules, tests them and saves in under 10 minutes without help in at least 90% of the test group's attempts.
- **SC-005**: All attempts from accounts without `CanManageServiceGroups` are refused with no change (100%).

## Assumptions

- **Actor**: the System Admin only.
- **PROVISIONAL** — results are not a release reference: message texts not quoted from a source; the English names and messages of the initial data.
- **Dependencies**: [`001-condition-builder`](../001-condition-builder/spec.en.md); [`002-eligibility-form-fields`](../002-eligibility-form-fields/spec.en.md); [`007-lookup-option-conditions`](../007-lookup-option-conditions/spec.md); [`009-health-units`](../009-health-units/spec.en.md); [`013-eligibility-basic-readings`](../013-eligibility-basic-readings/spec.en.md) (showing eligibility); [`019-authorization`](../019-authorization/spec.en.md) (P-24); 027 (workflow stages); 020 – 025 (catalogs and lists).
- **Security**: a permission, an admin screen and file upload (the logo); Security Team sign-off on the "Authorization testing", "Business logic testing" and "File upload" categories of `docs/security/security-testing-checklist.md` is required before release (qa-standards §13.2).
- **Frontend**: the frontend constitution is still a draft (v2.0.0); it is followed as a baseline without being treated as approved.

## Gap Register

| ID | Topic | Status |
|---|---|---|
| G-01 | Saving together (DC) or «حفظ القاعدة» per rule (Figma) — FR-035 | Answered 2026-10-28: one save for the whole group (DC) |
| G-02 | Stopping or deleting a saved eligibility rule — FR-038 | Answered 2026-10-28: deletion allowed; kept in earlier revisions |
| G-03 | The RSD's «حالات مجموعات الخدمات» is 025's «حالات الطلب» (naming) | Answered 2026-10-28: named «حالات مجموعات الخدمات», 025 updated |
| G-04 | Group code format and the lengths of the name, rule name and rejection message — FR-011, FR-012, FR-030 | Answered 2026-10-28: as in 020; names ≤ 100; message ≤ 250 |
| G-05 | Initial «أهداف مجموعات الخدمات» values — FR-080 | Answered 2026-10-28: the five goals |
| G-06 | Seeding the four groups and their DC rules, and «نصائح غذائية للأم» in Figma — FR-090 | Answered 2026-10-28: the four with their rules; not «نصائح غذائية للأم» |
| G-07 | Enforcing "used by an active service group" in 020 – 024 (depends on stage items) | Routed to 027 |
| G-08 | Showing the eligibility result and rejection reasons in the eligibility form | Routed to DC §10 / 013 |
| G-09 | Performance and data-volume targets (eligibility evaluation per visit) | PENDING — QA Control Register |
| G-10 | Security sign-off categories — Assumptions | Settled: constitution and qa-standards §13.2 |
| G-11 | Test group size and method for SC-004 | Routed to test-plan §11 |
| G-12 | Who sees the rejection message — FR-030 | Settled: the nurse (Figma «رسالة الرفض للممرض») |
| G-13 | An English rejection message — FR-030 | Answered 2026-10-28: required |
| G-14 | Maximum number of rules per group — FR-030 | Answered 2026-10-28: no limit |
| G-15 | Safety of SVG logo content — FR-020 | Routed to Security Team sign-off and `/speckit-plan` |
| G-16 | Which groups a visit is bound to and evaluated against — FR-062 | Answered 2026-10-28 (revised): all active groups; those not available at the unit unselected and disabled with the «غير متاحة بالوحدة» hint |
| G-17 | The women's-health group name after RSD 0.4 — FR-090 | Answered 2026-10-28: «مجموعة خدمات دعم صحة المرأة» |
| G-18 | Chronic eligibility after Eligibility RSD 0.4 (BMI ≥ 30, average BP > 139/89) — FR-090 | Answered 2026-10-28: added with «OR»; the rejection message wording is for the client to confirm |
| G-19 | The pregnancy group name: «العناية بصحة الأم والجنين» (Eligibility RSD 0.4) versus «الأم والجنين» (Mother and Child RSD 0.4) — FR-090 | Answered 2026-10-28: «مجموعة خدمات العناية بصحة الأم والجنين» |
| G-20 | A rule on a field hidden by its visibility condition ("incomplete data" or not met) — FR-039 | Answered 2026-11-01: answered-empty, not met (POC M.39); documented in 026 only, 001 / 004 not changed |
| G-21 | Effect of POC M.40 (stage «الاقتراحات», the questionnaire «تفسير» field, dropping the 🚩 flag) on this specification | Settled 2026-11-02: no behaviour change here; band codes are frozen in the revision under FR-060; details in 027, 022 and 024 |
