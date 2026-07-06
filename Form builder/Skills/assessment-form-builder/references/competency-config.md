# Competency module — Paramétrage mapping

The Competency module ("Type spécifique = Compétence") has a rich Paramétrage tab.
Open the module with `editModule(id)`, click the **Paramétrage** tab (inside the
iframe), click **Modifier**, set the controls below from the JSON, then click
**Enregistrer**. All ids are matched loosely with a regex on `element.id`
(e.g. `/uxShowLevel/i`) because the full ASP.NET id has a long prefix.

## Checkboxes (JSON boolean → control id)

Set `checkbox.checked = <json bool>`. Plain checkboxes don't need a change event;
saving posts their state.

| JSON field                     | control id (matches)          |
|--------------------------------|-------------------------------|
| `GroupByDomain`                | `uxGroupByDomain`             |
| `GroupBySubdomain`             | `uxGroupBySubdomain`         |
| `GroupByType`                  | `uxSplitTypes`               |
| `GroupByActivity`              | `uxGroupByActivity`          |
| `GroupByMission`               | `uxGroupByMission`           |
| `GroupByProcess`               | `uxGroupByProcess`           |
| (score-grade grouping)         | `uxGroupByScoreGrade`        |
| `EnableCompetencyReviewPortal` | `uxEnableCompetencyReviewPortal` |
| `ShowComment`                  | `uxShowComment`              |
| `UseGroupComment`              | `uxUseGroupComment`          |
| `ShowLevel`                    | `uxShowLevel`                |
| `ShowContext`                  | `uxShowContext`              |
| `ShowAutoEvaluationScoreGrade` | `uxAutoEvaluationScoreGrade` |
| `ShowTargetScoreGrade`         | `uxTargetScoreGrade`         |
| `ShowN1Score`                  | `uxShowN1Score`              |
| `ShowN1TargetScore`            | `uxShowN1TargetScore`        |
| `UseN1TargetScore`             | `uxUseN1TargetScore`         |
| `UseN1ScoreDefault`            | `uxUseN1ScoreDefault`        |
| `UseInActionPlanModule`        | `uxUseInActionPlanModuleInput` |
| `ShowActivity`                 | `uxShowActivityInput`        |
| `ShowQualification`            | `uxShowQualificationInput`   |
| `HideRequiredLevel`            | `uxHideRequiredLevelInput`   |
| `ScoreIsCompulsory`            | `uxScoreIsCompulsoryInput`   |
| `CollapsedByDefault`           | `uxCollapsedByDefaultInput`  |
| `UseNaDefault`                 | `uxUseNaDefaultInput`        |
| `HideNaLevel`                  | `uxHideNaLevelInput`         |
| `ShowAverageScore`             | `uxShowAverageScoreInput`    |

Some checkboxes (e.g. `ShowSeniority`, `ShowKeyCompetency`) only appear for certain
directory types and may be absent — skip any id you don't find.

## Radio groups (JSON enum → value)

Set the radio whose `value` matches, e.g.
`[...doc.querySelectorAll('input[type=radio]')].find(r=>r.name.includes('uxScoreViewInput') && r.value==='2').checked=true`.

| JSON field         | control name (matches) | mapping                                     |
|--------------------|------------------------|---------------------------------------------|
| `ScoreGradeView`   | `uxScoreViewInput`     | `None`→`0` (Aucun), `Detail`→`1` (Détaillée), `Tooltip`→`2` (Infobulle) |
| `DescriptionView`  | `uxDescriptionViewInput` | `None`→`0` (Aucun), `Detail`→`1` (Détaillée), `Tooltip`→`2` (Infobulle) |

## Selects

| JSON field                   | control id (matches)        | notes                                   |
|------------------------------|-----------------------------|-----------------------------------------|
| `CompetencyDirectoryType`    | `uxCompetencyDirectoryType` | `Job` = value `2` ("Référentiel emploi"). Read the live options to map other directory types. |
| `UseEmployeePreviousScore` / previous-score source | `uxPreviousScoreList` | leave as-is unless the JSON sets a previous-score source. |

## Notes

- `Grouping` in the JSON is a computed bitmask, not a settable field. The related
  UI number is **Seuil de regroupement** (`Portail Compétences` block, default
  `10`) which usually already matches — leave it unless the JSON clearly differs.
- **Options de filtre → Type** (Savoir / Savoir-être / Savoir-faire) usually
  defaults to all three, matching most exports. Only touch it if `CompetencyTypes`
  in the JSON is a strict subset.
- After setting everything, **re-read** the checkbox/radio states in the iframe
  and confirm they're what you intend **before** clicking Enregistrer — a stray
  auto-postback can reset a field.
