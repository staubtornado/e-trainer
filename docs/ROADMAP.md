# Implementation sequence

Milestones are vertical slices. M0–M5 produce a usable private journal, wish engine and evidence-based coach without native inference. Hardware spikes may run after M0 on synthetic data; their results affect only M6–M8. The owner of a milestone must run its tests and attach results to the change. Scope and dependency labels below are normative. “Pass” means a test ran on the named target. An unavailable phone leaves a hardware gate open.

## M0 — Reproducible foundation

**Objective and scope.** Establish a buildable Kotlin/Compose shell and enforce the offline boundary before importing libraries. Depends on the blueprint and existing GPLv3 license.

**Tasks.** Create Gradle wrapper, four projects and package/dependency boundaries from [Architecture](ARCHITECTURE.md). Pin stable compatible toolchain, native ABI plan, locks and verification metadata. Configure Room schema export, JUnit, coroutine test dispatcher, instrumentation runner, test container, injectable clock and code formatting. Add opt-in-free Material theme, Navigation 3 tab stacks, skip-friendly onboarding and blank/empty destinations. Default to bundled resources. Declare backup exclusions and a reviewed manifest baseline.

**Acceptance.** APK installs on API 31 emulator and a current Pixel build without Play. No account/network prompt. Clean offline-cache build succeeds and a second clean build reproduces the unsigned APK. Tabs retain position; light/dark and large-text shell are usable.

**Required tests.** Domain smoke test, architecture dependency check, Compose launch/navigation/back/rotation, final APK permission/dependency inspection and unsigned hash comparison.

**Done when** CI runs the named checks, toolchain/source versions are recorded, and the shell launches with no model/catalog import.

## M1 — First usable training journal

**Objective and scope.** Complete a real workout on first launch. Depends on M0.

**Tasks.** Add profile, exercise, plan, session/set and measurement tables. Seed a small licensed/authored exercise list, add custom exercises, plan creation/reorder, live draft editor, rest timer, explicit finish, history/edit/delete, PR and body trends. Put resume/start, date selector and weekly session count on Today. Autosave draft changes and enforce version-aware commands.

**Acceptance.** In airplane mode create a plan, finish a loaded and timed session, reopen it after restart, correct a set and see the PR change. Rotating or killing the activity mid-workout retains confirmed sets. Show history with an accessible point list.

**Required tests.** PR/units/tie/date golden vectors, Room draft uniqueness, retry/concurrency, rotation/process recovery, rest timer resume and cold-start smoke.

**Done when** a fresh installation supports a repeat workout without retyping its plan and all M1 checks pass.

## M2 — Manual nutrition and personal targets

**Objective and scope.** Deliver the second daily logging flow, independent of camera, runtime or downloaded data. Depends on M1.

**Tasks.** Build a deterministic USDA seed converter from pinned release inputs, notices and test fixtures. Add searchable base and custom foods, basis/servings, recipes with cooked yield, meals/immutable snapshots, logged targets, complete-day switch and corrections. Add date-aware Today summaries and history. Expose nutrient provenance, unknowns and intervals.

**Acceptance.** Log the 150 g golden example, save a cooked recipe, correct a portion and confirm a complete day. After recipe/catalog changes or deletion of a source, the saved meal still shows its original calculation. A user can log a label with incomplete nutrient data.

**Required tests.** Snapshot persistence, unknowns/interval arithmetic, recipe yield, basis mismatch, completeness resets, seed hashes/license attribution, UI manual search/add/edit plus denial-independent logging.

**Done when** Train, Eat and Today are useful on an empty offline device with only the APK installed.

## M3 — Wishes and deterministic progression

**Objective and scope.** Connect everyday logs to real-life wishes. Depends on M1–M2.

**Tasks.** Implement versioned effective policies, exception editor, candidate/credit replay, XP/streak/achievement projections, goal preview/activation/pinning, quarter milestones, forecast, goal history/claim, optional independent savings pot and weekly policy review. Bind Today, progress animation and evidence sheet to committed state.

**Acceptance.** The documented example proposes 20 days and explains why. Duplicate sessions do not exceed caps; exceptions and rest protect streaks. Correcting a past entry updates the correct weeks without duplicating a claim. No calorie or weight outcome earns XP.

**Required tests.** Every [rule vector](TEST_STRATEGY.md), full vs incremental replay, conflicting commands, schedule revision boundary, expiry/midnight, savings and reduced-motion paths.

**Done when** someone can complete and claim either stream’s wish in an offline fixture and trace each credited date to evidence.

## M4 — Private data lifecycle

**Objective and scope.** Make the journal portable and recoverable before expanding the native surface. Depends on M1–M3.

**Tasks.** Implement strict local SAF gateway, bounded pack reader, USDA/OFF catalog indexing and version switching, source/license UI and manual GTIN search. Implement logical ZIP export, encrypted backup envelope, staging validation and confirmed transactional replace. Add delete-all, process-death cleanup, old-schema fixture conversion, capacity estimates and errors for unavailable local volume.

