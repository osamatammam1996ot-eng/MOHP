# Feature Specification: Service Group Workflow Stages (workflow tree, follow-up)

> **Language version: English (translation).** The Arabic [`spec.md`](spec.md) is the source of truth; on any difference, the Arabic version prevails. Both versions use the same IDs (FR-, SC-, G-), and every change must be applied to both files.

**Feature Branch**: `feature/authentication-authorization-specs`

**Created**: 2026-10-28

**Status**: Draft

**Input**: User description: a specification for a service group's «مراحل سير العمل» using the **POC workflow tree** (user decision 2026-10-28, completing [`026-service-groups`](../026-service-groups/spec.en.md)): stages of fixed kinds, the items of each stage with their applicability conditions, branches by condition or by choice, linked persons, a medication-administration stage, and a «المتابعة» tab with follow-up visit types, their paths and scheduling rules.

**Sources**:

| Source | Sections used | Status |
|---|---|---|
| Prototype (POC) `C:\Projects\MOHP\RuleBuilder`, branch `master` (`03de267`; first written from `b36b2c5`, updated 2026-11-01 from `1a31c4e` and 2026-11-02 from `03de267`) — **POC** | `WorkflowDefinition` (the path as a tree: «الفحوصات التكميلية», «خطوات طبيب الوحدة», «خطوة إحالة خارجية», «إعطاء الدواء», «تفرّع», «إنهاء»; items, their conditions and «حسب الإتاحة بالوحدة»; linked persons; fixed medicines; branches by condition or choice and «غير ذلك»; the parallel branch «أكثر من فرع معًا» and its join — plan M.38; stage «الاقتراحات» — plan M.40); `WorkflowDefinitionValidator` (validation messages); `FollowUpDefinition` (follow-up visit types, inherited or custom path, recurrence, activation condition, scheduling rules, «الطبيب يختار», reminder); the `WorkflowTreeEditor` and `FollowUpSettingsEditor` screens; the seed paths (chronic diseases, elderly, mother and child, women's health) | Behavioural reference — not an approved source |
| Dynamic configuration document — **DC** | §7 («مراحل سير العمل», scenario fifteen "applicability condition" per lab test and exam, «تخصص الطبيب», «اسم الحالة قبل بدء الفحص», «لا يمكن استخدام نفس الحالة في أكثر من مرحلة», «من يؤدي هذه المرحلة؟», «الصلاحيات المتاحة للطبيب»); §9 (revisions) | Draft |
| RSD "Eligibility Form and Portal Management" and the pathway RSDs | «مراحل سير العمل» (basic exams, primary-care-unit doctor steps, external referral steps); each service group's path (chronic diseases, elderly, mother and child, women's health); «جدولة متابعة فحص بالوحدة بناءً على عامل الخطورة»; «تأكيد إعطاء الدواء حسب جدول العلاج»; treating the husband | Draft — not yet approved by the client |
| Figma `VtUVDWHAZ0O6q3l40Wvq5N`, "Admin — Service Management" | "Laboratory Configuration Screen" (`726:429`: lab tests in groups = nurse steps, and «تعديل الشرط»), "Primary Care Workflow Screen" (`729:1113`), "External Referral Workflow Screen" (`729:1393`), "Visibility Rule Drawer" (`727:658`); built on the DC fixed stages, not the tree | Reviewed 2026-10-28 (partial) |
| [`020`](../020-lab-tests/spec.en.md) – [`026`](../026-service-groups/spec.en.md), [`018-authentication`](../018-authentication/spec.en.md) | 020 – 024: the catalogs (lab tests and kits, medicines, exams and kits, diagnoses, questionnaires); 025: «أنواع زيارات المتابعة», «أنواع الارتباط», «حالات مجموعات الخدمات»; 024: «فترات المتابعة»; 026: the group, the single save and revisions; 018: doctor specialties | Draft |

> Per Principle I of the constitution, this specification is derived exclusively from the sources above and the user's decisions. What the sources do not define is listed as **PROVISIONAL** and its results are not a release reference, and every unsettled value is **PENDING** in the Gap Register (constitution, Principle III).

**Tags**: **[FE]** frontend, **[BE]** server and business rules, **[FE+BE]** both, **the server being the final authority**.

## Clarifications

### Session 2026-10-28

