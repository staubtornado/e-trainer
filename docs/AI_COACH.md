# Coach and meal-assistance contracts

The coach answers from the same calculations the user can inspect on Today, Train, Eat and Wishes. Its two implementations share an evidence renderer. The model-free implementation is present on first launch. An installed model adds natural-language intent and follow-up interpretation. All numerical text, dates, trends, forecasts and suggested schedule values are rendered by Kotlin from a single journal snapshot.

## Read and inference ports

Contract sketches describe domain types, not Android components.

~~~kotlin
interface CoachFacts {
    suspend fun snapshot(query: FactQuery, access: ContextAccess): FactBundle
}
interface TextInterpreter {
    suspend fun interpret(request: IntentRequest): IntentSelection
    suspend fun select(request: EvidenceRequest): EvidenceSelection
    suspend fun cancel(turnId: TurnId)
}
data class FactQuery(val topic: Topic, val window: Window, val subject: LocalRef?)
data class FactBundle(val revision: Long, val asOf: Instant, val facts: List<Fact>)
data class Fact(val id: FactId, val metric: Metric, val kind: EvidenceKind,
                val value: TypedValue, val period: DateRange, val coverage: Coverage,
                val origin: CalculationRef)
~~~

**Allowed topics** are training consistency, exercise progress, body trend, logged nutrition, wish progress/ETA, achievements, plan review, preferences and app help. Window enum is TODAY, THIS_WEEK, LAST_WEEK, LAST_7_DAYS or LAST_28_DAYS. A prediction uses its specified closed-week sample even when the question window differs; label it. Resolve exercise or wish names locally against the user’s shortlist. Ask which one on ambiguity. Subject is a validated local reference scoped to that topic. No arbitrary predicates, joins, SQL strings, file paths, shell, calendar intents, tools discovered at runtime or write methods on these ports.

Fact service enforces access per call, regardless of model selection. Default to a local topic picker at first use; request category opt-in for training, nutrition, body and wishes separately. Deselection immediately cancels pending turns and clears prior handles/history. App help needs no grant. A hidden calorie preference removes energy values and derived energy conclusions even when nutrition access is granted. Savings, private notes, raw meal photos, raw tables, wish descriptions, free-text constraints and full transcripts never enter the model. Pass allowlisted constraint tags (e.g. avoid weight talk, recovery only), unit and tone through the local response policy. The model sees ephemeral subject/fact IDs and structured metric names/coverage from the current request only. Treat the user’s question as untrusted text, with a visible length limit and removal of control characters.

| Topic | Deterministic evidence | Eligible response or action |
|---|---|---|
| Consistency and exercise | Count finished sessions and distinct credited days separately; compare equal completed weeks or equal elapsed weekdays. Aggregate sets/reps/loads from completed work sets by exercise and day. Show comparison windows, PR and evidence dates. | “You finished {count} sessions {period}.” Only say “more frequent” if supported by paired comparable windows. Show training history. |
| Body objective | Use the measured means, delta and minimum coverage in the data contract. Respect opted-out metrics/weight talk. | Present neutrally as a recorded trend with coverage. No prescribed weight or target date. |
| Nutrition | Summarize logged kcal/macros, partial totals and completeness flags. For “below target”, count only days explicitly complete with a known effective target and fully known nutrient; if there is a range, its upper bound must be below target. | “Of {eligible} complete days, {below} had logged energy below your target.” Offer to review meals or open target editor. No automatic target or assumption that a meal was skipped. |
| Wish | Credited days, accepted target, milestones, actual pace, coverage and remaining weeks from the domain engine. | Show observed progress first, a separately labeled estimate with the rate/window next. If paused or insufficient history, return that reason. |
| Plan/constraints | Retrieve approved schedule, neutral dates, goal configuration preview and eligible review with exact policy revision. | Offer to open review, postpone or choose a gentler horizon. Rest takes precedence over a training suggestion. |
| Achievements | Earliest qualifying evidence from the award projection. | Explain how it was earned; link to qualifying records. No reward for calories, bodyweight or load. |

## Turn protocol

1. User message is limited to 512 Unicode characters; also enforce actual tokenizer budget before invoking a model. Display suggested topics for ambiguous/out-of-scope messages. Keep only current topic, window and subject aliases across turns, in memory. History visible in chat is ephemeral, cleared on leave/background/erase or permission change. A follow-up such as “and last week?” inherits the validated topic and subject.
2. Explicit topic chips bypass routing. Without weights use a localized deterministic phrase/keyword parser for the supported topics, date phrases and follow-ups. Prefer an exact match; ties ask for clarification. With weights, constrain decoding to enum fields and authorized subject aliases. Instruction: “Choose a topic and period for the question. Use only the supplied choices. Select clarify if more than one subject fits. Return a single JSON object.” Delimit and serialize input as data; never concatenate user text into instructions.
3. Validate exact schema, enums and IDs, intersect with grants, then request **one** read-only snapshot. Cap to 12 facts, one subject and one topic. Bound per-fact coverage; include observed/estimated/missing labels, unit and exact calculation reference. Keep deep-link IDs and unredacted labels in the renderer. Empty history yields a missing-evidence response, not a zero trend. Reject questions about diagnosis, extreme restriction or compensatory exercise; offer a neutral journaling or recovery action from the fixed catalog.
4. For selection use the instruction “Choose only supplied evidence IDs, compatible response style and an offered action. Return JSON. If insufficient, choose the missing-data style.” Request includes approved templates and allowable IDs; example output is below. Use grammar-constrained generation with strict JSON decoding, reject unknown fields, nested objects beyond schema, duplicate keys, foreign IDs, invalid enum combinations, trailing text and oversized output. Expose loading/cancel state; buffer tokens privately until selection is complete. No raw model prose reaches UI, speech or logs.

