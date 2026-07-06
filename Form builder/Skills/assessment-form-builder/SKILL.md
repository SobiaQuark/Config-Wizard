---
name: assessment-form-builder
description: >-
  Recreate a QuarksUp / Komeo career "Assessment Form" (Formulaire d'entretien
  carrière) inside the Komeo web app from a JSON export, using the Claude-in-Chrome
  extension. Takes two inputs: the form's JSON export (an AssessmentForm object,
  usually a file named like AssessmentForm_REF_date.json) and the desired form
  title (Libellé). Use this skill whenever the user wants to create, rebuild,
  replicate, duplicate-from-export, or import an assessment/career form, a
  "formulaire d'entretien", a competency form, or an "entretien carrière" into
  Komeo/QuarksUp from a JSON — even if they just say "crée ce formulaire",
  "recrée ce formulaire", "monte ce formulaire dans Komeo", or paste a form JSON
  and a title. Also trigger when they mention komeo.qupdev.net or a QuarksUp
  AssessmentForm and want it built via the Chrome extension. It drives the legacy
  WebForms UI efficiently through JavaScript rather than pixel-clicking, and ends
  with a verification pass and the link to the created form.
---

# Assessment Form Builder

## What this does and why it works this way

Komeo/QuarksUp is a **legacy ASP.NET WebForms** app: actions run through
`__doPostBack` with ViewState, editors open as **popups/iframes**, and there is
**no REST API or JSON import**. Pixel-clicking is slow and brittle. The reliable,
fast path is to **drive the page with JavaScript** through the Chrome extension:
navigate straight to the create URL, call the page's own JS functions
(`addStep`, `addModule`, `editModule`, `manageGroup`, `addItem`), and set field
values directly inside the modal iframe. Screenshots are used only for
checkpoints, not for every action.

## Inputs (ask for all three up front, before doing anything)

1. **Base URL** — which Komeo instance to build the form on (e.g.
   `https://komeo.qupdev.net` for dev, or the production domain). **Always ask the
   user for this at the very start** unless they already stated it in the
   conversation. Do not assume dev. Everything below is relative to this URL.
2. **JSON export** of the source form — an `AssessmentForm` object (top-level keys
   include `Reference`, `Label`, `CampaignType`, `Header`, `Steps`, `ShowTableOfContents`).
   It may be a large file (hundreds of KB); parse it with a script, never by eye.
3. **Desired title** — the `Libellé` to give the new form (overrides the JSON `Label`).

If any of the three is missing, ask for it before starting. A single upfront
question that collects the base URL (and confirms the JSON + title) is ideal — it
avoids building on the wrong instance.

## Prerequisites (check first, don't assume)

- The Chrome extension is connected (`list_connected_browsers`).
- The user is **logged in** to the Komeo instance. Navigate to the target and
  screenshot. If you land on the ADFS/quarksUp login page, **stop and ask the user
  to log in themselves** — never type passwords (see security rules). Resume once
  they confirm.
