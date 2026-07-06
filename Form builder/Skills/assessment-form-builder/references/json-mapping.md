# JSON structure & parse script

The export is a serialized .NET object. Lists are wrapped as
`{"$type": "...", "$values": [...]}`. Always read through `$values`.

## Top-level keys you need

- `Reference` — source reference (candidate for the new Référence)
- `Label` — source title (ignored; the user gives the new title)
- `CampaignType.Reference` → campaign type (see type-mapping.md)
- `ShowTableOfContents` (bool) → "Afficher le sommaire"
- `Header.Groups.$values[]` → header groups; each `.Label` + `.Fields.$values[].Label`
- `Steps.$values[]` → steps; each `.Label`, `.Order`, `.Modules.$values[]`
- Ignore for this skill: `StateTimelines` / `ShowStateTimeline` (the "Fil d'ariane"
  workflow is intentionally NOT replicated).

## Per-module keys

- `.Reference`, `.Label`
- `.SpecificType.Reference` → Type spécifique (see type-mapping.md)
- `.Legend` — HTML/entity-encoded legend text (decode `&#39;`→`'`, `&amp;`→`&`, etc.)
- Competency modules only: the settings block listed in competency-config.md
  (`GroupBySubdomain`, `ShowLevel`, `ScoreGradeView`, `CompetencyDirectoryType`, …)

## Ready-to-run extractor

Run this against the JSON file to print a compact build plan. Adjust the path.

```python
import json, html, sys

path = sys.argv[1] if len(sys.argv) > 1 else "form.json"
d = json.load(open(path, encoding="utf-8"))

def vals(x):
    return x.get("$values", []) if isinstance(x, dict) else (x or [])
def ref(x):
    return x.get("Reference") if isinstance(x, dict) else x
def dec(s):
    return html.unescape(s) if isinstance(s, str) else s

print("REFERENCE:", d.get("Reference"))
print("CAMPAIGN :", ref(d.get("CampaignType")))
print("SOMMAIRE :", d.get("ShowTableOfContents"))

hdr = d.get("Header") or {}
for g in vals(hdr.get("Groups", {})):
    print("HEADER GROUP:", g.get("Label"))
    for f in vals(g.get("Fields", {})):
        print("   field:", f.get("Label"))

COMP_BOOL = ["GroupByDomain","GroupBySubdomain","GroupByType","GroupByActivity",
    "GroupByMission","GroupByProcess","EnableCompetencyReviewPortal","ShowComment",
    "UseGroupComment","ShowLevel","ShowContext","ShowAutoEvaluationScoreGrade",
    "ShowTargetScoreGrade","ShowN1Score","ShowN1TargetScore","UseN1TargetScore",
    "UseN1ScoreDefault","UseInActionPlanModule","ShowActivity","ShowQualification",
    "HideRequiredLevel","ScoreIsCompulsory","CollapsedByDefault","UseNaDefault",
    "HideNaLevel","ShowAverageScore"]

for s in vals(d.get("Steps", {})):
    print(f"\nSTEP [{s.get('Order')}] {s.get('Label')}")
    for m in vals(s.get("Modules", {})):
        st = ref(m.get("SpecificType"))
        print(f"  MODULE ref={m.get('Reference')} type={st} :: {m.get('Label')}")
        if m.get("Legend"):
            print("    legend:", dec(m["Legend"])[:100], "...")
        if st == "Competency":
            on = [k for k in COMP_BOOL if m.get(k) is True]
            print("    comp.bools ON:", ", ".join(on))
            print("    ScoreGradeView =", m.get("ScoreGradeView"),
                  "| DescriptionView =", m.get("DescriptionView"),
                  "| Directory =", ref(m.get("CompetencyDirectoryType")) or m.get("CompetencyDirectoryType"))
```

Use the printed plan to drive the UI steps. Keep decoded legend text for the
Légende editor.
