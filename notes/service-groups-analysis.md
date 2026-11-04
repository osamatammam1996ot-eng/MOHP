# Service Groups Management — analysis (specs 026 + 027 vs. old Figma design)

Status: working notes, 2026-11-04. Inputs read: `specs/026-service-groups`, `specs/027-service-group-workflow` (EN + checklists; Arabic is the source of truth), Figma page "Admin — Service Management" (`726:371`). POC (RuleBuilder-UX) and the meeting video: **not yet reviewed**.

## 1. What the feature is

| Spec | Scope | Tabs in group details |
|---|---|---|
| **026** | List, basic data (code, AR/EN name, description, goals), service identity (logo + 2 colours), eligibility rules + rule test, configuration revisions, seed data | «البيانات الأساسية», «شروط الأهلية» |
| **027** | Workflow as a **tree** (POC model, not the 3 fixed stages), stage items + applicability conditions, branches (single / parallel), linked persons, fixed medicines, stage suggestions, validation + flowchart, follow-up types | «مراحل سير العمل», «المتابعة» (new) |

Actor: System Admin only (`CanManageServiceGroups`). **One save for the whole group across all tabs** — all or nothing (026 FR-035, G-01). Every save that changes rules / workflow / items creates an immutable revision; visits and cases stay on the revision they started on.

## 2. Old Figma design → what must change

### 2.1 List — `812:11113`
- Shows 5 groups incl. «نصائح غذائية للأم» → **not seeded** (G-06). Rename: «صحة المرأة (سرطان الثدي)» → «مجموعة خدمات دعم صحة المرأة» (G-17); «الأم والجنين» → «مجموعة خدمات العناية بصحة الأم والجنين» (G-19).
- Missing: **pagination** (FR-010), **empty search** «لا توجد مجموعات خدمات مطابقة لبحثك.», **deactivation refused** state «لا يمكن تعطيل مجموعة الخدمات، لأنها مستخدمة بوحدة صحية مفعلة» (FR-017), success toasts (FR-015).

### 2.2 Details · «البيانات الأساسية» — `812:11288`, identity `896:3203` / `896:3767` / `896:3992`, notes `898:3574`
- Missing fields: **group code** (required, unique, locked after creation — FR-011) and **English name** (required, ≤100 — FR-012). Name ≤100, duplicate message «اسم مجموعة الخدمات مستخدم بالفعل».
- Missing states: **create mode** (new group), duplicate-name error, save disabled «لا يوجد تغيير غير محفوظ» (FR-018).
- Identity matches FR-020 – FR-024 well (upload/drag, 1:1 crop to 512, extracted colours, HEX edit, source tag, restore, contrast warning non-blocking). Colour sources must include «افتراضي», «مستخرج من الشعار», «مولّد من اللون الأساسي», «معدّل يدويًا».
- **Save/Cancel sit inside each tab's card** → must become one page-level save for the whole group, with unsaved changes kept across tabs and «توجد تعديلات لم تُحفظ بعد» (027 FR-004).
- Content overflows the 960 frame (frame sizing only).

### 2.3 «شروط الأهلية» — `796:1707`, editor `798:18184`, states `800:2603`
Keep: AND explanation at top, numbered rule cards, «يجب أن تتحقق أيضًا» between them, paths joined by «أو», «رسالة الرفض للممرض», «إضافة قاعدة…», side panel «جرّب على مستفيد» with per-rule result, condition builder (All/Any, NOT), three-valued test result.
Change / add:
- **No per-rule save** («حفظ القاعدة» / drawer «حفظ» not adopted — G-01). The drawer's action must only apply the edit to the draft.
- **English rejection message** (required, ≤250 — G-13).
- Per-card **state line** (FR-034): «جاهزة — ستُحفظ هذه القاعدة تلقائيًا عند الضغط على "تعديل وحفظ".» / «أكمل اسم القاعدة ورسالة الرفض وشغّل الاختبار — …» / «لا يوجد تغيير في هذه القاعدة.»
- **Version label** above each saved card: «قاعدة #رقم — النسخة الحالية: v(رقم)» (FR-036).
- **Delete rule** + confirm «هل تريد حذف القاعدة؟ ستُطبَّق المجموعة بدونها على الزيارات الجديدة فقط.» (FR-038).
- Per-rule «اختبار هذه القاعدة» (inputs = fields used in its conditions; «أضف شروطاً للقاعدة أولاً» + disabled run when empty; test invalidated after editing conditions) (FR-050 – FR-053).
- **No rules** state: every beneficiary is eligible (FR-033).
- Save-level messages: «أكمل اسم كل قاعدة ورسالة الرفض وشغّل الاختبار عليها قبل حفظ الخدمة.» (FR-034); after a revision «تم الحفظ. ستُطبَّق التعديلات على الزيارات الجديدة فقط — هناك (عدد) زيارات مفتوحة و(عدد) حالات جارية…» (FR-064); concurrency «قام شخص آخر بتعديل إعدادات هذه الخدمة بعد فتحك للصفحة — أعد تحميل الصفحة ثم أعد تعديلاتك» (FR-065 — replaces state 5 in `800:2603`, which is per-rule).
- Editor drawer footer overlaps the content (sticky footer drawn mid-frame) — layout fix.