~~~json
{"template":"WISH_WITH_ETA","facts":["f1","f2"],"action":"OPEN_WISH"}
~~~

5. Verify f1 and f2 satisfy the template’s topic, period and evidence kinds. For example, WISH_WITH_ETA requires progress and nonmissing pace estimate for the same wish. An observed progress card reads “10 of 20 training days”, followed by “Estimate: about 4 weeks at 2.5 credited days per week” **only** when those values come from the engine. Each fact opens its evidence sheet listing eligible dates, numerator, denominator and assumptions. Metric formatting and plural forms are localized. Any invalid, unsafe, stale or timed-out response renders the rule-based selection for the same facts. Check revision before actions and refresh stale evidence before confirmation.
6. Actions are a closed enum: OPEN_HISTORY, OPEN_MEALS, OPEN_WISH, REVIEW_SCHEDULE, REVIEW_TARGETS, CONFIGURE_WISH, NONE. Open the ordinary edit form with its current revision and defaults. User reviews the diff and presses Save. The app invokes its normal command with optimistic concurrency. The model never issues a command or supplies a trusted parameter. Multiple taps or repeated turns cannot apply an edit twice.

For “I want a television and an easier challenge”, a CONFIGURE_WISH selection may preselect TRAIN/MEAL and GENTLE/STANDARD/EXTENDED from closed enums. Use the local question or user input to populate the editable title in the app. Never send the stored wish description to inference. The same domain preview as manual creation calculates P, B, H and the proposed count. Show the inputs and require explicit activation. Model-free users reach the identical form from a chip.

For the primary adapter use one active turn, serialized access, temperature zero/greedy decoding, published tokenizer/chat template, 1,024 total tokens per invocation, 96 generated tokens per invocation and an overall end-to-end deadline of 15 s. Tokenize before use; prune optional facts in template-compatible groups or revert to templates. Never truncate serialized JSON by character offset. Limit exactly two model invocations per turn, no tool-call loop. Cancellation or worker death always completes the turn. Mark failed digest inactive after reproducible initialization failures until user retries. Never silently switch to a remotely hosted model.

## Food assistance pipeline

~~~mermaid
flowchart TD
  Image[Private image or manual crop] --> Labels[Recognizer proposes ranked food names]
  Labels --> Choose[User selects food or searches catalog]
  Barcode[Barcode or label OCR] --> Choose
  Manual[Manual entry or saved recipe] --> Portion[Portion review]
  Choose --> Portion
  Portion --> Calc[Pure nutrient calculator]
  Calc --> Review[Review source, uncertainty and values]
  Review --> Save[Explicit save in one Room transaction]
~~~

| Port | Input → output | Decision owner |
|---|---|---|
| FoodRecognizer | Oriented bounded pixels and optional crop → up to three label IDs, scores, model digest and optional boxes per region | Model maps pixels to names only. Start with one user-selected crop per item. No model means show manual search immediately. |
| PortionAssistant | Chosen item, basis, known serving conversions, optional user reference → measured amount, labeled count × serving size, or editable low/high and representative amount | User confirms. No calibrated scale/density means no automatic mass. Ask “How much did you eat?” and offer labeled portions if available. |
| FoodResolver | Label/alias, typed query or validated GTIN → ranked local food/recipe candidates with name, preparation, source, basis and release | User chooses preparation and adds missing components. Unlabeled cooking oil/sauce is a clarification, never an assumed quantity. |
| NutrientCalculator | Immutable selected composition and confirmed quantities → energy/macros, ranges, unknown fields and provenance | Pure Kotlin fractions per [data model](DATA_MODEL.md). No logits, embeddings or model text are inputs. |
| MealEditor | Candidate items and calculated totals → corrected snapshot | User can add/remove foods, change portion/basis, correct label nutrients, abandon or Save. Save nothing from inference alone. |

After capture a suggested category can inform the portion question, but the exact serving conversion requires a resolved catalog item. Ask for item, serving and preparation in one review screen. Show “possible foods” with alternatives. Start conservatively: offer a highlighted first choice only for calibrated score ≥0.80 and gap to runner-up ≥0.20 on an in-scope crop. Otherwise ask the user to choose/type; all paths require confirmation. Treat raw softmax scores as rankings until calibrated. Package any changed thresholds, class mapping and calibration set revision with the approved digest. A confident wrong answer costs an extra correction; an abstention costs only a search. Track these in offline synthetic/consented evaluation, never via app telemetry.

Photo interpretation from a future multimodal model plugs into FoodRecognizer and follows the same contracts. V1 chat accepts text; asking about a meal photo opens the review flow. See [device gates](AI_FEASIBILITY.md) for the exact hardware and dataset acceptance criteria.
