# Product contract

E‑Trainer helps someone keep a routine and see how it relates to a wish they chose. A first-time user can record a set, log lunch, set a wish and ask how their week is going on a Pixel 8 in airplane mode. Installation of a model, provision of body measurements and permission to use the camera are all optional.

## People and journeys

| Person and situation | Path through the app | Observable outcome |
|---|---|---|
| A returning lifter wants a camera lens and trains around shift work | Skip measurements → select flexible weekly frequency → name the wish → inspect the calculation → accept → log the first session | One training day advances the wish. Two sessions on the same day still appear separately in history. |
| A home cook repeats family meals | Search the bundled catalog → enter ingredient masses → enter cooked yield → save recipe → log a portion → edit the recipe tomorrow | Today’s log keeps the original composition. Missing nutrients stay visibly unknown. |
| A traveler has no model or camera access | Type a barcode → select a local product or create from its label → review serving and save → ask about nutrition | The meal and coach work offline. An incomplete day is described as incomplete. |
| A user returns after an illness | Mark the affected dates as recovery → open the weekly recap → review an easier future schedule | No debt, catch-up workout or retroactive increase in the wish target. |
| A privacy-conscious lifter changes phones | Export an encrypted local archive → transfer it personally → preview counts on the new installation → confirm replacement | Sessions, meals, goals, settings that affect calculations and recorded savings survive. Model files can be imported separately. |

## First launch and repeat use

One local profile for an adult’s self-tracking. Ask for display units, scoring time zone and preferred training days per week. Offer editable defaults (metric, device zone at creation, two flexible training days). Nutrition logging, targets, personal objectives, constraints, coach context categories and gamification can each be skipped. No age, name, bodyweight, price or medical history is required. Preview a wish before activation. No permissions or model picker during onboarding.

| Surface | User actions and data | Empty, correction and success behavior |
|---|---|---|
| Today | Show today’s recorded activity, latest session, logged nutrition vs optional targets, current wish, weekly consistency, newest achievement and one evidence-linked coach card. Primary actions resume a draft or start a log. | Empty values say “No meals logged today” or “Set a target”. Expand a total to its entries and coverage. Return to the same date after editing. |
| Train | Pick a plan or start empty, search built-in/local exercises, create a custom movement, enter sets, reps, load and rest. Pause/resume a foreground rest timer. Finish explicitly. Browse history by date/exercise, charts and records. | Autosave draft sets. Duplicate a plan, reorder exercises, edit/delete sessions. Offer the previous load with its date. Never increase it automatically. |
| Eat | Log a meal from search, recent items, recipe, manual label, barcode or photo. Choose measured mass, labeled serving or a confirmed estimate. Review/edit calories and macros; explicitly mark a day complete. | Unknown barcode opens manual label entry with code retained. Mixed dishes support multiple components. Show missing nutrient fields; route uncertain photos to search. Corrections update totals immediately. |
| Wishes | Create training-day or meal-log-day challenges with custom name, reward and difficulty. Pin a wish, see milestones, XP, levels, weekly streaks and achievement evidence. Optionally enter a separate savings target and deposits. | Activation requires a rule preview. Rest/exceptions remain visible. Claiming a reward is a user action. Money is recorded by hand. No bank access or transfer. |
| Coach | Ask in ordinary language, choose topic chips, inspect dates, data coverage and assumptions. Open the relevant session/meal or review a proposed schedule. | The model-free coach supports every core topic. Unsupported questions lead to a clarification or a list of supported topics. Conversation is session-only. |
| Settings & Data | Change units, future schedule, objectives, category consent, reduced motion and theme. Manage imports, view licenses/sources, change/remove models, back up, restore or erase. | Show active versions, progress, cancellation and rollback outcome. Wipe needs explicit confirmation. User-managed external archives remain under the user’s control. |

## Boundaries and acceptance

Training records accommodate bodyweight repetitions and timed exercises as well as loaded sets. Nutrition targets are user supplied and optional. Hiding calories removes those figures from Today, celebrations and coach context. Coaching covers patterns in the journal, preferences and app workflows. The app does not diagnose conditions, prescribe diets or treat a weight change as an achievement.

A day without logs is missing evidence. Predictions are labeled with their window and assumptions. No competitions, public feeds, purchases, penalty notifications or mandatory sign-in. All personal data processing takes place on-device. The only supported way to take personal records out of the sandbox is a user-confirmed export to local storage.

The [roadmap](ROADMAP.md) defines when each journey becomes available. The [design system](DESIGN_SYSTEM.md) defines interaction states and the [rule specification](GAMIFICATION.md) defines every awarded unit.