### 2.4 «مراحل سير العمل» — `726:429`, `729:1113`, `729:1393`, `727:658`
The old design is **3 fixed toggled stages** (فحوصات أساسية / خطوات طبيب وحدة الرعاية الأولية / خطوات الإحالة الخارجية). 027 replaces this with a **workflow tree**; the old screens inform the look only (G-12). Needed:
- **Tree editor**: ordered nodes; kinds «الفحوصات التكميلية», «خطوات طبيب الوحدة», «خطوة إحالة خارجية», «إعطاء الدواء», «تفرّع», «إنهاء»; «+» at any position; move up/down; delete (branch deletes «وكل ما تحته» after confirm).
- **Stage editor**: optional title (≤100), required «حالة الطلب أثناء المرحلة», show basic readings yes/no, items (lab tests, kits, exams, exam kits, questionnaires, field questions with mode «إدخال · إلزامي / إدخال · اختياري / للقراءة»), per-item «شرط التطبيق», «حسب الإتاحة بالوحدة» (labs/exams/kits only), linked persons (relation, condition, required, items), «الاقتراحات» (condition, reason, suggested choice arm after the stage).
  - Nurse stage: ≥1 item; «صلاحيات التمريض: إحالة خارجية».
  - Unit doctor: «من يؤدي المرحلة؟» (doctor / nurse on behalf), permissions (diagnose / treatment / external referral).
  - External referral: «من يرى الخطوة», «تخصص الطبيب», «الإحالة لنفس المستشفى المحال إليها سابقاً».
  - Medication: fixed medicines (medicine, dose text, dose count, interval days).
- **Branch editor**: «نوع التفرّع» فرع واحد / أكثر من فرع معًا (parallel + «تجميع»), arms «شرط على البيانات» / «اختيار الطبيب», mandatory «غير ذلك», read-only «يُقترح عندما» on choice arms.
- **Validation panel**: «لم يُحفظ المسار:» with all problems linked to positions / «✓ المسار سليم — لا توجد مشكلات».
- **Flowchart** view (read-only, click for details; parallel drawn with split bar + «تجميع»).
- Renames vs old design: «فحوصات أساسية» → «الفحوصات التكميلية»; «حالة القضية» → «حالة الطلب أثناء المرحلة».
- Reusable from old design: grouped item list with drag + «يظهر بشرط» + «تعديل الشرط», «كما يظهر للممرض» preview, condition drawer «شرط ظهور: <item>» (+ «حذف الشرط (يُطلب دائمًا)»), performer and permission checkboxes, referral step fields.

### 2.5 «المتابعة» — new, no Figma
Follow-up types (priority order, enabled «· متوقف», empty state «لا متابعة في هذه المجموعة — أضف نوعًا من الأعلى»), path inherited / custom (+ «⚠ عُدِّلت مراحل سير العمل بعد نسخ هذا المسار»), recurrence (recurring / on attendance), activation condition, ordered scheduling rules (condition → next visit after / keep date / stop; mandatory / recommendation), «غير ذلك» (doctor chooses from intervals / fixed interval), reminder (enabled, 1 – 30 days before, SMS / WhatsApp / call, text with {الاسم} {الموعد} {الوحدة}).

## 3. Existing components to build the new flow from
Admin/Tab item, Admin/Stage nav item, Admin/Section header, Admin/Drawer header, Admin/Switch, Admin/Segment option, Admin/Condition line, Admin/Test row, **RB/** Group header (All/Any + NOT), Condition row (7 value types), Simple criterion, Test trace row, Version item; Admin/Logo uploader, Color field, Color picker popover; Molecules: Input field, Text area, Choice option/group, Inline alert, Toast, Options overlay, Confirmation, Empty state; Atoms: Tag, Status badge, Icon button, Button; Organisms: Page header, Stepper.

## 4. Open questions (for the meeting / POC)
1. Where does the single group save live (sticky page bar?) and what is the rule drawer's primary action («تطبيق» / «تم»)?
2. Is a revision-history UI in scope (RB/Version item exists; 026 only requires the current-version label)?
3. Tree editor layout: tree + side editor panel, or canvas; flowchart as a toggle or a separate tab?
4. Create-group flow: the same tabbed page in "new" mode?
5. Tablet / phone and English LTR designs (FR-101 / FR-131 require them) — needed now?
6. Where is the tab-level «جرّب على مستفيد» kept when rules are edited in a drawer?
