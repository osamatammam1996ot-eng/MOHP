# Specification Quality Checklist: إدارة مجموعات الخدمات (026-service-groups)

**Purpose**: Validate specification completeness and quality before proceeding to planning
**Created**: 2026-10-28
**Feature**: [spec.md](../spec.md) (Arabic, source of truth) · [spec.en.md](../spec.en.md) (English translation, same IDs)

## Content Quality

- [x] No implementation details (languages, frameworks, APIs). The permission name is named only as a catalogue identifier shared with 019.
- [x] Focused on user value and business needs
- [x] Written for non-technical stakeholders (Arabic, RSD terminology)
- [x] All mandatory sections completed

## Requirement Completeness

- [x] No [NEEDS CLARIFICATION] markers remain — G-01 (one save for the whole group), G-02 (saved rules may be deleted) and G-03 (list named «حالات مجموعات الخدمات», 025 updated) answered 2026-10-28.
- [x] Requirements are testable and unambiguous (DC / RSD / Figma messages quoted; provisional texts marked)
- [x] Success criteria are measurable
- [x] Success criteria are technology-agnostic
- [x] All acceptance scenarios are defined (Given / When / Then with FR references)
- [x] Edge cases are identified
- [x] Scope is clearly bounded (workflow stages in 027; eligibility display in 013; health-unit links in 009)
- [x] Dependencies and assumptions identified (PROVISIONAL and PENDING listed explicitly)

## Feature Readiness

- [x] All functional requirements have clear acceptance criteria
- [x] User scenarios cover primary flows (list and basic data, identity, rules, rule test, revisions)
- [x] Feature meets measurable outcomes defined in Success Criteria
- [x] No implementation details leak into specification

## Notes

- User decisions 2026-10-28: the POC workflow tree for the workflow; two specifications — 026 (this) and 027 (workflow stages).
- Figma «Admin — Service Management» (`726:371`) has full designs for the list, details, identity, eligibility rules, rule editor and rule states; the three workflow screens (`726:429`, `729:1113`, `729:1393`, `727:658`) follow the DC fixed stages and are inputs for 027.
- DC vs Figma conflicts recorded: save together vs per-rule save (G-01); two-valued vs three-valued test result (Figma / DC §11 point 5 adopted, FR-052).
- Cross-spec change: 019 P-24 `CanManageServiceGroups` now defined by 026.
- FR IDs FR-003 – FR-009, FR-019, FR-025 – FR-029, FR-039 – FR-049, FR-055 – FR-059, FR-066 – FR-079, FR-081 – FR-089, FR-091 – FR-099 are unused.
- Cross-spec change: 025 «حالات الطلب» renamed «حالات مجموعات الخدمات» (G-03).
- `/speckit-clarify` 2026-10-28: G-04 – G-06, G-13, G-14, G-16 answered; G-13 (English rejection message required) and G-14 (no rule limit) differ from the recommendation; «DC» explained to the user as the dynamic configuration document.
- Next: 027 (workflow stages), then `/speckit-plan`.

## Gap Register (QC-5)

Story risk: US1 — Medium — permission-protected list and basic data; deactivation tied to health units
Story risk: US2 — Low — optional visual identity; file upload needs security sign-off
Story risk: US3 — High — eligibility rules decide who enters every service group
Story risk: US4 — Medium — the test must match the real three-valued evaluation
Story risk: US5 — High — revisions keep visits and cases on the configuration they started with; errors change patients' requirements mid-journey

- [x] G-01 [High][Q] One save for the whole group (DC) [Spec §FR-035]
- [x] G-02 [High][Q] Saved rules may be deleted; kept in earlier revisions [Spec §FR-038]
- [x] G-03 [Medium][Q] List named «حالات مجموعات الخدمات»; 025 updated [Spec §Sources]
- [x] G-04 [Low][Q] Code as in 020; names ≤ 100; rejection message ≤ 250 [Spec §FR-011, FR-012, FR-030]
- [x] G-05 [Medium][Q] Five goals seeded [Spec §FR-080]
- [x] G-06 [Medium][Q] The four DC groups and their rules seeded; not «نصائح غذائية للأم» [Spec §FR-090]
- [ ] G-07 [Medium][X] "Used by an active service group" in 020 – 024 → 027
- [ ] G-08 [Medium][X] Showing the eligibility result and rejection reasons → DC §10 / 013
- [ ] G-09 [Low][X] Performance and data-volume targets → QA Control Register
- [x] G-10 [Low][S] Security sign-off categories (constitution, qa-standards §13.2)
- [ ] G-11 [Low][X] Test group size and method for SC-004 → test-plan §11
- [x] G-12 [Low][S] Rejection message shown to the nurse (Figma) [Spec §FR-030]
- [x] G-13 [Medium][Q] English rejection message required [Spec §FR-030]
- [x] G-14 [Low][Q] No maximum number of rules [Spec §FR-030]
- [ ] G-15 [Medium][X] SVG logo content safety → Security Team sign-off and `/speckit-plan`
- [x] G-16 [High][Q] Visits evaluated against all active groups; groups not at the visit's unit shown unselected and disabled with «غير متاحة بالوحدة» (revised by the user) [Spec §FR-062]
- [x] G-17 [Low][Q] Women's group renamed «مجموعة خدمات دعم صحة المرأة» per RSD 0.4 [Spec §FR-090]
- [x] G-18 [High][Q] Chronic rule adds BMI ≥ 30 OR average BP > 139/89 (Eligibility RSD 0.4); rejection wording for the client [Spec §FR-090]
- [x] G-19 [Low][Q] Pregnancy group named «مجموعة خدمات العناية بصحة الأم والجنين» (Eligibility RSD 0.4) [Spec §FR-090]
- [x] G-20 [Medium][Q] A field hidden by its visibility condition counts as answered-empty (POC M.39) [Spec §FR-039]
- [x] G-21 [Low][S] POC M.40 changes nothing here; band codes frozen in the revision [Spec §FR-060]
