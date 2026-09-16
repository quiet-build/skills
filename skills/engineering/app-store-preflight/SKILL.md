---
name: app-store-preflight
license: MIT
description: Review Apple App Store submission readiness or investigate an App Review rejection. Check listing metadata against the actual release build and current Apple rules, with concrete fixes and evidence. Use for early submission planning and final preflight, not routine code review.
---

# App Store preflight

Find likely rejection causes early and produce actionable, evidence-backed fixes. Reduce risk; never promise Apple approval.

## Scope and evidence

Infer platform, project and review stage from the request. Identify app name, bundle ID, version/build, target storefronts, localizations and any prior rejection. Ask only for missing facts that materially affect the review; continue independent checks.

- Early review: inspect available designs, implementation and draft metadata; identify expensive-to-change policy risks before release. Missing release artifacts are future checks, not fabricated failures.
- Final preflight: inspect the exact submission metadata and release candidate, including entitlements/configuration and production service availability. Record which build was actually exercised.
- Rejection follow-up: start with Apple's exact message, verify the named surface, then check related surfaces for the same issue. Do not expand into unrelated refactoring.

Use existing project instructions and release materials. Read connected App Store Connect pages when available; otherwise distinguish local drafts from the live listing. Never infer live state from old notes. Keep reviewer credentials and private account data out of reports.

A review request authorizes inspection, not code edits, metadata saves, reviewer messages or submission. If fixes are requested, apply scoped authorized fixes and verify them. Draft proposed metadata and replies before requesting any still-needed authorization. Do not add approval gates to already-authorized work.

## Verify current rules

Consult current official Apple sources for applicable checks and cite the exact supporting section, URL and date checked. Use an available web search or page-reading tool. Do not install tools just to read documentation. If access fails, report policy verification as incomplete rather than asserting remembered requirements.

Start with relevant sections, not a full documentation crawl:

- [App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/): Before You Submit, completeness (2.1), metadata (2.3), software requirements (2.5), business (3), design (4), privacy (5.1), intellectual property (5.2).
- [Apple trademark guidance](https://www.apple.com/legal/intellectual-property/guidelinesfor3rdparties.html): names, logos, endorsement and permitted compatibility references.
- [App Review preparation](https://developer.apple.com/app-store/review/).
- [Edit app information](https://developer.apple.com/help/app-store-connect/create-an-app-record/view-and-edit-app-information/).
- [Resolve review messages](https://developer.apple.com/help/app-store-connect/manage-submissions-to-app-review/reply-to-app-review-messages/).

Follow official links for feature-specific requirements, SDK/privacy manifest obligations and current submission SDK deadlines. Payment, external-link and distribution exceptions vary by storefront and entitlement: verify the actual applicable case.

## Review surfaces

Apply relevant rows; explicitly mark others not applicable. Automated text searches are discovery aids, not proof of compliance.

| Surface | Inspect and verify |
| --- | --- |
| Name, subtitle, keywords, promotional text, description | Read every submitted localization. Flag prominent Apple/product/competitor trademarks, implied endorsement, keyword stuffing, unsupported superlatives, unsuitable pricing copy, placeholder text and claims the build cannot demonstrate. Check field limits against current Connect requirements. Supply exact replacement copy. |
| Icons, screenshots, previews | Visually inspect actual submitted assets, including embedded captions. Check confusing Apple branding, copied assets, misleading UI, unsupported features and mismatch with platform/build. A filename or OCR scan alone is insufficient. |
| Rights | Verify relevant third-party code, artwork, fonts, audio, brands and content permissions. Code and asset licenses are separate. Do not presume permission from public availability. |
| Privacy and permissions | Trace actual collection, storage, transmission and third-party SDK behavior against privacy labels, policy and in-app disclosures. Verify working policy links, accurate purpose strings, required consent, manifests/required-reason API declarations where applicable, and deletion flows if accounts are created. Local-only/no-tracking claims need implementation evidence. |
| Purchases and accounts | Where applicable, test purchase, restoration, entitlement and subscription flows; verify price/renewal disclosures, terms and privacy links, login requirements, account deletion and applicable sign-in rules. Check current storefront-specific payment rules before flagging a violation. |
| Functional readiness | Fresh normal launch of the release candidate; exercise onboarding and core advertised flows using real UI input. Check permission denial, empty/offline/error states where relevant, crashes, persistence and production backend access. Builds/unit tests/simulator fixtures do not prove the shipped flow works. Record unavailable device/runtime acceptance as unverified. |
| Platform requirements | Check distribution signing, sandbox/entitlements, public API usage, embedded extensions/SDKs, OS/device support and current submission requirements. Inspect only applicable platform requirements; do not apply iOS-only rules to macOS. |
| Reviewer access and listing setup | Confirm build selection, complete contact details, working support/privacy URLs, accurate age rating/category/content-rights declarations, and reviewer access to non-obvious or gated features. Provide clear review steps and demo access where needed. Verify services remain usable during review. |
| Conditional policy risks | Review user-generated content/moderation, children, health, finance, gambling, AI data sharing or regulated functionality only when present. Verify relevant current rules and jurisdiction rather than inventing universal requirements. |

Do not turn architecture preferences, file sizes, missing abstractions or unrelated technical debt into App Store blockers. Trace implementation only as needed to validate behavior and policy claims.

## Example: Apple product names in subtitles

A subtitle such as “Clipboard history on your Mac” can attract scrutiny under guideline 5.2.5. A feature-focused alternative is “Clipboard history made simple”. Treat this as a risk-review example, not an Apple-approved wording template.

For comparable submissions, flag Apple product names in prominent metadata for contextual review and prefer feature-focused subtitle wording when the platform reference is unnecessary. Do not assert that all uses of “Mac” or “for Mac” are banned: Apple's trademark guidance permits some referential compatibility uses, while App Review assesses metadata in context. Do not remove accurate compatibility statements indiscriminately.

## Findings and completion

Lead with READY FOR SUBMISSION, BLOCKED, or UNVERIFIED for final preflight. READY means no identified blockers and all applicable required checks verified; it is not an approval guarantee. For early reviews, report current risks and remaining release checks without declaring a draft submission-ready.

For each finding include:
- Severity and certainty: confirmed violation/rejection, likely risk, or missing evidence.
- Exact affected field/localization, screenshot, file location or reproducible user flow.
- Supporting evidence and current Apple rule; distinguish the rule from your interpretation.
- Smallest concrete fix, including replacement copy where appropriate.
- Verification result or exact remaining acceptance step.

Summarize checked, not-applicable and unverified areas compactly. Keep local tests, real release-build interaction, live metadata, submission and Apple approval distinct. Preserve a dated `docs/app-store-preflight.md` report for substantial reviews when project conventions permit, updating existing release documentation instead of duplicating it. Include build identity and policy check date; invalidate affected checks after build or metadata changes.

For metadata-only rejections, verify whether the same build may be resubmitted using current Apple guidance. Draft a truthful reviewer reply describing only completed changes. Never claim an unsaved proposal was implemented or that a submission was sent without observing confirmation.
