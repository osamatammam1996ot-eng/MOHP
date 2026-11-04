# Specification Quality Checklist: مراحل سير عمل مجموعة الخدمات (027-service-group-workflow)

**Purpose**: Validate specification completeness and quality before proceeding to planning
**Created**: 2026-10-28
**Feature**: [spec.md](../spec.md) (Arabic, source of truth) · [spec.en.md](../spec.en.md) (English translation, same IDs)

## Content Quality

- [x] No implementation details (languages, frameworks, APIs). Node and item kinds are named only as configuration concepts.
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders (Arabic, RSD terminology)
- [x] All mandatory sections completed

## Requirement Completeness

- [x] No [NEEDS CLARIFICATION] markers remain — G-01 (mandatory rule wins, otherwise the band follow-up is suggested), G-02 (repeated status allowed) and G-03 (unit availability list on the health-unit screen, a change to 009) answered 2026-10-28.
- [x] Requirements are testable and unambiguous (POC validation messages quoted; provisional wording marked)
- [x] Success criteria are measurable
- [x] Success criteria are technology-agnostic
- [x] All acceptance scenarios are defined (Given / When / Then with FR references)
- [x] Edge cases are identified
- [x] Scope is clearly bounded (running the path, sending reminders and unit availability out of scope)
- [x] Dependencies and assumptions identified (PROVISIONAL and PENDING listed explicitly)

## Feature Readiness

- [x] All functional requirements have clear acceptance criteria
- [x] User scenarios cover primary flows (stages, items, branches, links and fixed medicines, follow-up, validation)
- [x] Feature meets measurable outcomes defined in Success Criteria
- [x] No implementation details leak into specification

## Notes