**Acceptance.** Export, erase and restore the populated journal on a fresh profile. Compare canonical records, historic calculations and claims. Failed import leaves the old data intact. Update/remove an optional OFF pack without changing a meal.

**Required tests.** Archive authentication/truncation, format and FK validation, migration fixtures, disk full/worker interruption, cloud provider rejection, isolated build reproducibility including crypto, backup/transfer exclusions and post-wipe checks.

**Done when** both encrypted and explicitly plain local exports pass round trips, data reset is verified and each distributable pack has corresponding license/source records.

## M5 — Grounded coach without weights

**Objective and scope.** Explain real progress before attempting generation. Depends on M2–M4.

**Tasks.** Implement topic-specific opt-ins, fixed query ports, read-only snapshot builder, localized route parser/chips, stateful follow-up resolution, deterministic templates, evidence sheets, coverage and constraint filtering. Add review-only actions through normal editors. Reuse facts for the Today insight. Make history ephemeral.

**Acceptance.** The three example questions (sessions, nutrition coverage and wish ETA) work on local fixtures with an explanation for every number. Missing history yields clarification. A question cannot change a policy, journal entry or target.

**Required tests.** Fact/query/renderer contract, consent withdrawal, stale data, missing targets, estimates vs observed dates, hide-calorie preference, invalid topic and no-model end-to-end UI.

**Done when** adding weights is optional for every primary journey and a cold install passes core offline/privacy tests.

## M6 — Local text model qualification and integration

**Objective and scope.** Add natural-language routing/selection without affecting data ownership. Depends on M4–M5. Start with the physical-device spike from [AI feasibility](AI_FEASIBILITY.md).

**Tasks.** Build pinned llama.cpp arm64 CPU backend and isolated PFD loader. Validate official Qwen digest, prompt template and grammar; implement cancel/death/thermal handling, install/activate/remove UI, strict output validator and fallback. Benchmark against fixed parser and fixtures. Try 1.5B if needed for quality or LiteRT-LM/Gemma CPU if the primary runtime fails. Recheck redistribution terms for each tested file.

**Acceptance.** On a clean no-Play Pixel 8 answer scoped questions, parse a follow-up and recover from removal, malformed output and worker death. Pass end-to-end latency, PSS, correctness and heat gates for each digest marked supported.

**Required tests.** Real-device benchmark matrix, malicious/unexpected model selections, rapid install/switch/close/cancel, cold vs warm memory, no network or personal logging, no-weights regression.

**Done when** measured results and a supported digest/runtime are recorded or the feature is explicitly marked deferred with working templates. A deferred model is an open product capability, never a passing model result.

## M7 — Scan, OCR and meal recognition

**Objective and scope.** Accelerate logging while preserving manual correction. Depends on M2, M4 and the bounded worker/import contract from M6 (a passing LLM is unnecessary).

**Tasks.** Implement lifecycle-bound CameraX, ZXing decode/typed fallback, Tesseract label capture with basis review, metadata-stripped file import and cleanup. Integrate standalone LiteRT CPU food classifier and curated label aliases, multi-crop editing, portion assistant and per-item catalog match. Evaluate detector only if it improves held-out mixed-plate results. Keep multimodal captions behind a separate PoC decision.

**Acceptance.** Known barcode resolves offline; unknown code retains digits in manual form. Correct a suggested food, add a missing ingredient, adjust portion and save; stored totals equal the manual calculation. Denial, low confidence, unsupported language or missing model preserves the draft and manual path.

**Required tests.** Scan rotation, check digits/UPC normalization, noisy OCR per-100 vs serving labels, food recall/abstention and memory gates, photo bounds/EXIF cleanup, draft/permission recovery and accessibility of portion correction.

**Done when** each enabled recognition path has an artifact/license and target-device evidence. Record disabled paths separately; scan and manual entry ship independently.

## M8 — Production release gate

**Objective and scope.** Qualify the complete supported experience and disclose deferred capabilities. Depends on M0–M7 outcomes.

**Tasks.** Run fresh GrapheneOS no-Play journeys with and without approved models, benchmark the five-year journal, verify recovery/backup rules and erase, review native packaging on 4 KiB and 16 KiB page-size test configurations, OS hardening compatibility and accessibility. Compare independent unsigned builds. Package licenses, dataset transformations, corresponding source, digest matrix, install/import guide and SBOM. Sign the verified release separately.

**Acceptance.** Every enabled feature passes [Test strategy](TEST_STRATEGY.md). No runtime network permission or transmission. Core flows meet accessibility and performance budgets. Claims about hardware support match measured builds. Record missing hardware gates as blockers to advertising that configuration.

**Required tests.** Release APK static and dynamic privacy audit, offline journey, migration/restore, TalkBack/200% font/reduced motion, screen resize, long session under memory pressure, native cancellation, SBOM and reproducibility checks.

**Done when** release evidence is reproducible, known limitations match settings and release notes, and rollback to templates/manual entry is verified on the shipping configuration.
