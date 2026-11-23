# Local data contract

Room owns one private single-profile journal. Each installed public catalog is a separate rebuildable, immutable SQLite database, opened read-only after indexing. No cross-database foreign keys. Identifiers are UUID strings generated before a command, except natural/composite keys listed below. Every editable aggregate has an integer revision for compare-and-set. Use SQLite INTEGER (signed 64-bit), TEXT and explicit nullability. Dates are ISO local dates, instants are Unix milliseconds and zones are IANA strings. Currency has its ISO minor-unit scale. Reject NaN, overflow, impossible dates and negative quantities; use checked decimal/integer math before storage.

## Authoritative schema

In the table below, fields ending in Id reference the named parent where it exists in the journal. A question mark means optional. Child order is unique within parent. A write increments the revision on its aggregate root. Codes/enums are stable serialized strings. A schema migration owns any renaming.

| Table and key | Required payload, relationships and invariants |
|---|---|
| Profile(singleton) | enrolledDay, scoringZone (fixed for this journal), preferredUnits, objectiveCode, constraints (bounded enum set), permittedCoachDomains, hideCalories, gamificationEnabled, journalRevision. Opt-out hides rewards; eligible logs stay in the journal and progress can be reconstructed if opted back in. |
| WeeklyPolicy(effectiveMonday) | frequency 1..4, recurringRest weekday mask (0..6 days), ruleVersion=1. Initial row applies from Monday of enrollment week, eligibility starts at enrolledDay. Later revisions start on a future Monday. |
| DayException(day) | kind REST/ILLNESS/TRAVEL/PERSONAL. At most one reason per date. |
| NutritionTarget(effectiveDay) | nullable energy milli-kcal, protein/carbs/fat mg, nonnegative; a cleared target is null. Initial targets optional. Changes default to today; explicit backdating shows impact. |
| BodyMeasurement(id) | day, recordedAt, bodyMassG >0. Unique day; a second entry edits with confirmation. |
| Exercise(id) | displayName, normalizedName, kind LOADED_REPS/BODYWEIGHT_REPS/TIMED, builtInKey?, archived. Seed with stable IDs; referenced entries are archived rather than hard-deleted. |
| Plan(id), PlanItem(id) | name, archived; item has planId, exerciseId, position, plannedSets, targetReps? or targetSeconds?, loadG?, restSeconds. Editing a plan changes future starts only. |
| Session(id) | planId?, planNameSnapshot?, startedAt, finishedAt?, sourceZone, day, status DRAFT/FINISHED, timerDeadlineMs?, pausedRestRemainingMs?. Finish validates a completed set; at most one draft via a partial unique index. |
| SessionExercise(id), WorkoutSet(id) | sessionId, exerciseId, name/kind snapshot, position, restSeconds; set has sessionExerciseId, position, setKind WORK/WARMUP, completed, reps?, seconds?, externalLoadG?. Zero external load is valid. Use kind to select reps vs seconds. Negative load unsupported in v1. |
| CustomFood(id), FoodServing(id) | name, prepState?, GTIN14?, basis MASS/VOLUME, nutrient vector per 100 g or 100 ml, source MANUAL/LABEL, archived; serving has foodId, label and positive amount in base units. |
| Recipe(id), RecipeItem(id) | name, cookedYieldMg >0, archived; item has recipeId, position and the same snapshot/confirmed quantity fields as a meal item. Ingredients must be individual foods; no recursive recipes. |
| Meal(id), MealItem(id) | meal has recordedAt, sourceZone, day, mealType and status DRAFT/SAVED. Item has mealId, position, immutable-at-save food/recipe snapshot, amount, optional bounds and provenance as specified below. Saved meal needs at least one positive confirmed item. |
| NutritionDay(day) | explicit complete flag and confirmedAt. Saving, moving, editing or deleting an item clears completeness for every affected date until reconfirmed; disallow complete with no saved meals. |
| Wish(id) | name, description, stream TRAIN/MEAL, startDay, difficulty, activation inputs (B numerator/denominator, P, H, sample weeks/counts, proposedTarget), acceptedTarget, targetOrigin, ruleVersion, status ACTIVE/ARCHIVED/CLAIMED, closedDay?, pinned. At most one pinned active wish. |
| Claim(wishId) | unique receipt with claimedAt, day, progressSnapshot and targetSnapshot. Write with status change. No financial postings. |
| ReviewDismissal(week,stream) | suppress repeat prompts within the current review week. |
| SavingsPot(wishId), SavingsEntry(id) | currency, optional targetMinor; entry has potWishId, day, recordedAt, signedAmountMinor. Same currency throughout. Check running balance ordered by day, recordedAt, id on every edit. |
| Installation(kind,digest) | local-only index for CATALOG/TEXT/VISION/OCR with semantic version, source/license ref, app-controlled relative path, size and state. At most one active per slot (catalog slots include source/region). Reconstruct/check files on startup. Exclude this table from journal backups. |

DataStore contains theme mode, reduced-motion override and haptic preference. These device-local options reset to defaults on a new installation. On restore retain business row revisions and advance journalRevision from its current local value. User-selected units affect formatting. Use canonical grams for bodyweight/exercise loads, milliseconds for instants, seconds for timers and **mg or µl** for food quantities. A nutrient vector contains nullable energy in milli-kcal and nullable protein, carbohydrate, fat in mg per reference basis. Null means unknown; zero means declared zero. A food with no nutrient values can be saved explicitly as an incomplete item. Disallow fake zero defaults. For manual kcal overrides store original vector, corrected vector, edited fields and source USER_CORRECTION. Limit labels to 160 characters, notes to 1,024; require positive portions ≤100 kg or 100 L per item. Parse locale-aware display at the boundary; serialize base units as integers.