- User decision 2026-10-28: the POC workflow tree (not the DC / Figma fixed stages); 027 completes 026.
- Sources: POC `WorkflowDefinition`, `WorkflowDefinitionValidator`, `FollowUpDefinition`, `WorkflowTreeEditor`, `WorkflowFlowchart`, `FollowUpSettingsEditor`, seed paths; DC §7 / §9; RSD paths; Figma `726:429`, `729:1113`, `729:1393`, `727:658` (look only).
- FR-110 closes the "used by an active service group" items in 020 – 024 (020 G-03, 021 G-02, 022 G-03, 023 G-06, 024 G-06).
- FR IDs FR-005 – FR-009, FR-016 – FR-019, FR-024 – FR-029, FR-033 – FR-039, FR-045 – FR-049, FR-052 – FR-059, FR-061 – FR-069, FR-071 – FR-074, FR-076 – FR-079, FR-089, FR-091 – FR-099, FR-107 – FR-109, FR-112 – FR-119, FR-126 – FR-129 are unused.
- Cross-spec change needed: 009 gains a «العناصر غير المتاحة بالوحدة» list (G-03).
- `/speckit-clarify` 2026-10-28: G-04 – G-07, G-16 answered; G-07 answered as a rule (RSD first, POC only for gaps, conflicts asked) — saved as a standing user preference; validation messages reworded in formal Arabic (G-05); G-17 (the detail of the four seeded paths) stays open, to be drafted from the RSDs with the user.
- G-17 drafting 2026-10-28: the four seeded paths written as FR-121 – FR-124 and their catalog items as FR-125, from the latest RSDs (Chronic Diseases 0.4, Geriatrics 0.5, Mother and Child 0.4, Women's Health Support 0.4, Eligibility 0.4); every RSD/POC conflict asked (G-21 – G-33); G-34 provisional POC answers to confirm with the client; G-36 confirmatory BP visit deferred by the user.
- Cross-spec changes from the new RSDs: 024 updated to Geriatrics 0.5 and 026 FR-090 renamed (both done); 008, 009, 010 – 014, 020 and 022 still to review against the new versions (G-37).
- Next: `/speckit-plan`.

## Gap Register (QC-5)

Story risk: US1 — High — the path drives every case; a wrong stage, status or end stops or misroutes cases
Story risk: US2 — High — applicability conditions decide which tests each beneficiary gets
Story risk: US3 — High — branches decide referrals; suggestion reasons shape the doctor's choice; parallel branches (2026-11-01) add tracks and join rules
Story risk: US4 — Medium — linked persons and fixed medicines (mother and child)
Story risk: US5 — Medium-High — scheduling rules and their interplay with questionnaire bands set follow-up dates
Story risk: US6 — Medium — validation completeness and the read-only flowchart

- [x] G-01 [High][Q] Mandatory scheduling rule wins, otherwise the questionnaire-band follow-up is suggested to the doctor [Spec §FR-105]
- [x] G-02 [Medium][Q] The same request status allowed in more than one stage [Spec §FR-021]
- [x] G-03 [Medium][Q] Unit availability from a «العناصر غير المتاحة بالوحدة» list on the health-unit screen (change to 009) [Spec §FR-042]
- [x] G-04 [Low][Q] Optional stage title ≤ 100, defaulting to the kind name [Spec §FR-014]
- [x] G-05 [Low][Q] Validation messages reworded in formal Arabic [Spec §FR-090]
- [x] G-06 [Low][Q] Reminder days 1 – 30 [Spec §FR-106]
- [x] G-07 [High][Q] Seeded paths from the RSDs first, POC only for gaps, conflicts asked [Spec §FR-120]
- [ ] G-08 [High][X] Running the path on cases → stage specifications
- [ ] G-09 [Medium][X] Sending reminders → messaging specifications
- [ ] G-10 [Low][X] Performance and data-volume targets → QA Control Register
- [ ] G-11 [Low][X] Test group size and method for SC-004 → test-plan §11
- [ ] G-12 [Medium][X] Figma for the tree and the follow-up tab → design team
- [x] G-13 [Low][S] Cases on a deleted stage continue on their revision (026 FR-063)
- [x] G-16 [Low][Q] Scheduling-rule conditions use all condition fields [Spec §FR-103]
- [x] G-17 [High][Q] Detail of the four seeded paths — FR-121 – FR-125 [Spec §FR-120]
- [x] G-21 [High][Q] Chronic random-glucose condition from the RSD [Spec §FR-121]
- [x] G-22 [High][Q] HbA1c, lipids, kidney, dipstick conditional (RSD) [Spec §FR-121]
- [x] G-23 [Medium][Q] «HbA1c 10% أو أكثر» suggestion and «باطنة» specialty (POC) [Spec §FR-121]
- [x] G-24 [High][Q] Previously referred high reading → direct referral to the same hospital (RSD) [Spec §FR-121]
- [x] G-25 [Medium][Q] HbA1c follow-up rules as recommendations (POC), doctor picks 3 / 6 / 12 [Spec §FR-121]
- [x] G-26 [High][Q] Elderly items in the doctor stage; awareness question hides 4AT (RSD 0.5) [Spec §FR-122]
- [x] G-27 [Medium][Q] Four elderly referral destinations with suggestions (POC) [Spec §FR-122]
- [x] G-28 [Low][Q] «كل شهر» allowed for the elderly doctor (POC) [Spec §FR-122]
- [x] G-29 [High][Q] Syphilis with Hepatitis B / HIV → doctor then referral (RSD) [Spec §FR-123]
- [x] G-30 [High][Q] Women's breast-exam and glucose condition by age band and symptoms (RSD) [Spec §FR-124]
- [x] G-31 [Medium][Q] Normal clinic confirmation closes the case (RSD) [Spec §FR-124]
- [x] G-32 [High][Q] Malignant → MDT for all ages [Spec §FR-124]
- [x] G-33 [Medium][Q] Catalog items the paths need seeded from the RSD [Spec §FR-125]
- [ ] G-34 [Medium][Q] Exam answers the RSD does not define — provisional from the POC, client to confirm [Spec §FR-125]
- [ ] G-35 [Medium][X] «متفاعل / غير متفاعل» result type → 020 client question
- [ ] G-36 [Medium][D] Confirmatory BP visit (Chronic RSD 0.4) — deferred by the user
- [x] G-37 [High][Q] New-RSD impacts reviewed: 010 – 012, 014 match; 008, 009, 013, 020, 022, 024, 026 updated after the user's answers
- [x] G-38 [Medium][Q] Haemoglobin referral limits by sex as a suggestion condition on the «طبيب باطنة» arm [Spec §FR-122, FR-125]
- [x] G-39 [Low][Q] Pregnancy group named «مجموعة خدمات العناية بصحة الأم والجنين» [Spec §FR-123]
- [x] G-40 [High][Q] Parallel branch (POC M.38): type, validation and seed here; tracks at run time → stage specifications [Spec §FR-086 – FR-088]
- [x] G-41 [Medium][Q] Stalled track waits, no maximum, no exclusive arms (POC; client to confirm) [Spec §FR-086]
- [x] G-42 [Medium][Q] Only the elderly «جهة الإحالة» seeded parallel; mother and child stays sequential (RSD) [Spec §FR-122, FR-123]
- [x] G-43 [High][Q] Every sequence must end with «إنهاء» — save refused in the editor and on the server [Spec §FR-085]
- [x] G-44 [Medium][Q] Nurse «إحالة خارجية» with a unit-doctor stage after it is refused, not a POC warning [Spec §FR-032]
- [ ] G-45 [Low][X] Arm key must not contain the separator used to store several chosen arms → /speckit-plan
- [ ] G-46 [Medium][X] Two System Admins saving the same group's path at once → 026
- [x] G-47 [High][Q] Referral suggestions on the stage, arm shows them read-only (POC M.40) [Spec §FR-023, FR-082]
- [x] G-48 [Medium][Q] Abnormal ECG suggests «طبيب باطنة», unlike the POC [Spec §FR-122]
- [x] G-49 [Low][Q] No hearing-exam suggestion (not in the RSD) [Spec §FR-122]
- [x] G-50 [High][Q] 🚩 flag dropped; seed referrals become stage suggestions reading «تفسير»; 022 and 024 updated [Spec §FR-121 – FR-125]
- [x] G-51 [Low][S] Moving older arm reasons to the stage — no older paths in the product
- [x] G-52 [Medium][Q] Questionnaire follow-up is a scheduling rule on «تفسير» (024 G-57); nutrition rule before «مريض مزمن» [Spec §FR-105, FR-122]
