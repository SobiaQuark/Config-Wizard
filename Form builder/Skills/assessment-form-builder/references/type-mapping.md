# Type mappings (Komeo AssessmentForm)

All values are the exact `<option value="...">` strings in the app's `<select>`
elements. Set them with `select.value = '...'`.

## Campaign type — `uxTypeInput` on the base-form create page

Map from JSON `CampaignType.Reference`.

| JSON `CampaignType.Reference` | UI label            | select value        |
|-------------------------------|---------------------|---------------------|
| `AssessmentForm`              | Entretien carrière  | `1.AssessmentForm`  |
| `Questionnaire`               | Questionnaire       | `2.Questionnaire`   |
| `PeopleReview`                | PeopleReview        | `4.PeopleReview`    |
| `CareerMission`               | Évaluation de missions | `5.CareerMission`|
| `Remuneration`                | Rémunération        | `6.Remuneration`    |

Default: `1.AssessmentForm`.

## Module "Type" — `uxTypeInput` inside the module modal

| UI label   | value        | when                                   |
|------------|--------------|----------------------------------------|
| Classique  | `1.Classic`  | plain (free) modules                   |
| Spécifique | `2.Specific` | every specialized module (all below)   |

For any module whose JSON `SpecificType.Reference` is set, use `2.Specific`, then
pick the `Type spécifique` below.

## Module "Type spécifique" — `uxSpecificTypeInput` inside the module modal

Map from JSON `SpecificType.Reference`.

| JSON `SpecificType.Reference` | UI label                          | select value            |
|-------------------------------|-----------------------------------|-------------------------|
| `Training`                    | Historique Formation              | `1.Training`            |
| `Table`                       | Tableau                           | `2.Table`               |
| `Competency`                  | Compétence                        | `3.Competency`          |
| `TrainingWish`                | Souhaits de Formation             | `4.TrainingWish`        |
| `CareerPlanWish`              | Souhaits de mobilité              | `5.CareerPlanWish`      |
| `Goal`                        | Objectifs                         | `6.Goal`                |
| `GoalFollowUp`                | Suivi des objectifs               | `7.GoalFollowUp`        |
| `CareerPlanWishFollowUp`      | Suivi des projets professionnels  | `8.CareerPlanWishFollowUp` |
| `TrainingWishFollowUp`        | Suivi des demandes de formation   | `9.TrainingWishFollowUp`|
| `Signature`                   | Signature                         | `10.Signature`          |
| `Questionnaire`               | Questionnaire                     | `11.Questionnaire`      |
| `ActionPlan`                  | Plan d'actions                    | `15.ActionPlan`         |
| `MobilityFollowUp`            | Suivi des mobilités               | `18.MobilityFollowUp`   |
| `RemunerationFollowUp`        | Suivi des rémunérations           | `19.RemunerationFollowUp` |
| `CareerMissionFollowUp`       | Missions                          | `21.CareerMissionFollowUp` |
| `DiplomaFollowUp`             | Suivi des diplômes                | `22.DiplomaFollowUp`    |
| `StatusGradeFollowUp`         | Historique des classifications    | `23.StatusGradeFollowUp`|
| `InterviewFollowUp`           | Suivi des entretiens              | `24.InterviewFollowUp`  |
| `PointerModuleFollowUp`       | Récupération des réponses précédentes | `25.PointerModuleFollowUp` |

If a JSON `SpecificType.Reference` isn't in this table, open the module modal once
and read the live `uxSpecificTypeInput` options
(`[...doc.querySelectorAll('select')].find(s=>/uxSpecificTypeInput/i.test(s.id))`)
to find the current value — the app version may have added types.

## Left-nav section tab ids (resolve by text, don't hardcode)

Observed on dev: Informations générales ≈ `uxDetailTab1`, Fil d'ariane ≈
`uxDetailTab3`, Entête ≈ `uxDetailTab4`, Etapes et Modules ≈ `uxDetailTab5`.
These can shift between versions — always resolve the tab by the link's visible
text (see the helper in SKILL.md) rather than trusting these numbers.