~~~mermaid
erDiagram
    PLAN ||--o{ PLAN_ITEM : prescribes
    EXERCISE ||--o{ PLAN_ITEM : identifies
    SESSION ||--o{ SESSION_EXERCISE : contains
    SESSION_EXERCISE ||--o{ WORKOUT_SET : records
    MEAL ||--o{ MEAL_ITEM : freezes
    RECIPE ||--o{ RECIPE_ITEM : contains
    WISH ||--o| CLAIM : redeems
    WISH ||--o| SAVINGS_POT : tracks
    SAVINGS_POT ||--o{ SAVINGS_ENTRY : records
~~~

## Catalog and historical nutrition

Catalog header fields: formatVersion=1, source, upstreamRelease, region, locale, rowCount, attribution, licenseId. Each record has unique sourceKey, name, aliases, preparation state, basis, nutrients, optional serving conversions and validated barcode strings. Require 100% of rows to pass structural/range checks and at least one searchable record; unknown nutrients are valid. Reject duplicate source keys. Preserve conflicting barcode matches for user selection. Index normalized names with SQLite FTS4, stable key and GTIN14 with B-trees. Normalize valid EAN-8, UPC-A and EAN-13 by left-padding to GTIN14 after check-digit validation; never cast to a number. Rank exact barcode, recent selections, exact normalized name, token prefix then stable source/key. Match a returned candidate explicitly. Index journal by session/meal day and status, exercise history by exerciseId, child FKs/position, policies by effective date and active wishes by status. Bind search terms and escape FTS operators.

A saved snapshot contains display name, food preparation state, namespace + key (or recipe/custom UUID), upstream version, attribution/license, basis, per-100 nutrient vector and serving conversion used. For a recipe, derive per-100 g from ingredient totals and the actual cooked yield; freeze the result and recipe revision when logging. Store unrounded numerator/denominator for derived recipe composition to avoid intermediate rounding. Subsequent catalog or recipe changes only affect new selections. Meal edit preloads the snapshot; refreshing from a newer source requires a separate confirmed replacement.

PortionMethod = WEIGHED, LABEL_SERVING, HOUSEHOLD or USER_ESTIMATE. Convert a measured/labeled portion to amount q in mg or µl; a household unit needs a food-specific conversion. Never assume 1 ml = 1 g. For an estimate allow [lo,hi] with 0 < lo ≤ q ≤ hi, q chosen explicitly or user-confirmed midpoint. Record source of each endpoint. A range from a user is a scenario interval, not statistical confidence. Keep confirmed uncertainty in saved meals. Always label contributions from HOUSEHOLD and USER_ESTIMATE as approximate even when the user supplies just one value. For each nutrient n, contribution = n × q / 100000. Example: 150 g of an item with 200 kcal and 8 g protein per 100 g contributes 300 kcal and 12 g protein. Use the same formula with lo/hi; known exact quantities use q for both endpoints. Sum rational values before half-up rounding once for display (whole kcal and 0.1 g macros). Unknown ingredient makes that recipe nutrient unknown. For partial daily totals show known subtotal and missing item count; never compare an incomplete nutrient/day to a target as if complete.

For the USDA seed choose kcal energy explicitly by source nutrient ID (prefer Atwater specific 2048, then general 2047, then energy 1008 if declared), protein 1003, carbohydrate 1005, fat 1004. Do not add alternate energy measures. For OFF use declared kcal per 100 g/ml if basis is unambiguous, otherwise convert explicitly labeled kJ at 4.184 kJ/kcal with source recorded. For per-serving labels require a measured serving basis. Never infer a missing macro from calorie remainder. [USDA energy definitions](https://fdc.nal.usda.gov/Foundation_Foods_Documentation/) · [USDA downloads and field descriptions](https://fdc.nal.usda.gov/download-datasets/)

## Projections and analytics

Credit(stream,day) unique with XP, week, source evidence and rule version; Achievement(code) unique with earliest evidence date; weekly totals, levels, streaks, milestones and readiness follow [Gamification](GAMIFICATION.md). They may be recomputed on demand. Body trend = mean of available daily masses in each of two adjacent seven-day windows, compare only with ≥3 distinct dates per window; display sample count, delta and unit. A weekly training count is completed sessions; weekly credited days are separately named. PRs are maximum external load among completed work sets for the same exercise and reps, maximum reps for bodyweight or longest completed timed set. Tie goes to earliest start instant then set ID. Show actual reps and load; never infer a one-rep max. Plot date and measured value with gaps, allow an accessible history table.

Use ON DELETE CASCADE for aggregate children only, RESTRICT or archive referenced library entries. Clear optional plan reference if plan removed; keep its snapshot. Update meal/session, completeness, derived rows and revision together. On profile erase clear all. Foreign keys enabled, WAL, explicit Room transactions, bounded paginated history, optimistic conflicts reload current values. On a large restore build projections in staging and check referential integrity before final transaction. No raw journal input is sent to native inference.

## Evolution and crash tests

Schema v1 is the first public export; add tables incrementally before release, and preserve every schema once it has user data. Export schema JSON to version control. Forward-only explicit migrations backfill units, dates and versions deterministically. For a new derived algorithm, migrate authoritative data first and rebuild derived indices from versioned rules. Refuse unknown newer versions with a recovery message; never use destructive fallback or delete an unreadable source. Keep archive DTO migrators separate from Room migrations. Verify every supported historical schema → latest with MigrationTestHelper, full populated fixture, FK check, row counts and identical historical totals. Simulate disk-full, interrupted replace, duplicate command, stale editor and clock jumps. [Room migrations](https://developer.android.com/training/data-storage/room/migrating-db-versions)