- Q: Which workflow model? → A: **the POC workflow tree**, not the three fixed stages of the DC and Figma. (026; Input)
- Q: May the same request status be used in more than one stage? → A: **yes** (POC), unlike the DC. (G-02 — FR-021)
- Q: Where is it set which lab tests and exams a health unit cannot perform? → A: **on the health-unit screen**, with a «العناصر غير المتاحة بالوحدة» list (a change to 009). (G-03 — FR-042)
- Q: If a questionnaire band suggests a follow-up (nutrition 1 → every month) and the group's rule says "every 6 months", which applies? → A: **a "mandatory" rule wins; otherwise the band's follow-up is suggested to the doctor**, who confirms or changes it. (G-01 — FR-105) *(void since session 2026-11-03: no follow-up on a questionnaire band — G-52)*
- Q: Where do the seeded groups' paths come from? → A: **the RSDs first** (the business objective), then the POC for what the RSD does not define, and **any conflict is put to the user before it is resolved**. (G-07 — FR-120)
- Q: In what wording are the validation messages taken from the colloquial POC written? → A: **formal Arabic**, same meaning. (G-05 — FR-090)
- Q: Does a stage have a title? → A: **optional, up to 100 characters**, defaulting to the stage kind name. (G-04 — FR-014)
- Q: What is the range of "days before the date" in the reminder? → A: **1 – 30 days**. (G-06 — FR-106)
- Q: Which fields can scheduling-rule conditions use? → A: **all fields available in conditions and rules**. (G-16 — FR-103)
- Q: From which RSD versions are the paths built? → A: **the latest** in the main folder (Chronic Diseases 0.4, Geriatrics 0.5, Mother and Child 0.4, Women's Health Support 0.4, Eligibility 0.4), under the rule: prefer the POC when it meets the RSD's goal more flexibly, the RSD when the POC changes an explicit rule. (G-17)
- Q: The chronic random-glucose condition? → A: **the RSD**. (G-21) — HbA1c, lipids, kidney and dipstick? → A: **conditional as in the RSD**. (G-22)
- Q: The «HbA1c 10% أو أكثر» suggestion and the «باطنة» specialty for referral? → A: **yes (POC)**. (G-23) — A high reading for a previously referred patient? → A: **direct referral to the same hospital (RSD)**. (G-24)
- Q: The comprehensive-visit date for a chronic patient? → A: **HbA1c rules as recommendations (POC), the doctor picking 3 / 6 / 12**. (G-25)
- Q: Elderly: where the exams go, referral destinations and «كل شهر»? → A: **items in the doctor stage (RSD), four referral destinations (POC), «كل شهر» allowed (POC)**. (G-26, G-27, G-28)
- Q: Mother and child — syphilis with Hepatitis B / HIV? → A: **doctor then referral (RSD)**. (G-29)
- Q: Women's health — the exam and glucose condition, the clinic confirmation, and Malignant? → A: **the RSD condition; a normal clinic confirmation closes the case; MDT for all**. (G-30, G-31, G-32)
- Q: The new confirmatory BP visit in RSD 0.4? → A: **deferred**. (G-36) — The catalog items? → A: **seeded from the RSD**. (G-33)
- Q: How are the haemoglobin referral limits by sex represented? → A: **a condition-builder suggestion on the «طبيب باطنة» arm** (POC). (G-38)
- Q: The new RSDs' impact on 008 – 014, 020 and 022? → A: **reviewed and applied** after the user's answers (status «…للفحوصات التكميلية», reference range per RSD, kit reasons from the shared list, follow-up intervals 1 / 3 / 6 / 12, chronic eligibility with BMI and BP, the pregnancy group renamed). (G-37, G-39)

### Session 2026-11-01

- Q: The POC (M.38) lets a branch take several arms at once (e.g. referral to several destinations). Is it in this specification, and how far? → A: **yes — the branch type, its validation rules and the seed** here; running the tracks on cases (statuses, worklists, choosing several destinations) is routed to the stage specifications. (G-40 — FR-086, FR-087, FR-088)
- Q: A track that stalls, the maximum number of arms taken, and arms excluding each other? → A: **as in the POC**: no timeout (the case waits), no maximum, no arm excludes another — the doctor picks freely; to be confirmed by the client. (G-41 — FR-086)
- Q: Which seeded branches are parallel? → A: **the elderly «جهة الإحالة»** (FR-122); the mother-and-child «نتائج التحاليل» **stays sequential as in the RSD** (G-29), unlike the POC, which makes it parallel. (G-42 — FR-122, FR-123)
- Q: Must every sequence (the path, each arm of an ordinary branch, «غير ذلك», and what follows «تجميع») end with «إنهاء» for the save to be accepted? → A: **yes, the save is refused** («لا ينتهي المسار بـ «إنهاء»», or «… بعد التجميع»), in the editor and on the server — stricter than the POC, whose server does not refuse. (G-43 — FR-085)
- Q: Enabling «إحالة خارجية» for the nurse on «الفحوصات التكميلية» when a unit-doctor stage follows it: refuse or save with a warning? → A: **the save is refused** (FR-032 as written), unlike the POC, which only warns. (G-44 — FR-032)

### Session 2026-11-02

- Q: The POC (M.40) puts referral suggestions on the stage («الاقتراحات»: a condition, a reason and a choice arm of a later branch) instead of on the choice arm. Adopt it? → A: **yes, as in the POC**; the arm shows what suggests it, read-only. (G-47 — FR-023, FR-082)
- Q: Which destination does an abnormal ECG suggest in the elderly seed? → A: **«طبيب باطنة»**, as in the specification, unlike the POC («الطبيب المختص»). (G-48 — FR-122)
- Q: Add the POC's «ضعف سمع» → «الطبيب المختص» suggestion? → A: **no**; the Geriatrics RSD names no referral for the hearing exam. (G-49 — FR-122)
- Q: The POC drops the 🚩 «يحتاج إحالة» flag from exam answers and questionnaire bands and adds a «تفسير» field for each questionnaire with bands. Adopt it? → A: **yes, and 022 and 024 are updated**; what a flag referred on becomes stage «الاقتراحات» in the seeds, and the elderly suggestions read the questionnaires' «تفسير». (G-50 — FR-121 – FR-125)

### Session 2026-11-03

- Q: With follow-up dropped from questionnaire bands (024 G-57), how is the nutrition «1 ← after a month» follow-up defined in the elderly group? → A: **a "recommendation" scheduling rule of the group on the «تفسير الاستبيان التغذوي» field**, before the «مريض مزمن» rule. (G-52 — FR-105, FR-122)

## Scope

**In scope**:

1. The **«مراحل سير العمل»** tab in the group details (026): building the path as a tree of stages, branches and an end.
2. The items of each stage (lab tests, lab kits, exams, exam kits, questionnaires, field-registry questions) and each item's applicability condition.
3. Persons linked to the case (e.g. the husband) and fixed medicines in «إعطاء الدواء».
4. The **«المتابعة»** tab: the group's follow-up visit types, each type's path, scheduling rules and reminder.
5. Validating the path and showing it as a flowchart.
6. Defining "used by an active service group" for the 020 – 024 catalogs.

**Out of scope**:

- Running the path on cases: the nurse and doctor screens, recording results, branch decisions, confirming doses and sending reminders — future stage specifications; this feature defines the configuration only.
- Saving, revisions and eligibility rules — [`026-service-groups`](../026-service-groups/spec.en.md).
- Managing the «العناصر غير المتاحة بالوحدة» list itself — a change to [`009-health-units`](../009-health-units/spec.en.md) (G-03).

## Reference Design (Figma)

The Figma design (`726:429`, `729:1113`, `729:1393`, `727:658`) is built on the three fixed DC stages, so it informs the **look** of the elements, not the structure (G-12):

- **FIG-1 — Nurse-stage items** (`726:429`): items in groups ordered by dragging, «يظهر بشرط» and «تعديل الشرط» per item, and a «كما يظهر للممرض» preview.
- **FIG-2 — Applicability condition** (`727:658`): a «شرط ظهور: <item>» side panel with the condition builder, and «حذف الشرط (يُطلب دائمًا)».
- **FIG-3 — Doctor stage** (`729:1113`): «من يؤدي هذه المرحلة؟» (doctor / nurse on the doctor's behalf), «الصلاحيات المتاحة للطبيب» (diagnose / set treatment / external referral) and «حالة القضية أثناء هذه المرحلة».
- **FIG-4 — Referral step** (`729:1393`): «تخصص الطبيب», «من يرى هذه الخطوة», «حالة القضية قبل بدء الخطوة», and the items; and «إضافة خطوة».
- **FIG-5 — Tree and flowchart**: no Figma design; the POC (`WorkflowTreeEditor`, `WorkflowFlowchart`, `FollowUpSettingsEditor`) is followed until designed (G-12).

## User Scenarios & Testing *(mandatory)*

Single actor: **System Admin** (with `CanManageServiceGroups`).

### User Story 1 — Building the path from stages (Priority: P1)

I want to build the group's path from stages («الفحوصات التكميلية», «خطوات طبيب الوحدة», «خطوة إحالة خارجية», «إعطاء الدواء»), order them and close it with «إنهاء».

**Why P1**: no case moves without a path.

**Independent Test**: with a new group: add a nurse stage, then a doctor stage, then «إنهاء», save the group and check the revision.

**Acceptance Scenarios**:

1. **Given** the «مراحل سير العمل» tab, **When** I press «+» at a position in the path and pick a stage kind, **Then** the stage is added at that position and opened for editing. (FR-010 – FR-013)
2. **Given** a «الفحوصات التكميلية» stage with no items, **When** I save the group, **Then** the system refuses with «يجب أن تحتوي «الفحوصات التكميلية» على عنصر واحد على الأقل» (POC). (FR-031, FR-090)
3. **Given** a stage with no «حالة الطلب أثناء المرحلة», **When** I save, **Then** the system refuses with «حالة الطلب أثناء المرحلة مطلوبة» (POC). (FR-021)
4. **Given** «خطوات طبيب الوحدة» with no performer, **When** I save, **Then** the system refuses with «حدد من يؤدي المرحلة» (POC). (FR-050)
5. **Given** a valid path, **When** I save the group (026 FR-035), **Then** the path is saved with the group and a new revision is created (026 FR-060). (FR-003)

---

### User Story 2 — Stage items and their applicability conditions (Priority: P1)

I want to choose what each stage asks for (lab tests, kits, exams, questionnaires, field-registry questions), each item with a condition deciding for whom it is asked.

**Why P1**: it decides what the nurse and doctor do for each beneficiary (DC §7 scenario fifteen).

**Independent Test**: with a nurse stage: add a lab test with a condition, another without, a questionnaire and a read-only question, and check the save.

**Acceptance Scenarios**:

1. **Given** a nurse stage, **When** I add «السكر التراكمي (HbA1c)» with the condition "age between 35 and 64", **Then** the item is saved with its condition and asked only for those the condition holds for; an item with no condition is always asked. (FR-040, FR-041)
2. **Given** the same item added twice in the same stage, **When** I save, **Then** the system refuses with «العنصر «<code>» مكرر في المرحلة نفسها» (POC). (FR-043)
3. **Given** a field-registry question in the stage, **When** I pick its mode, **Then** it is «إدخال · إلزامي», «إدخال · اختياري» or «للقراءة». (FR-044)
4. **Given** «حسب الإتاحة بالوحدة» on a questionnaire, **When** I save, **Then** the system refuses with ««حسب الإتاحة بالوحدة» للتحاليل والفحوصات وأطقمها فقط» (POC). (FR-042)

---

### User Story 3 — Branches (Priority: P1)

I want to split the path with a branch: an arm taken automatically when its condition holds, or an arm the doctor chooses (e.g. the referral destination), suggested by an earlier stage's «الاقتراحات» with a reason the doctor sees, and a mandatory «غير ذلك»; and to make a branch parallel when several arms must be taken together (e.g. referral to several destinations).

**Why P1**: real paths branch (e.g. referral to different destinations in the elderly group, high readings in the monthly follow-up).

**Independent Test**: add a branch with two choice arms and «غير ذلك» and a suggestion for them on the stage before, make it parallel, and check the validation messages.

**Acceptance Scenarios**:

1. **Given** a «جهة الإحالة» branch after «خطوات طبيب الوحدة», **When** I add a choice arm «الطبيب المختص» and «غير ذلك» named «لا يحتاج إحالة», and on «خطوات طبيب الوحدة» a suggestion with the condition «تفسير التقييم المعرفي لكبار السن 4AT» = «اشتباه اضطراب حاد في الوعي» and the reason «اشتباه اضطراب حاد في الوعي (4AT أكبر من 4)» pointing to «الطبيب المختص», **Then** the path is saved and the suggestion shows on the arm read-only. (FR-023, FR-080 – FR-083)
2. **Given** a choice arm with no name, or a condition arm with no condition, **When** I save, **Then** the system refuses with «اسم الفرع مطلوب لأنه يظهر للطبيب اختيارًا» or «يجب أن يكون للفرع شرط» (POC). (FR-081)
3. **Given** a branch with no arm other than «غير ذلك», **When** I save, **Then** the system refuses with «يجب أن يحتوي التفرّع على فرع واحد على الأقل غير «غير ذلك»» (POC). (FR-080)
4. **Given** a stage after an ordinary branch at the same level, **When** I save, **Then** the system refuses with «توضع المراحل التالية للتفرّع داخل فروعه أو «غير ذلك»» (POC). (FR-084)
5. **Given** the «جهة الإحالة» branch with four choice arms, each ending in «خطوة إحالة خارجية» then «إنهاء», **When** I set «نوع التفرّع» to «أكثر من فرع معًا», **Then** each arm's «إنهاء» is replaced by one «إنهاء» after «تجميع», and the branch is saved (POC). (FR-086, FR-088)
6. **Given** a parallel branch with a single arm other than «غير ذلك», **When** I save, **Then** the system refuses with «يجب أن يحتوي التفرّع المتوازي على فرعين على الأقل» (POC). (FR-087)
7. **Given** «إنهاء» or another parallel branch inside a parallel arm, or a choice branch right after «تجميع», **When** I save, **Then** the system refuses with «لا يُسمح بـ «إنهاء» داخل فرع متوازٍ — يستمر الطلب بعد التجميع», «لا يجوز تفرّع متوازٍ داخل فرع متوازٍ» or «هذا الاختيار يلي التجميع مباشرة — ضع قبله مرحلة يُتخذ فيها القرار» (POC). (FR-087)
8. **Given** a stage suggestion with no reason, or pointing to an arm before the stage or one that was removed, **When** I save, **Then** the system refuses with «سبب الاقتراح مطلوب» or «يجب أن يشير الاقتراح إلى فرع «اختيار» بعد هذه المرحلة» (POC). (FR-023)

---

### User Story 4 — Linked persons and fixed medicines (Priority: P2)

I want a stage to require linking a person to the case (e.g. the husband) with items recorded on their file, and to put fixed medicines in «إعطاء الدواء» given without a prescription.

**Why P2**: the mother-and-child path needs them (treating the husband, the doses), and other paths work without them.

**Independent Test**: add a «الزوج» link with a condition and items, and an «إعطاء الدواء» stage with a fixed medicine and its doses.

**Acceptance Scenarios**:

1. **Given** a nurse stage, **When** I add a required «الزوج» link with a condition and items for them, **Then** it is saved, and its items are recorded on the linked person's file at run time. (FR-075)
2. **Given** a link with no type, **When** I save, **Then** the system refuses with «نوع الارتباط مطلوب» (POC). (FR-075)
3. **Given** an «إعطاء الدواء» stage, **When** I add the fixed medicine «حمض الفوليك» with a dose every 30 days × 3, **Then** it is saved; a dose count or interval below 1 is refused with «يجب أن يكون عدد الجرعات والفاصل بالأيام عددين موجبين» (POC). (FR-070)
4. **Given** a fixed medicine in a stage other than «إعطاء الدواء», **When** I save, **Then** the system refuses with «الأدوية الثابتة في مرحلة «إعطاء الدواء» فقط» (POC). (FR-070)

---

### User Story 5 — Follow-up (Priority: P2)

I want to set the group's follow-up visit types, each type's path, when it applies to a beneficiary, the rules for the next visit date, and the reminder for it.

**Why P2**: the RSD needs it («جدولة متابعة فحص بالوحدة», «زيارة دورية شهرية»), but the first visit works without it.

**Independent Test**: add a recurring «زيارة دورية شهرية» type with the activation condition «مريض مزمن», two rules and «الطبيب يختار».

**Acceptance Scenarios**:

1. **Given** the «المتابعة» tab, **When** I add the «زيارة دورية شهرية» type, **Then** it is added with an "inherited" path from «مراحل سير العمل» and can be switched to "custom". (FR-100, FR-101)
2. **Given** a recurring type, **When** I add the rule "HbA1c ≥ 8 → every 3 months (mandatory)" and «غير ذلك ← الطبيب يختار من: كل 3 أشهر، كل 6 أشهر، كل سنة», **Then** they are saved, evaluated when the case closes, and the first rule that holds is applied. (FR-103, FR-104)
3. **Given** a "custom" path copied and then «مراحل سير العمل» edited, **When** I open the type, **Then** «⚠ عُدِّلت مراحل سير العمل بعد نسخ هذا المسار» is shown (POC). (FR-101)
4. **Given** an "on attendance" «زيارة عارضة» type with an activation condition, **When** the condition holds for the beneficiary, **Then** it is offered to the nurse as an available visit type (in the stage specifications). (FR-102)

---

### User Story 6 — Validation and flowchart (Priority: P2)

I want to see the path as a flowchart, and know every problem in it before saving.

**Why P2**: the tree is complex, and a mistake in it stops cases.

**Independent Test**: a path with two problems, then fixing them.

**Acceptance Scenarios**:

1. **Given** a valid path, **When** I open it, **Then** «✓ المسار سليم — لا توجد مشكلات» is shown (POC). (FR-090)
2. **Given** stages after «إنهاء», **When** I save, **Then** the system refuses with «توجد مراحل بعد «إنهاء» لن يُوصل إليها» (POC). (FR-085)
3. **Given** the flowchart, **When** I click a stage or arm, **Then** its details are shown for viewing only, editing being in «مراحل سير العمل» (POC). (FR-015)
4. **Given** an arm whose last node is a stage, or a parallel branch with nothing after «تجميع», **When** I save, **Then** the system refuses with «لا ينتهي المسار بـ «إنهاء»» or «لا ينتهي المسار بـ «إنهاء» بعد التجميع» (G-43). (FR-085)

---

### Edge Cases

- **Deactivating a catalog item used in an active group's path**: refused (FR-110).
- **Editing the path while cases are ongoing**: they continue on their revision (026 FR-063).
- **A path with no stage anyone performs**: refused «لا توجد في المسار مرحلة يؤديها أحد» (FR-090).
- **The same request status in more than one stage**: allowed (FR-021 — G-02).
- **A follow-up a questionnaire interpretation (024) calls for and another follow-up rule in the same case**: both are scheduling rules here, and the first rule that holds in order applies (FR-103, FR-105 — G-52).
- **A choice branch after «الفحوصات التكميلية» with no doctor stage**: requires enabling «إحالة خارجية» for the nurse in that stage (FR-032).
- **A parallel branch where no arm holds and none is chosen**: «غير ذلك» is taken, then the path continues after «تجميع» (FR-086).
- **A parallel branch where only one arm is taken**: it is walked like an ordinary branch, with no separate track (FR-086).
- **A suggestion pointing to an arm that was removed or moved before its stage**: the save is refused with «يجب أن يشير الاقتراح إلى فرع «اختيار» بعد هذه المرحلة» (FR-023).

## Requirements *(mandatory)*

### Functional Requirements — Tab and saving

- **FR-001** [FE]: The **«مراحل سير العمل»** and **«المتابعة»** tabs MUST appear in the service-group details (026 FIG-2).
- **FR-002** [BE]: The tabs are protected by the `CanManageServiceGroups` permission (019 P-24).
- **FR-003** [BE]: The path, the follow-up paths and their rules are saved with the group in the **single save** (026 FR-035), and every change in them creates a new revision (026 FR-060); a save with no change produces «تم الحفظ (لا تغيير يستدعي إصدارًا جديدًا)» (POC).
- **FR-004** [FE]: Moving between tabs does not lose changes, and «توجد تعديلات لم تُحفظ بعد» is shown when there are some (POC).

### Functional Requirements — Tree structure (Story 1)

- **FR-010** [FE+BE]: The **path** is an ordered sequence of nodes; a node is a **stage**, a **branch** or an **end**; each arm of a branch is its own sequence under the same rules (POC).
- **FR-011** [BE]: Node kinds are **fixed in the system** (a new kind is a development request): «الفحوصات التكميلية» (the nurse), «خطوات طبيب الوحدة», «خطوة إحالة خارجية», «إعطاء الدواء», «تفرّع», «إنهاء» (POC).
- **FR-012** [BE]: Each node has a **stable key**, unique in the path and unchanged by edits, which a case stores to know its position (POC); messages «المرحلة بلا مفتاح» and «المفتاح «<key>» مكرر».
- **FR-013** [FE]: «+» at any position adds a node there; clicking a node opens it for editing; a stage can be moved up and down and deleted, and deleting a branch deletes it «وكل ما تحته» after confirmation (POC).
- **FR-014** [FE]: Each stage has an optional **title** up to 100 characters shown in the path and to the nurse and doctor, defaulting to the stage kind name (G-04).
- **FR-015** [FE]: The path is also shown as a **flowchart**, view only, and clicking a stage or arm shows its details (POC `WorkflowFlowchart`).

### Functional Requirements — Stages in general

- **FR-020** [FE+BE]: Each actionable stage («الفحوصات التكميلية», «خطوات طبيب الوحدة», «خطوة إحالة خارجية», «إعطاء الدواء») has optional **items** (FR-040) and **linked persons** (FR-075) (except what FR-031 requires).
- **FR-021** [FE+BE]: Each stage has a required **«حالة الطلب أثناء المرحلة»** from the active «حالات مجموعات الخدمات» (025 FR-040), with «حالة الطلب أثناء المرحلة مطلوبة» (POC); **the same status may be used in more than one stage** (e.g. «بانتظار التشخيص النهائي» in several referral arms), unlike DC §7, since a case stores its stage key (FR-012) (POC — G-02).
- **FR-022** [FE+BE]: "Show basic readings" yes / no per stage (POC).
- **FR-023** [FE+BE]: Every actionable stage has optional **«الاقتراحات»**, each with: a required **condition** in the condition builder (001) on every field available to conditions and rules (including the questionnaires' «تفسير» — 024, exam and lab results, registration answers, or several together) («يجب أن يكون للاقتراح شرط»); the **reason** the doctor sees («سبب الاقتراح مطلوب»); and the **suggested arm**: a «اختيار الطبيب» arm of a branch **that comes after the stage** (in the rest of its sequence, or after the branch that contains it, including after «تجميع»); anything else — an arm before the stage, in another arm, or one that was removed — is refused with «يجب أن يشير الاقتراح إلى فرع «اختيار» بعد هذه المرحلة», and suggestions on a non-stage node with «الاقتراحات على المراحل فقط»; when no choice branch follows the stage, «لا يوجد بعد هذه المرحلة تفرّع «اختيار الطبيب» — أضف تفرّعًا لتتمكن من الاقتراح» is shown. A suggestion is evaluated **when the case reaches that branch**, on everything the case has by then; the stage is only where it is written (FR-082) (POC M.40, reworded in formal Arabic — G-05, G-47).

### Functional Requirements — «الفحوصات التكميلية» (the nurse)

- **FR-030** [BE]: Performed by the **nurse** (POC; RSD «يقوم بها الممرض»).
- **FR-031** [FE+BE]: It MUST contain **at least one item**, with «يجب أن تحتوي «الفحوصات التكميلية» على عنصر واحد على الأقل» (POC; RSD «يجب إضافة على الأقل تحليل او فحص سريري واحد»).
- **FR-032** [FE+BE]: «صلاحيات التمريض: إحالة خارجية» is optional, for paths with no unit doctor (e.g. an abnormal breast exam): the nurse's choice at the next branch is then the referral decision; a choice branch after the stage without this permission and without a doctor stage between them is refused with «هذا الاختيار يلي «الفحوصات التكميلية» — فعّل صلاحية «إحالة خارجية» للتمريض في هذه المرحلة», and enabling it with a doctor stage after it is refused with ««الفحوصات التكميلية»: تليها مرحلة طبيب وحدة، وهو من يقرر الإحالة» (POC; refused, not the POC warning — G-44).

### Functional Requirements — Items (Story 2)

- **FR-040** [FE+BE]: Item types: **lab tests** and **lab kits** (020), **exams** and **exam kits** (022), **questionnaires** (024), **stage questions** from the field registry (002); only **active** items are offered; in an order set by the System Admin (POC; FIG-1).
- **FR-041** [FE+BE]: Each item has an optional **«شرط التطبيق»** built with the condition builder (001; FIG-2); empty = always asked, otherwise asked only for those the condition holds for (DC §7 scenario fifteen); the item's own visibility condition in its catalog (007) must hold too.
- **FR-042** [FE+BE]: **«حسب الإتاحة بالوحدة»** is optional for lab tests, exams and their kits only: an item unavailable at the case's unit is shown locked with «غير متاح بالوحدة» (POC); the availability source is a **«العناصر غير المتاحة بالوحدة»** list (lab tests, exams, kits) on the health-unit screen (a change to [`009-health-units`](../009-health-units/spec.en.md); POC — G-03); on anything else it is refused with ««حسب الإتاحة بالوحدة» للتحاليل والفحوصات وأطقمها فقط».
- **FR-043** [BE]: An item does not repeat within the same stage, with «العنصر «<code>» مكرر في المرحلة نفسها» (POC).
- **FR-044** [FE+BE]: A stage question has a **mode**: «إدخال · إلزامي», «إدخال · اختياري» or «للقراءة» (shown with its current value, not editable); anything else is refused with «طريقة الحقل في المرحلة غير صالحة» (POC).

### Functional Requirements — «خطوات طبيب الوحدة»

- **FR-050** [FE+BE]: **«من يؤدي المرحلة؟»** is required, one or more of: «الطبيب», «التمريض نيابة عن الطبيب» (RSD; FIG-3), with «حدد من يؤدي المرحلة» (POC).
- **FR-051** [FE+BE]: The available **«الصلاحيات»**: «تشخيص», «تحديد العلاج», «إحالة خارجية» (RSD; FIG-3); diagnosis and treatment per person (the beneficiary and each linked person), from their diagnoses' treatment plans (023) (POC).

### Functional Requirements — «خطوة إحالة خارجية»

- **FR-060** [FE+BE]: **«من يرى الخطوة»** is required (default «الطبيب بالمستشفى الخارجي»), with «حدد من يرى الخطوة» (POC; FIG-4); **«تخصص الطبيب»** is a single choice from doctor specialties (018) (RSD; FIG-4); «الإحالة لنفس المستشفى المحال إليها سابقاً» yes / no (POC); the external-hospital doctor decisions form as in the RSD (in the stage specifications).

### Functional Requirements — «إعطاء الدواء» (Story 4)

- **FR-070** [FE+BE]: The nurse confirms the due doses in it; it can carry **«أدوية ثابتة (من غير روشتة)»**: an active medicine (021), an optional dose text up to 100 characters, and a **dose count** and **interval in days**, whole numbers ≥ 1 (with «يجب أن يكون عدد الجرعات والفاصل بالأيام عددين موجبين»), its schedule starting when the case reaches the stage; a medicine does not repeat in the stage («الدواء مكرر في المرحلة نفسها»), «اختر الدواء» when missing; fixed medicines in any other stage are refused («الأدوية الثابتة في مرحلة «إعطاء الدواء» فقط») (POC).

### Functional Requirements — Linked persons (Story 4)

- **FR-075** [FE+BE]: Each stage has zero or more links; each link has: a required **relation type** from «أنواع الارتباط» (025 FR-030) («نوع الارتباط مطلوب»), an optional **request condition** (empty = if already linked by an earlier stage), **required before saving the stage** yes / no, and **items** recorded on the linked person's file; it is linked in the first stage whose condition holds, and later stages reuse it (POC); the search and linking themselves are in the stage specifications.

### Functional Requirements — Branch (Story 3)

- **FR-080** [FE+BE]: A branch has **arms** tried in order, and an **«غير ذلك»** arm that is always mandatory; there must be **at least one arm other than «غير ذلك»** («يجب أن يحتوي التفرّع على فرع واحد على الأقل غير «غير ذلك»») (POC).
- **FR-081** [FE+BE]: An arm's kind: **«شرط على البيانات»** (taken when its condition holds; the condition required — «يجب أن يكون للفرع شرط») or **«اختيار الطبيب»** (chosen by the previous stage's doctor, or its nurse per FR-032, from the arm names; the name required — «اسم الفرع مطلوب لأنه يظهر للطبيب اختيارًا»); the arm key is stable and unique («مفتاح الفرع ناقص أو مكرر») (POC).
- **FR-082** [FE+BE]: A choice arm carries no suggestion reasons; the editor shows on it, **read-only**, the stage suggestions pointing to it (FR-023) under «يُقترح عندما — للعرض فقط، ويُعدَّل من «الاقتراحات» على المرحلة» with a link to their stage, or «لا توجد اقتراحات لهذا الفرع»; at run time an arm whose suggestion holds is shown suggested to the doctor with the reason and is not picked automatically, and in a parallel branch every suggested arm is shown suggested (POC M.40 — G-47).
- **FR-083** [FE+BE]: The **«غير ذلك»** name is required when the arms are choices («اسم «غير ذلك» مطلوب لأنه يظهر للطبيب اختيارًا (مثل «لا يحتاج إحالة»)»), optional otherwise (POC).
- **FR-084** [BE]: No stages go after an **ordinary** branch at the same level (the parallel branch is the exception — FR-087); they go inside its arms or «غير ذلك» («توضع المراحل التالية للتفرّع داخل فروعه أو «غير ذلك»») (POC).
- **FR-085** [BE]: **«إنهاء»** closes the case; no stages after it («توجد مراحل بعد «إنهاء» لن يُوصل إليها»); the classification is computed from «تصنيفات المستفيد» (025), and appointments from «المتابعة» (POC). **Every sequence must end**: the path, each arm of an ordinary branch and its «غير ذلك» end with «إنهاء» or an ordinary branch, and a parallel branch is followed by stages ending with «إنهاء» (the sequences inside a parallel arm end with the arm — FR-087); otherwise the save is refused with «لا ينتهي المسار بـ «إنهاء»», or «لا ينتهي المسار بـ «إنهاء» بعد التجميع» after a parallel branch (G-43).
- **FR-086** [FE+BE]: Each branch has a **«نوع التفرّع»**: **«فرع واحد»** (the default: the first arm whose condition holds, or the one chosen) or **«أكثر من فرع معًا»** (parallel): every condition arm that holds and every choice arm chosen are taken together, for condition and choice arms alike; «غير ذلك» is taken only when no arm is, and is chosen only on its own; a single arm taken is walked as usual; two or more become **tracks** of the same case, and the path continues after the branch (**«تجميع»**) once every track is done. What follows «تجميع» is the System Admin's configuration («إنهاء» closes the case; a unit-doctor stage after it brings the case back to the doctor once every referral is done); a stalled track keeps the case waiting with no timeout; there is no maximum number of arms taken and no arm excludes another — the doctor picks freely (POC M.38 — G-41, to be confirmed by the client). Running the tracks is in the stage specifications (G-08).
- **FR-087** [BE]: Rules of the parallel branch: **at least two arms** other than «غير ذلك» («يجب أن يحتوي التفرّع المتوازي على فرعين على الأقل»); **no «إنهاء» inside a parallel arm or its «غير ذلك»** («لا يُسمح بـ «إنهاء» داخل فرع متوازٍ — يستمر الطلب بعد التجميع»); **no parallel branch inside a parallel arm or its «غير ذلك»** («لا يجوز تفرّع متوازٍ داخل فرع متوازٍ»); stages may follow it at the same level, and the path must still end in «إنهاء» after «تجميع» (FR-085); a choice branch **right after «تجميع»** is refused, since no stage precedes it to make the decision («هذا الاختيار يلي التجميع مباشرة — ضع قبله مرحلة يُتخذ فيها القرار») (POC, reworded in formal Arabic — G-05).
- **FR-088** [FE]: Switching «نوع التفرّع» in the editor reshapes the path so it stays valid: to «أكثر من فرع معًا», each arm's «إنهاء» is removed and a single «إنهاء» is added after «تجميع»; back to «فرع واحد», what followed «تجميع» is copied to the end of each arm and of «غير ذلك»; the flowchart (FR-015) draws the parallel branch splitting on a bar and its arms meeting again at «تجميع — يستمر الطلب عندما تنتهي كل الفروع» (POC).

### Functional Requirements — Validation (Story 6)

- **FR-090** [FE+BE]: The system validates the path on every edit and on save, and shows all problems together under «لم يُحفظ المسار:» linked to their positions, or «✓ المسار سليم — لا توجد مشكلات»; saving is refused on any problem; rules beyond the above include: «يجب أن يحتوي المسار على مرحلة واحدة على الأقل», «لا توجد في المسار مرحلة يؤديها أحد», «صيغة الشرط غير صحيحة: …», «نوع مرحلة غير معروف», «نوع عنصر غير معروف», «نوع الفرع غير معروف», «العنصر بلا كود» (POC, reworded in formal Arabic — G-05).

### Functional Requirements — Follow-up (Story 5)

- **FR-100** [FE+BE]: The **«المتابعة»** tab: the group's **follow-up visit types** from «أنواع زيارات المتابعة» (025 FR-010), each type once, **in priority order**, each with "enabled" yes / no («· متوقف»); with no types «لا متابعة في هذه المجموعة — أضف نوعًا من الأعلى» (POC).
- **FR-101** [FE+BE]: **The type's path**: "inherited" (walks «مراحل سير العمل» as they are, read only) or "custom" (a copy of them edited on its own under FR-010 – FR-090); switching to "inherited" clears the customisation after confirmation («سيُمسح التخصيص ويعود المسار مطابقًا لمراحل سير العمل.»), and "copy again" replaces the custom path after confirmation; if «مراحل سير العمل» were edited after the copy, «⚠ عُدِّلت مراحل سير العمل بعد نسخ هذا المسار» is shown (POC).
- **FR-102** [FE+BE]: **Recurrence**: "recurring" (with a date set by the scheduling rules) or "on attendance" (no date, offered to the nurse when its activation condition holds); an optional **activation condition** (empty = for everyone followed in the group), e.g. «زيارة دورية شهرية» for those classified «مريض مزمن» (025) (POC).
- **FR-103** [FE+BE]: A recurring type has ordered **scheduling rules**: a condition built with the condition builder on **all fields available in conditions and rules** (including lab-test, exam and questionnaire results, classifications and multiple-choice fields — G-16), an action ("next visit after" an interval from «فترات المتابعة» (024 FR-034) / "keep the date as it is" / "stop the follow-up"), and "mandatory" (the doctor cannot change it) or "recommendation" (the doctor's choice wins when recorded); evaluated **when the case closes**, the first rule that holds being applied, and the date counted from the visit itself (POC).
- **FR-104** [FE+BE]: The rules' **«غير ذلك»**: «الطبيب يختار» from allowed intervals set by the System Admin, or a "fixed interval" (POC).
- **FR-105** [BE]: **No follow-up on questionnaire bands** (024 FR-033 — 024 G-57): the follow-up a questionnaire interpretation leads to is a **scheduling rule** here (FR-103) whose condition is on the «تفسير <questionnaire>» field (024 FR-042), "mandatory" or "recommendation" under FR-103, the first rule that holds applying; e.g. «تفسير الاستبيان التغذوي = معرّض لسوء التغذية ← كل شهر» before «مريض مزمن ← بعد 6 أشهر» (Geriatrics RSD: nutrition «1 : يحتاج للمتابعة بعد شهر» versus «المتابعة الدورية كل 6 أشهر») (POC M.40 — G-52; replaces G-01).
- **FR-106** [FE+BE]: A **reminder** per type: enabled yes / no, "days before the date" (a whole number 1 – 30 — G-06), "channel" (SMS / WhatsApp / call), and "reminder text" with the placeholders {الاسم} {الموعد} {الوحدة}, default «مرحبًا {الاسم}، نذكّرك بموعد متابعتك في {الوحدة} يوم {الموعد}.»; configuration only ("sending not enabled yet"), the sending being in the messaging specifications (POC; RSD «إرسال رسالة نصية قبل أسبوع من موعد المتابعة»).

### Functional Requirements — Usage in catalogs

- **FR-110** [BE]: A catalog item (lab test, lab kit, exam, exam kit, questionnaire, fixed medicine, diagnosis in a plan) is **"used by an active service group"** when it appears in the base path or the custom path of an enabled follow-up type of an **active** group; this applies the deactivation-refusal rules pending in 020 – 024 with their messages (closes 020 G-03 / 021 G-02 / 022 G-03 / 023 G-06 / 024 G-06); deactivating a request status used in a stage is refused (025 FR-041).
- **FR-111** [BE]: A deactivation refused by FR-110 names the groups using the item (provisional).

### Functional Requirements — Initial data

- **FR-120** [BE]: The paths of the four groups seeded in 026 FR-090 are seeded **from the RSDs first** — the latest versions in `C:\Projects\MOHP\Repo\docs\RSDs` (Chronic Diseases 0.4, Geriatrics 0.5, Mother and Child 0.4, Women's Health Support 0.4, Eligibility 0.4) — using the POC for structure and for what the RSD does not define, every conflict settled with the user (G-07, G-17); the detail in FR-121 – FR-125.
- **FR-121** [BE]: **The «مجموعة خدمات الأمراض المزمنة» path** (Chronic Diseases RSD 0.4):
  - Stage 1 «الفحوصات التكميلية» (the nurse; «بانتظار المتابعة مع التمريض للفحوصات التكميلية»; basic readings shown): **random glucose** when: age 35 – 64, or age 18 – 34 with (a personal chronic-disease history, or BMI ≥ 30, or average BP > 139/89) (G-21); **HbA1c**, the **lipid kit** (Total Chol. / Trig / HDL / LDL), the **kidney kit** (creatinine + eGFR) and **dipstick** when: a chronic-disease history, or BMI ≥ 30, or average BP > 139/89, or random glucose ≥ 200 (G-22).
  - Stage 2 «خطوات طبيب الوحدة» (the doctor or the nurse on the doctor's behalf; «بانتظار مراجعة الطبيب»; diagnose, set treatment, external referral).
  - Branch «قرار الإحالة» (doctor's choice): «يحتاج إحالة خارجية» (suggested by «خطوات طبيب الوحدة»: «HbA1c 10% أو أكثر» — POC G-23, FR-023) → «خطوة إحالة خارجية» (external-hospital doctor, specialty «باطنة» (POC — G-23), «بانتظار التشخيص النهائي») → end; «غير ذلك» = «لا يحتاج إحالة خارجية» → end.
  - Follow-up: (a) a recurring «زيارة دورية روتينية شاملة» on the inherited path, with the recommendation rules «HbA1c ≥ 8 ← بعد 3 أشهر» and «HbA1c ≥ 7 ← بعد 6 أشهر» (POC; values to be confirmed by the client — G-25), and «غير ذلك ← الطبيب يختار» from 3 / 6 / 12 months (RSD 0.4); (b) a recurring «زيارة دورية شهرية», activated for «مريض مزمن» (025), with «غير ذلك ← مدة ثابتة كل شهر» the doctor cannot change (RSD 0.4), on a custom path: a nurse stage (random glucose under the same condition, and BP readings) → a condition branch «قراءة مرتفعة» (random glucose ≥ 200 or BP > 139/89): arm «سبقت إحالته» → «خطوة إحالة خارجية» to the same hospital as before → end; arm «لم تسبق إحالته» → «خطوات طبيب الوحدة» → the «قرار الإحالة» branch as above; «غير ذلك» (normal readings) → end (RSD — G-24); a reminder a week before (RSD).
- **FR-122** [BE]: **The «مجموعة خدمات كبار السن» path** (Geriatrics RSD 0.5):
  - Stage 1 «الفحوصات التكميلية» (the nurse): random glucose, HbA1c, the lipid kit, the kidney kit, dipstick — no conditions (RSD).
  - Stage 2 «خطوات طبيب الوحدة» (the doctor or the nurse on the doctor's behalf; all three permissions) with items **«حسب الإتاحة بالوحدة»**: haemoglobin, the ECG kit (Heart Rate, Rhythm, ECG Signs of Ischemic Heart Disease, BBB, LVH), clinical exam, right-eye and left-eye vision, dental, sonar, hearing; and the questionnaires: «سؤال درجة الوعي», «4AT» when the awareness answer = «لا», «Mini-Cog», «التغذوي», «مؤشر كارتز» (RSD 0.5 — G-26).
  - Branch «جهة الإحالة» (doctor's choice, **parallel**: the doctor may pick any number of destinations, each a track; POC — G-27, G-42): «الطبيب المختص», «الأمانة العامة للصحة النفسية», «أخصائي تغذية», «طبيب باطنة»; each arm → «خطوة إحالة خارجية» («بانتظار التشخيص النهائي»); «غير ذلك» = «لا يحتاج إحالة»; then «إنهاء» after «تجميع».
  - **«الاقتراحات» on «خطوات طبيب الوحدة»** (FR-023 — G-47, G-50), reading the questionnaires' «تفسير» (024), not their totals: «الطبيب المختص» when «تفسير التقييم المعرفي لكبار السن 4AT» = «اشتباه اضطراب حاد في الوعي» (reason «اشتباه اضطراب حاد في الوعي (4AT أكبر من 4)»); «الأمانة العامة للصحة النفسية» when «تفسير اختبار Mini-Cog» = «ضعف إدراكي» («اشتباه ضعف إدراكي (Mini-Cog أقل من 3)»); «أخصائي تغذية» when «تفسير الاستبيان التغذوي» = «سوء تغذية» («سوء تغذية (الاستبيان التغذوي 2 أو أكثر)»); «طبيب باطنة» when vision is 6/12 or 6/18 in either eye («ضعف في النظر (6/12 أو 6/18)»), or haemoglobin is outside the Geriatrics RSD 0.5 limits by sex (female: below 9 or above 16; male: below 9 or above 18 — G-38), or the ECG is abnormal (Rhythm = Irregular, or ECG Signs = ST-depression or T-inversion, or BBB = LBBB; «رسم القلب غير طبيعي» — G-48); no suggestion for the hearing exam (G-49).
  - Follow-up: a recurring «زيارة دورية روتينية شاملة» on the inherited path, with two recommendation rules in order: «تفسير الاستبيان التغذوي = معرّض لسوء التغذية ← كل شهر» (RSD nutrition «1 : يحتاج للمتابعة بعد شهر»; FR-105 — G-52), then «مريض مزمن ← بعد 6 أشهر» (RSD «المتابعة الدورية كل 6 أشهر للمرضى المزمنين»), and «غير ذلك ← الطبيب يختار» from monthly / 3 / 6 / 12 months (G-28); a reminder a week before.
- **FR-123** [BE]: **The «مجموعة خدمات العناية بصحة الأم والجنين» path** (name — 026 G-19) (Mother and Child RSD 0.4):
  - Stage 1 «الفحوصات التكميلية» (the nurse; «صلاحيات التمريض: إحالة خارجية»): the **mother-and-child kit** (Hepatitis B, HIV, syphilis — one batch and one not-performed reason, RSD 0.4); the question «تأكيد إبلاغ الوقائي بموعد الولادة المتوقع» when Hepatitis B is reactive; and a required «الزوج» link when syphilis is reactive, with the item «اختبار الزهري» on his file.
  - Branch «نتائج التحاليل» (condition): (1) «الزهري متفاعل» → «خطوات طبيب الوحدة» (items for the wife and, through the link, the husband: «اختبار الحساسية من البنسلين»; treatment from the «الزهري» diagnosis plans in 023: long-acting penicillin, non-long-acting penicillin, or erythromycin on allergy or when penicillin is out of stock) → a condition branch «علاج يُعطى بالوحدة» → «إعطاء الدواء» («بانتظار الانتهاء من خطة العلاج»); then in both cases a condition branch «Hepatitis B أو HIV متفاعل» → «خطوة إحالة خارجية» → end, else end (RSD — G-29; an ordinary branch, not parallel as in the POC — G-42); (2) «Hepatitis B أو HIV متفاعل» (without syphilis) → «خطوة إحالة خارجية» (external-hospital doctor, «بانتظار التشخيص النهائي»; items: confirming the disease, treatment, «تأكيد إعطاء تطعيم للمخالطين», «تأكيد إعطاء الطفل جرعة immunoglobulin») → end; «غير ذلك» (all tests non-reactive) → end.
  - Follow-up: an "on attendance" «زيارة تأكيد إعطاء الدواء» while doses are due, on a custom path «إعطاء الدواء» → end, with a reminder a day before (POC; RSD «تأكيد إعطاء الدواء حسب جدول العلاج»); the tests run «مرة واحدة لكل حمل جديد» (RSD).
- **FR-124** [BE]: **The «مجموعة خدمات دعم صحة المرأة» path** (RSD 0.4; the new name — 026):
  - Stage 1 «الفحوصات التكميلية» (the nurse; «صلاحيات التمريض: إحالة خارجية»): the question «هل تشعر بآلام أو أعراض بالثدي؟» (required); **random glucose** and the **clinical breast exam** when: (18 – 24 and symptoms = yes) or (25 – 34 and (symptoms = yes, or tumours in the personal or family history, or hormonal contraception / ovulation stimulants)) or (35 and over) (RSD — G-30); and «نوع المشاكل بالثدي» when the exam is abnormal.
  - Branch «نتيجة فحص الثدي» (nurse's choice): «يحتاج إحالة خارجية» (suggested by «الفحوصات التكميلية»: «فحص الثدي غير طبيعي» when the clinical breast exam = «غير طبيعي» — FR-023) → «طبيب العيادة» (external hospital, «بانتظار التشخيص النهائي»; exam confirmation, problem types, mass size in cm, side, mass location) → a condition branch «تأكيد الفحص غير طبيعي» (else end — RSD, G-31) → «طبيب الأشعة» («بانتظار التشخيص من الاشعة»; sonar report and results, mammogram and results when 35 and over, «هل تحتاج سحب عينة؟», «هل تم سحب العينة بالمستشفى؟», «تاريخ سحب العينة») → a condition branch «سُحبت العينة بالمستشفى» (else end) → «طبيب الباثولوجي» («بانتظار التشخيص من الباثولوجي»; pathology results, final diagnosis) → a condition branch «Malignant» → «مقرر لجنة متعددة التخصصات MDT» («بانتظار تحديد خطة العلاج»; «خطة العلاج المحددة» from the «سرطان الثدي» diagnosis plans in 023) → end, else (Benign) end (G-32); «غير ذلك» = «لا يحتاج إحالة» → end.
  - Follow-up: a recurring «زيارة دورية روتينية شاملة» on the inherited path (all stages repeat by the current age band — RSD), with mandatory rules: «Malignant ← أوقف المتابعة» (POC), «Benign ← بعد 6 أشهر», «تحتاج سحب عينة ولم تُسحب بالمستشفى ← بعد 6 أشهر», and «غير ذلك ← مدة ثابتة سنة» (RSD); an "on attendance" «زيارة عارضة» for a female who used the group before (Eligibility RSD 0.4); a reminder a week before.
- **FR-125** [BE]: The **catalog items the paths need are seeded with them, from the RSD** (G-33; units, digits and answers as in the RSD, and what the RSD refers on becomes stage «الاقتراحات» in FR-121 – FR-124, not a flag on an answer or band (G-50), and whatever the RSD lacks from the POC, to be confirmed — G-34): lab tests (020): random glucose, the lipid kit, the kidney kit, dipstick, haemoglobin (g/dL, 2 decimals; its referral limits by sex are the FR-122 suggestion condition, not a 020 property — G-38), and the mother-and-child kit («التهاب الكبد الفيروسي ب» (Hepatitis B), HIV, syphilis with a «متفاعل / غير متفاعل» result — G-35); exams (022): clinical breast exam, the ECG kit, right-eye and left-eye vision, clinical exam, dental, sonar, hearing, breast sonar, mammogram, pathology results; questionnaires (024 once updated to RSD 0.5); the diagnoses and treatment plans needed («الزهري», «سرطان الثدي» — 023); and the medicines those plans need (021).

### Functional Requirements — Shared

- **FR-130** [BE]: Every change to the path, the follow-up paths and their rules is audited with the actor and time (within the 026 FR-100 audit).
- **FR-131** [FE]: The screens are available in Arabic (default, RTL) and English (LTR), WCAG 2.1 AA compliant, on desktop, tablet and phone (constitution, Principle V); QA testing is in Arabic only.

### Key Entities

- **Workflow path**: a sequence of nodes for a group (base) or a follow-up type (custom).
- **Node**: stable key, kind, title; a stage has its status, items, links, «الاقتراحات» and kind settings; a branch has its type (one arm / several together), its arms and «غير ذلك»; and «إنهاء».
- **Stage item**: type, catalog item, applicability condition, by unit availability, field mode.
- **Arm**: key, kind (condition / choice), name, condition, stages.
- **Stage suggestion**: condition, reason, the suggested branch and arm (a choice arm after the stage).
- **Link**: relation type, condition, required, items.
- **Fixed medicine**: medicine, dose, dose count, interval.
- **Follow-up definition**: visit type, enabled, priority, path (inherited / custom), recurrence, activation condition, rules, «غير ذلك», reminder.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: No path breaking any rule of FR-012 – FR-088 is saved (0 false accepts in a sample covering every message in FR-090).
- **SC-002**: Every validation error is shown at its position together with the other errors in a single attempt (100% in the sample).
- **SC-003**: No catalog item used in an active group's path is deactivated (0 accepts).
- **SC-004**: The System Admin builds a path with two stages and a two-arm choice branch in under 15 minutes without help in at least 80% of the test group's attempts.

## Assumptions

- **Actor**: the System Admin only.
- **Source**: the structure comes from the POC by user decision; the Figma design for the fixed stages informs the look only (G-12).
- **PROVISIONAL**: naming the using groups (FR-111).
- **Dependencies**: [`001-condition-builder`](../001-condition-builder/spec.en.md); [`002-eligibility-form-fields`](../002-eligibility-form-fields/spec.en.md); [`007-lookup-option-conditions`](../007-lookup-option-conditions/spec.md); [`018-authentication`](../018-authentication/spec.en.md); 020 – 026; future stage and messaging specifications.
- **Security**: under the 026 permission; Security Team sign-off on "Authorization testing" and "Business logic testing" is required (qa-standards §13.2).
- **Frontend**: the frontend constitution is still a draft (v2.0.0).

## Gap Register

| ID | Topic | Status |
|---|---|---|
| G-01 | Questionnaire-band follow-up (024) versus the scheduling rules here — FR-105 | Answered 2026-10-28: mandatory rule wins, otherwise the band's follow-up is suggested to the doctor — void 2026-11-03 (G-52) |
| G-02 | Repeating a request status in more than one stage (DC forbids, POC allows) — FR-021 | Answered 2026-10-28: allowed (POC) |
| G-03 | The source of item "unit availability" — FR-042 | Answered 2026-10-28: a «العناصر غير المتاحة بالوحدة» list on the health-unit screen (a change to 009) |
| G-04 | The stage title and its length — FR-014 | Answered 2026-10-28: optional, up to 100 characters, defaulting to the kind name |
| G-05 | Wording of the validation messages (colloquial in the POC) — FR-090 | Answered 2026-10-28: reworded in formal Arabic |
| G-06 | Range of "days before the date" in the reminder — FR-106 | Answered 2026-10-28: 1 – 30 |
| G-07 | The source of the seeded paths — FR-120 | Answered 2026-10-28: RSD first, then the POC for what the RSD does not define, conflicts put to the user |
| G-08 | Running the path on cases (nurse, doctor, branches, doses), including parallel-branch tracks (POC M.38: a «مسارات متوازية جارية» status in 025, the case listed once per open track in the worklist, choosing several destinations with a hospital for each, a «١ من ٣ انتهت» card) | Routed to the stage specifications |
| G-09 | Actually sending the reminder | Routed to the messaging specifications |
| G-10 | Performance and data-volume targets | PENDING — QA Control Register |
| G-11 | Test group size and method for SC-004 | Routed to test-plan §11 |
| G-12 | Figma design for the tree and follow-up (the current design is for fixed stages) | Routed to the design team |
| G-13 | Cases sitting on a stage that is deleted | Settled: cases continue on their revision (026 FR-063) |
| G-16 | Fields available to scheduling-rule conditions — FR-103 | Answered 2026-10-28: all condition fields |
| G-17 | The detail of the four seeded paths — FR-120 | Answered 2026-10-28: FR-121 – FR-125 after the conflict questions |
| G-21 | The chronic random-glucose condition — FR-121 | Answered 2026-10-28: the RSD condition |
| G-22 | HbA1c, lipids, kidney and dipstick conditional or always — FR-121 | Answered 2026-10-28: conditional (RSD) |
| G-23 | The «HbA1c 10% أو أكثر» suggestion and the «باطنة» specialty — FR-121 | Answered 2026-10-28: yes (POC) |
| G-24 | A high reading for a previously referred patient in the monthly visit — FR-121 | Answered 2026-10-28: direct referral to the same hospital (RSD) |
| G-25 | HbA1c rules for the comprehensive-visit date — FR-121 | Answered 2026-10-28: recommendation (POC); values to be confirmed by the client |
| G-26 | Placement of the elderly items (doctor, not nurse) and the awareness question — FR-122 | Answered 2026-10-28: as in RSD 0.5 |
| G-27 | Elderly referral destinations — FR-122 | Answered 2026-10-28: four destinations with suggestions (POC) |
| G-28 | «كل شهر» in the elderly doctor's choices — FR-122 | Answered 2026-10-28: allowed (POC) so the nutrition rule works |
| G-29 | Syphilis with Hepatitis B / HIV — FR-123 | Answered 2026-10-28: doctor then referral (RSD) |
| G-30 | The breast-exam and random-glucose condition in women's health — FR-124 | Answered 2026-10-28: the RSD condition and the symptoms question |
| G-31 | The clinic doctor confirming «طبيعي» closes the case — FR-124 | Answered 2026-10-28: yes (RSD) |
| G-32 | Malignant: MDT for all or by age (RSD 0.4 contradicts itself) — FR-124 | Answered 2026-10-28: MDT for all |
| G-33 | Seeding the catalog items the paths need — FR-125 | Answered 2026-10-28: from the RSD |
| G-34 | Exam answers the RSD does not define (dental, sonar, hearing, clinical, mammogram, pathology) — FR-125 | Provisional — from the POC, to be confirmed by the client |
| G-35 | The «متفاعل / غير متفاعل» result type versus the 020 types — FR-125 | Routed to 020 (the existing client question) |
| G-36 | «زيارة تأكيدية لتأكيد الاشتباه بمرض الضغط» (Chronic Diseases RSD 0.4) | Deferred by user decision 2026-10-28 |
| G-37 | The impact of the new RSDs on other specifications (008, 009, 010 – 014, 020, 022) | Answered 2026-10-28: reviewed; 010 – 012 and 014 already match; 008, 009, 013, 020, 022, 024 and 026 updated (each records its change) |
| G-38 | Haemoglobin referral limits by sex (Geriatrics RSD 0.5) — FR-122, FR-125 | Answered 2026-10-28: a suggestion condition on the «طبيب باطنة» arm (POC) |
| G-39 | The pregnancy group name — FR-123 | Answered 2026-10-28: «مجموعة خدمات العناية بصحة الأم والجنين» (026 G-19) |
| G-40 | The parallel branch (POC M.38) in this specification — FR-086 – FR-088 | Answered 2026-11-01: type, validation and seed here; running the tracks routed to the stage specifications (G-08) |
| G-41 | A stalled track, maximum arms taken, arms excluding each other — FR-086 | Answered 2026-11-01: as in the POC (no timeout, no maximum, none exclusive); to be confirmed by the client |
| G-42 | Which seeded branches are parallel — FR-122, FR-123 | Answered 2026-11-01: the elderly «جهة الإحالة» only; the mother-and-child «نتائج التحاليل» stays sequential (RSD — G-29), unlike the POC |
| G-43 | Must every sequence end with «إنهاء» (the POC server does not refuse) — FR-085 | Answered 2026-11-01: yes, the save is refused, in the editor and on the server |
| G-44 | Nurse «إحالة خارجية» with a unit-doctor stage after it: refuse or warn — FR-032 | Answered 2026-11-01: refused (as written), unlike the POC warning |
| G-45 | An arm key must not contain the separator used to store several chosen arms | Routed to `/speckit-plan` (storage design) |
| G-46 | Two System Admins saving the same group's path at the same time | Routed to 026 (the single save and revisions) |
| G-47 | Referral suggestions on the stage, not on the choice arm (POC M.40) — FR-023, FR-082 | Answered 2026-11-02: as in the POC |
| G-48 | The destination an abnormal ECG suggests in the elderly seed — FR-122 | Answered 2026-11-02: «طبيب باطنة», unlike the POC |
| G-49 | The «ضعف سمع» → «الطبيب المختص» suggestion (POC) — FR-122 | Answered 2026-11-02: not added (not in the RSD) |
| G-50 | Dropping the 🚩 «يحتاج إحالة» flag; the questionnaire «تفسير» field — FR-121 – FR-125 | Answered 2026-11-02: adopted; 022 and 024 updated |
| G-51 | Moving reasons saved on arms to the stage (POC `MoveArmSuggestionsToStages`) | Settled: no older paths in the product; not applicable |
| G-52 | Follow-up of a questionnaire interpretation once dropped from the band (024 G-57) — FR-105, FR-122 | Answered 2026-11-03: a scheduling rule on the «تفسير» field; nutrition «معرّض لسوء التغذية ← كل شهر» as a recommendation before «مريض مزمن» |