- You have the **base URL** from the user (input #1). Never default silently to dev
  — building on the wrong instance is costly. All paths below are relative to it.

## Golden rules (learned the hard way)

- **Prefer JS over clicks.** Set inputs/selects in the iframe and call the page
  functions. Reserve `computer` screenshots for verifying a step landed.
- **Editors live in an iframe.** Get it with
  `var ifr=document.querySelector('iframe'); var doc=ifr.contentDocument||ifr.contentWindow.document;`
  Read/write fields on `doc`, not the top document.
- **Closing DevExpress popups is unreliable.** Don't fight the `×`. To reset
  between modals, just **reload** `AssessmentFormDetail.aspx?id=<ID>&cultureLcid=1036`
  and re-open the section you need. This is the single most reliable reset.
- **Open a section** by finding its left-nav link by text and firing its postback,
  e.g. `__doPostBack('ctl00$uxMPH$uxDetailMenu$uxDetailTab5','')` for "Etapes et
  Modules". Tab numbers can shift, so resolve them by link text (helper below).
- **Wait ~1.5–2s** after each postback/`addX()` call before reading the iframe.
- **The `javascript_tool` output is scrubbed** if it echoes cookie/query-like
  strings. Return only what you need (keys, short values), not full URLs.

## Out of scope (intentionally not replicated)

- **Droits** (the per-module rights matrix: role groups × workflow statuses). New
  modules keep Komeo's default rights. Don't configure the Droits tab unless the
  user explicitly asks — the exported matrix is large and tied to workflow statuses.
- **Fil d'ariane** (the state-timeline / workflow). Left at default.

If the user wants either, treat it as a separate, explicit request.

## High-level flow

1. Parse the JSON into a structured plan (steps → modules with types; header
   group + fields; competency settings; legends).
2. Create the base form → capture the new numeric `id`.
3. Build the Entête (header group + fields).
4. Create the steps.
5. Create each module (with correct `Type spécifique`).
6. Configure the Competency module(s): Paramétrage + Légende. Add legends to any
   other module that has one.
7. Verify everything against the JSON.
8. Ask whether to activate, then output the form link.

Track these as todos so nothing is skipped.

---

## Step 1 — Parse the JSON

Run a script (Python) to extract a compact plan. Read
`references/json-mapping.md` for the exact JSON paths. Produce, per step, an
ordered list of modules with: `Reference`, `Label`, `SpecificType.Reference`
(→ maps to a "Type spécifique" value), `Legend`, and — for Competency modules —
the full settings block. Also extract the Header group label and its field labels.

Key JSON paths (`$values` are list wrappers):
- Steps: `Steps.$values[]` → `.Label`, `.Modules.$values[]`
- Module type: `.SpecificType.Reference` (e.g. `Competency`, `ActionPlan`,
  `TrainingWish`, `Training`, `CareerPlanWish`, `Signature`)
- Header: `Header.Groups.$values[]` → `.Label`, `.Fields.$values[]` → `.Label`

## Step 2 — Create the base form

Navigate directly to the create page (this is what the popup-blocked "Ajouter"
button opens — skip the button):

```
navigate → <base>/App/Career/AssessmentForm/AssessmentFormDetail.aspx?mode=1
```

Read the page (interactive) to get refs, then set:
- **Référence** (`uxReference` textbox): use the JSON `Reference`. If it already
  exists you'll get a duplicate error on save — in that case append a short suffix
  (e.g. `-2`) or ask the user. References are per-form identifiers.
- **Libellé** (`uxLabel` textbox): the **desired title** from the user.
- **Type de campagne** (`uxTypeInput` / combobox): map from JSON
  `CampaignType.Reference` (see `references/type-mapping.md`; default
  `1.AssessmentForm` = "Entretien carrière").
- **Afficher le sommaire** checkbox: check it iff JSON `ShowTableOfContents` is true.
  `form_input` may reject a string for checkboxes — click it with `computer` instead.

Click **Enregistrer**. The URL becomes
`AssessmentFormDetail.aspx?id=<ID>&cultureLcid=1036`. **Capture `<ID>`** — every
later step and the final link use it.

## Step 3 — Entête (header)

Open the Entête section (resolve its nav link by text "Entête"). Then:

1. Create the group:
   ```js
   manageGroup(0,'',true);
   ```
   Wait, type the group label into the "Libellé" field of the small
   "Gestion des groupes" modal (this one is on the top document, not an iframe),
   click **Enregistrer**.
2. Find the new group's add-item handler and open the field picker:
   ```js
   // find addItem(groupId) on the group row
   var a=[...document.querySelectorAll('a')].find(x=>/addItem\(/.test(x.getAttribute('onclick')||''));
   var gid=a.getAttribute('onclick').match(/addItem\((\d+)\)/)[1];
   addItem(Number(gid));
   ```
3. The "Elément" control is a **multi-select checkbox list** — you can add all
   fields at once. Open the dropdown (click its arrow), then check every wanted
   row by dispatching mouse events on its cell:
   ```js
   var wanted=[/* exact field labels from JSON, e.g. */ "Nom Prénom","Age", "..."];
   [...document.querySelectorAll('tr')].filter(r=>wanted.includes(r.textContent.trim()) && r.querySelector('td'))
     .forEach(r=>{var td=r.querySelector('td');['mousedown','mouseup','click'].forEach(t=>td.dispatchEvent(new MouseEvent(t,{bubbles:true,cancelable:true,view:window})));});
   ```
   Verify by reading the combo's text input value — it should list all selected
   labels. Close the dropdown (click the modal title), then **Enregistrer**.

Note: the header field order in the picker may differ slightly from the JSON;
that's cosmetic and reorderable with the row arrows if the user cares.

## Step 4 — Create the steps

Open "Etapes et Modules". For each step, in order:

```js
addStep();                    // opens the step modal (iframe)
```
Wait ~1.5s, then set the label and save inside the iframe:
```js
var ifr=document.querySelector('iframe'); var doc=ifr.contentDocument||ifr.contentWindow.document;
var el=[...doc.querySelectorAll('input[type=text]')].find(e=>/uxLabel/i.test(e.id));
el.value = STEP_LABEL; el.dispatchEvent(new Event('input',{bubbles:true})); el.dispatchEvent(new Event('change',{bubbles:true}));
```
Type stays "Avec modules" (default) unless the JSON step type says otherwise.
Click **Enregistrer** (top button in the modal), wait, then **reload** the detail
page and re-open "Etapes et Modules" before adding the next step. Reloading is the
clean reset; don't try to close the popup.

## Step 5 — Create the modules

You need each step's numeric id. After opening "Etapes et Modules", read them from
the grid's own handlers (robust, order-preserving):

```js
[...document.querySelectorAll('a')]
  .filter(a=>/addModule\((\d+)\)/.test(a.getAttribute('onclick')||''))
  .map(a=>({id:a.getAttribute('onclick').match(/addModule\((\d+)\)/)[1],
            label:(a.closest('tr')||{}).innerText?.replace(/\s+/g,' ').trim().slice(0,40)}));
```
Step rows carry `addModule(stepId)`; module rows carry `editModule(moduleId)`.

For each module of a step:
```js
addModule(STEP_ID);           // opens the module modal (iframe)
```
Wait ~1.5s, then set fields in the iframe:
```js
var ifr=document.querySelector('iframe'); var doc=ifr.contentDocument||ifr.contentWindow.document;
function setInput(id,v){var e=[...doc.querySelectorAll('input[type=text]')].find(x=>new RegExp(id,'i').test(x.id)); if(e){e.value=v;e.dispatchEvent(new Event('input',{bubbles:true}));e.dispatchEvent(new Event('change',{bubbles:true}));} return !!e;}
function setSel(id,v){var e=[...doc.querySelectorAll('select')].find(x=>new RegExp(id,'i').test(x.id)); if(e){e.value=v;e.dispatchEvent(new Event('change',{bubbles:true}));} return e&&e.value;}
setInput('uxReference', MODULE_REF);
setInput('uxLabel',     MODULE_LABEL);
setSel('uxTypeInput','2.Specific');                 // "Spécifique" for all specific modules
setSel('uxSpecificTypeInput', SPECIFIC_VALUE);      // e.g. '3.Competency' — see mapping
```
Click **Enregistrer**. Then **reload** the detail page and re-open "Etapes et
Modules" before the next module (reliable reset). `addModule`/`editModule` are
defined on that panel, so they must be (re)loaded first.

Duplicate module references across different steps are allowed (the app exports
them that way) — don't rename them.

See `references/type-mapping.md` for the full `Type spécifique` value table.

## Step 6 — Competency Paramétrage + Legends

For a **Competency** module, after it exists, open it with `editModule(moduleId)`
(it opens directly in edit mode), then:

1. Go to the **Paramétrage** tab inside the iframe:
   ```js
   var ifr=document.querySelector('iframe'); var doc=ifr.contentDocument||ifr.contentWindow.document;
   [...doc.querySelectorAll('a')].find(x=>/Paramétrage/i.test(x.textContent)).click();
   ```
   Wait, click **Modifier** to enter edit mode, then set every checkbox/radio/select
   from the JSON. The full JSON-field → control-id mapping is in
   `references/competency-config.md`. Set `.checked = true/false` directly (no need
   to dispatch change for plain checkboxes), set radios via the matching value, then
   **Enregistrer**. Re-read the states before saving to confirm nothing reset.

2. **Légende** (any module type that has one): open the "Légende" tab, click into
   the rich-text area, type the legend text (strip HTML entities like `&#39;` → `'`
   from the JSON first), **Enregistrer**.

Reload the detail page between modules.

## Step 7 — Verify against the JSON

Reload, open "Etapes et Modules", collect module ids, then for each module call
`editModule(id)` and read back `uxReference`, `uxLabel`, `uxTypeInput`,
`uxSpecificTypeInput` from the iframe — compare to the plan. For the Competency
module also re-open Paramétrage and confirm the key booleans/radios persisted, and
that the Légende text is present. Reading is safe; **do not click Enregistrer**
during verification (re-opening an edit form and saving can flip incidental
defaults). Reload to move on. Present a compact ✓/✗ table: base form, header +
fields, each step, each module + type, competency settings, legends.

## Step 8 — Activation + link

Ask the user whether to **activate** the form now (it's created Inactive/draft).
If yes, open "Informations générales" and click **Activer**. If no, leave it and
tell them they can click Activer later.

Finish by giving the **link to the form**:
```
<base>/App/Career/AssessmentForm/AssessmentFormDetail.aspx?id=<ID>&cultureLcid=1036
```

## Helper: open a left-nav section by its text

Tab indices vary; resolve by visible text:
```js
(function(txt){
  var a=[...document.querySelectorAll('a')].find(x=>new RegExp(txt,'i').test(x.textContent));
  var m=a && (a.getAttribute('href')||a.getAttribute('onclick')||'').match(/uxDetailTab\d+/);
  if(m){ __doPostBack('ctl00$uxMPH$uxDetailMenu$'+m[0],''); return 'posted '+m[0]; }
  return 'not found';
})('Etapes et Modules');   // or 'Entête', 'Informations générales', ...
```

## Common pitfalls

- Landing on the login page → ask the user to log in; never enter credentials.
- `addModule/editModule/addStep is not defined` → the "Etapes et Modules" panel
  isn't loaded; reload and re-open it first.
- Fields empty after `addX()` → you read too early (wait) or read the top document
  instead of the iframe.
- Checkbox `form_input` error → click with `computer`, or set `.checked` via JS.
- Popup won't close → reload the detail page instead.
