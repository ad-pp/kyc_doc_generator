# KYC Document Generator: Agent Handoff

## Purpose

This document is the technical and product handoff for the PhonePe merchant
onboarding document generator. It is intended for coding agents and engineers
continuing development in another environment such as GitHub Copilot.

Read this file before modifying workflow rules, legal text, DOCX layout, PDF
generation, persistence, Google Sheet logging, or GitHub Pages deployment.

## Current Production State

- Repository: `https://github.com/ad-pp/kyc_doc_generator`
- Live site: `https://ad-pp.github.io/kyc_doc_generator/`
- Production release: v8
- Production commit: `8605bdb`
- Production Pages workflow:
  `https://github.com/ad-pp/kyc_doc_generator/actions/runs/31179859523`
- Deployment branch: `main`
- GitHub Pages source: GitHub Actions
- Pages artifact path: repository root (`.`)

The project is a static browser application. There is no Node.js server,
database, build process, or package installation required for production.

## Repository Structure

```text
.
|-- index.html
|-- app.js
|-- README.md
|-- AGENT_HANDOFF.md
|-- google-apps-script.gs
|-- vendor/
|   |-- docx-8.2.2.js
|   |-- docx-preview-0.4.0.min.js
|   |-- html2pdf-0.10.2.bundle.min.js
|   `-- jszip-3.10.1.min.js
`-- .github/workflows/static.yml
```

### Key Files

- `index.html`: application shell, responsive CSS, preview modal and cache-
  busted `app.js` import.
- `app.js`: state, rendering, validation, local storage, Google Sheet logging,
  DOCX generation, DOCX preview, direct PDF generation and downloads.
- `google-apps-script.gs`: Google Workspace Sheet logging web app.
- `vendor/`: pinned browser libraries hosted with the site to avoid runtime
  CDN availability and corporate-network issues.
- `.github/workflows/static.yml`: deploys the repository root to Pages.

## Current Cache Version

`index.html` currently imports:

```html
<script type="module" src="./app.js?v=8"></script>
```

Increment this query value whenever production JavaScript changes. Otherwise,
mobile browsers may continue using an older cached application.

## Authoritative Requirements and Templates

The requirements evolved through multiple review documents. Newer documents
supersede older ones only where they explicitly change behavior or wording.

### Latest Change Document

```text
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/EDIT 5.pdf
```

Latest changes from this document:

- Add standalone optional `ACE OSV (MDF)` to ACE Partnership.
- Add standalone optional `ACE OSV (MDF)` to ACE Company.
- Fix final downloaded PDF vertical alignment/top blank space globally.
- Update Partner Resolution V5A and V5B output only; no additional UI input
  changes beyond the existing V5 selector.

### Latest Partner Resolution Templates

```text
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/Edit 5/Partrner Resolution V5a.pdf
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/Edit 5/Partrner Resolution V5b.pdf
```

These supersede earlier V5A/V5B files.

Important latest legal wording:

- Registered partnership beneficial-owner threshold: more than 10%.
- Unregistered partnership beneficial-owner threshold: more than 15%.
- V5A item 3 confirms all partners are natural persons.
- V5B item 3 covers one or more non-natural partners and submission of UBO
  identity proofs.
- Partnership Resolution captures meeting date only, not time.
- Generated letterhead is present on page 1 only. Declaration page 2 uses an
  explicitly empty Word header.

### Other Authoritative Document Templates

Unless a later change document explicitly modifies them, use:

```text
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/Document Formats to be followed_5th Aug/
```

Files:

```text
ACE MDF final Draft- OSV flow.pdf
BO_Declaration_Format_Company.pdf
BO_Declaration_Format_Partnership.pdf
Board Resolution Format- Company Final.docx.pdf
PhonePe Merchant Declaration Form- Salesforce.pdf
```

The same folder contains older V5A/V5B files, but the files in `Edit 5/` are
now authoritative for Partner Resolution.

## Current Workflow Matrix

### ACE + Partnership

Five choices:

1. Partner Resolution: Mandatory badge
2. BO Declaration: Optional badge
3. Partner Resolution + BO Declaration: Optional badge
4. Partner Resolution + BO Declaration + ACE OSV (MDF): Optional badge
5. ACE OSV (MDF): Optional badge

Standalone ACE OSV renders Entity Details and MDF fields only. It must not
require partner records, ownership, resolution inputs, or BO inputs.

### ACE + Company

Five choices:

1. Board Resolution: Mandatory badge
2. BO Declaration: Mandatory badge
3. Board Resolution + BO Declaration: Mandatory badge
4. Board Resolution + BO Declaration + ACE OSV (MDF): Optional badge
5. ACE OSV (MDF): Optional badge

Standalone ACE OSV renders Entity Details and MDF fields only.

### Salesforce + Partnership

Four choices, all shown with Mandatory badges:

1. Partner Resolution
2. BO Declaration
3. MDF
4. Partner Resolution + BO Declaration + MDF

Salesforce Partnership MDF designation is fixed to `Partner`.

### Salesforce + Company

Four choices, all shown with Mandatory badges:

1. Board Resolution
2. BO Declaration
3. MDF
4. Board Resolution + BO Declaration + MDF

Salesforce Company MDF designation is a dropdown:

- Director (default)
- Managing Director
- Company Secretary
- CEO

## Tracking and Google Sheet Logging

The application logs two business data points plus metadata:

- Agent mobile number
- New merchant: merchant mobile number
- Existing merchant: Merchant ID (MID)
- Timestamp and merchant status are also sent by the application

The Apps Script endpoint is configured in `SHEETS_URL` near the top of
`app.js`. Do not paste credentials, tokens or passwords into source code.

### Workspace Authentication Constraint

The corporate Google Workspace permits access only within the domain, not
public `Anyone` access. Logging therefore uses a hidden HTML form targeted to
a hidden iframe instead of `fetch()`.

This allows an existing corporate Google browser session to accompany the
request. If the user is not signed into the permitted Workspace account, the
log may silently fail while document generation continues.

### Logging Trigger

Logging occurs once per merchant session when the user first:

- Downloads DOCX
- Downloads PDF
- Downloads DOCX from preview
- Prints from preview

Opening a preview alone does not log usage.

Do not redeploy Apps Script unless its server-side code changes. Updating the
static app or payload values does not necessarily require script redeployment
when field names remain compatible.

## Browser Persistence

Storage keys:

```text
docgen_agent_v1
docgen_merchant_v1
```

- Agent mobile is stored separately and retained for normal new-merchant
  resets.
- Merchant workflow/entity/member/document data is auto-saved.
- Stored merchant KYC data is unencrypted browser local storage.

Reset levels:

- Section Reset: clears only the named card.
- Page Reset: clears fields on the current step.
- Start New Merchant: clears merchant data but keeps agent mobile.
- Global Reset: clears merchant data and agent mobile after confirmation.

When state fields are renamed or defaults change, add a migration in
`loadMerchant()` so existing devices do not retain invalid old behavior.

## Current Input and Validation Rules

### Common

- Agent mobile: valid Indian 10-digit mobile.
- New merchant: valid merchant mobile.
- Existing merchant: non-empty MID.
- Legal entity/firm name required.
- Registered/Main Office address required.
- Different principal/outlet address required when `Different` is selected.

### Letterhead

Letterhead remains optional via checkbox.

When enabled, it is generated consistently across document types:

- White background
- Bold black legal firm name on the left
- Bold black address line on the right
- Registered address always included
- Principal address also included when different
- Purple bottom rule
- Responsive font sizes based on text length
- Cells are `noWrap` to preserve the requested single-line design

Partner Resolution V5 uses the generated letterhead only on page 1. Its
Declaration page has an empty header.

### Partnership

- Registration type has no default and must be selected.
- UI values:
  - LLP / Registered Partnership
  - Non LLP / Unregistered Partnership
- Partner designation is fixed to `Partner`.
- Ownership is required even for Resolution-only workflows.
- All listed ownership must total 100%.
- Selected Resolution signers must hold at least 50% collectively; exactly
  50% is valid.
- Partner Resolution version defaults to V5A but remains editable.
- PEP defaults to No but remains editable.
- Meeting date required; meeting time removed for Partnership Resolution.

### Company

- Board Resolution allows one selected signer.
- Listed ownership must total 100%.
- OPC mode allows one listed director; non-OPC normally requires at least two
  listed members.
- BO authorised signatory can be external to the board using Name +
  Designation.
- Company listing status and stock exchange name are conditionally validated.

### BO

- Category 1 requires at least one owner above the applicable threshold.
- Category 2 uses the authorised signatory/senior official logic.
- PEP defaults to No but remains editable.
- BO date must match Resolution date when both documents are selected.
- Partner/director KYC includes PAN, DOB, residential address, POA type and
  number, nationality and ownership.
- Aadhaar stores/displays only the masked final four digits.

### MDF

- Signatory name, designation, PAN and mobile required.
- ACE email is optional; if supplied, it must be valid.
- Salesforce email is required.
- TAN and GSTIN are validated only when the corresponding registered option
  is selected.
- PwD details are required only when PwD = Yes.
- Salesforce additionally captures father name, KYC proof and entity nature.
- PEP defaults to No but remains editable.

## Document Builders

Located in `app.js`:

```text
buildPartnershipResolution
buildPartnershipBO
buildCompanyResolution
buildCompanyBO
buildMDF
```

`buildMDF` branches between ACE and Salesforce formats.

Do not create separate abbreviated legal HTML templates. DOCX is the source
artifact for preview and final PDF generation. This avoids Word/PDF legal-text
drift.

## DOCX Formatting Lessons

### Prevent Table Rows Splitting

Every generated `TableRow` must use:

```js
{ cantSplit: true }
```

This includes:

- Entity-information tables
- Beneficial-owner tables
- Senior-management tables
- Notes/reference tables
- Signature tables
- Letterhead table

A previous bug allowed page breaks in the middle of a row and cell. A global
audit confirmed all current rows use `cantSplit`.

### Explicit Page Boundaries

Do not rely on natural Word pagination for semantic section boundaries.

Examples:

- Company BO page 2 starts before `List of Senior Management Officials`.
- Partnership BO notes start after an explicit page break.
- Salesforce MDF uses explicit breaks to preserve its three-page structure.
- V5 Partner Resolution uses two Word sections, not just `PageBreak`, because
  page 2 must suppress the header.

### Expected Explicit Section/Page Structure

- Partner Resolution V5A: 2 pages/sections
- Partner Resolution V5B: 2 pages/sections
- Partnership BO: 2 pages with sample data; natural overflow is possible for
  unusually long or numerous records
- Company BO: 2 pages with representative data
- Board Resolution: 1 page with representative data
- ACE MDF: 2 pages
- Salesforce MDF: 3 pages

The user previously allowed natural pagination for unusually large content,
but explicit legal section boundaries must remain intact.

## PDF Generation Architecture

The PDF is not generated independently from legal text. The flow is:

1. Build DOCX using the relevant builder.
2. Pack DOCX to a browser Blob.
3. Render Blob using `docx-preview`.
4. Identify each rendered `section.docx` page.
5. Clone each page into an off-screen `pdf-export-root` anchored at top-left.
6. Render each detached clone to a canvas.
7. Add exactly one canvas image to each A4 jsPDF page.
8. Save via `html2pdf.js`/jsPDF.

### Critical PDF Bugs and Fixes

#### Purple/Illegible Header

Root cause: generic application CSS targeted every HTML `<header>`, including
headers generated inside DOCX preview.

Bad pattern:

```css
header { background: ... }
```

Current fix:

```css
.app-header { ... }
.app-header h1 { ... }
.app-header p { ... }
```

Never add unscoped CSS rules for `header`, `table`, `section`, `p`, `span`, or
other tags that can appear inside `.docx-wrapper`.

#### Large Blank Space Above Downloaded PDF

Root cause: PDF capture used the page element while it was positioned inside
the scrolling preview modal. html2canvas preserved layout/scroll offsets,
shifting document content down on the final PDF even though preview was
correct.

Current fix: clone each `.docx` page into `.pdf-export-root`, force:

```css
position: relative;
top: 0;
left: 0;
margin: 0;
transform: none;
width: 210mm;
min-height: 297mm;
```

Capture the detached clone, not the modal element.

#### Table Split Mid-Row

Root cause: Word rows could split and PDF conversion preserved the split.

Fixes:

- `cantSplit: true` on every DOCX row.
- Explicit semantic page breaks before major sections.
- One rendered DOCX page per final A4 PDF page; no automatic html2pdf content
  pagination across page boundaries.

## Validation Performed for v7 and v8

### Company BO Stress Dummy

Dummy files are available locally:

```text
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/Dummy Outputs for Verification/DEMO_Company_BO_Corrected.docx
/Users/arindam.das/Documents/Code Gen/Onboarding Tool/Dummy Outputs for Verification/DEMO_Company_BO_Corrected.pdf
```

Stress data included three directors, long names, long addresses and different
registered/principal addresses.

Validation results:

- DOCX ZIP integrity passed.
- 21 non-splittable rows were present in the Company BO dummy.
- PDF was exactly 2 A4 pages.
- Beneficial-owner table stayed on page 1.
- Page 2 began with Senior Management as required.
- Header was white, black, legible and contained both addresses.

### Cross-Document v7 PDF Regression

Representative PDFs generated and validated:

```text
Partner Resolution V5A       2 pages
Partner Resolution V5B       2 pages
Partnership BO               2 pages
Company BO                   2 pages
Board Resolution             1 page
ACE Partnership MDF          2 pages
ACE Company MDF              2 pages
Salesforce MDF               3 pages
```

All had:

- A4 dimensions (`595.3 x 841.9` PDF points)
- No browser page errors
- Zero dark-purple header-background leakage
- White header backgrounds
- Non-splittable rows

### Edit 5 v8 PDF Top-Alignment Regression

Representative final PDFs generated for all document types. The first
non-white content row was measured programmatically:

```text
ACE standalone Company MDF       y=37
ACE standalone Partnership MDF   y=37
Board Resolution                  y=37
Company BO                        y=37
Partnership BO                    y=33
Partner Resolution V5A           y=37
Partner Resolution V5B           y=37
Salesforce MDF                    y=37
```

The previous broken ACE PDF started around the middle of the page. The v8
acceptance criterion is top content `< 120` points; all files passed at
`33-37`.

## Recommended Validation Plan for Future Changes

Do not deploy legal-format or PDF changes based only on preview appearance.

### 1. Static Checks

```bash
node --input-type=module --check < app.js
git diff --check
```

Also search for:

- Unscoped document-like CSS selectors
- `TableRow` without `cantSplit`
- Obsolete workflow set IDs
- Stale cache query in `index.html`

### 2. DOCX Generation

Generate all affected document types with:

- Short data
- Long names and addresses
- Multiple partners/directors
- Same and different principal address
- Letterhead enabled and disabled

Validate:

- ZIP integrity with `unzip -t`
- Expected legal wording in `word/document.xml`
- Header contents in `word/header*.xml`
- Number and placement of `sectPr`/page breaks
- Every table row includes `<w:cantSplit/>`

### 3. Browser Workflow

Use Chromium/Playwright to verify:

- Each platform/entity document option list
- Conditional cards and fields
- Validation errors
- Local, page and global resets
- Preview modal
- DOCX download
- Direct PDF download
- No `pageerror` events

### 4. PDF Validation

For each affected format:

- Confirm A4 media box (`595.3 x 841.9` points)
- Confirm expected page count/semantic boundaries
- Render pages to images for visual review
- Check no row is split across page boundaries
- Confirm first non-white content row is near top (`< 120`, normally `33-37`)
- Check header background is white and text is legible
- Test mobile viewport as well as desktop

### 5. User Dummy Before Deployment

For significant format fixes:

1. Generate representative dummy DOCX and PDF.
2. Place under `Dummy Outputs for Verification/`.
3. Ask the user to approve.
4. Deploy only after approval.

## GitHub Pages Deployment

Normal release process:

```bash
git status --short --branch
git diff --stat
git diff --check
git log --oneline -10
git add <intended files only>
git commit -m "Concise change description"
git push origin main
gh run list --workflow static.yml --limit 1
gh run watch <run-id> --exit-status
```

Then verify:

- Live `index.html` references the new cache version.
- Live `app.js?v=N` contains the intended code.
- Vendor assets return successfully.
- A production browser smoke test downloads a valid PDF.

Do not change the Pages workflow unless deployment requirements change.

## Security and Privacy Considerations

- The GitHub Pages application is publicly reachable.
- Merchant KYC data is stored unencrypted in local storage.
- The Sheet endpoint is visible in browser source and is not a secret.
- The domain-restricted Google session is the current logging access control.
- Managed devices and clearing merchant data after completion are strongly
  recommended.
- Do not send document content to third-party preview services.
- Vendored libraries reduce supply-chain/runtime availability risk but should
  be reviewed before version upgrades.

## Known Limitations

- Browser DOCX preview is approximate; Microsoft Word may paginate slightly
  differently.
- Final PDFs are image-based page captures, so text search/copy and screen-
  reader quality are limited.
- Very long or numerous partner/BO rows may require additional pages despite
  sample page counts.
- BO records currently derive from partner/director records. Juridical BOs,
  foreign entities, minor BOs and Form 60 cases would benefit from a separate
  BO data model if required operationally.
- Google Sheet logging can silently fail when the agent browser lacks an
  authenticated corporate Google session.

## Important Engineering Principles Learned

1. Treat the latest approved sample as authoritative, but ask before applying
   wording that conflicts with previously approved legal thresholds.
2. Keep one legal-text implementation. Generate preview and PDF from DOCX.
3. Separate semantic document sections explicitly; do not trust natural page
   flow for compliance layouts.
4. Never allow application CSS to leak into rendered document HTML.
5. Never capture PDF pages from a scrollable modal in-place; clone and anchor
   them off-screen first.
6. Prevent all DOCX table rows from splitting.
7. Validate actual downloaded files, not only in-app previews.
8. Use stress data with long fields and multiple records.
9. Produce a user-reviewable dummy before deploying major format changes.
10. Preserve existing Sheet logging and unrelated user changes during every
    release.

---

# Update: v9 to v11 (supersedes the PDF Export and Cache Version sections above)

Cache version is now `?v=11`. Documents are generated at **A4**, not US Letter.

## PDF export was scroll dependent (v9)

`html2pdf.toCanvas()` forwards our `html2canvas` options but sets none of its
own scroll options (its `toContext2d()` does; `toCanvas()` does not). html2canvas
therefore defaulted `scrollX`/`scrollY` to the live page scroll, derived the crop
origin from `getBoundingClientRect() + scroll`, and then rasterised a cloned
document whose scroll was reset to 0. The two disagreed by exactly the current
scroll offset, so downloading after scrolling produced pages shifted down and cut
off, or entirely blank.

Rules that follow from this:

- Pin `scrollX: 0, scrollY: 0` and force `window.scrollTo(0, 0)` for the capture.
- `windowWidth` **must** equal the real `window.innerWidth`. html2pdf centres its
  capture container inside a `left:-100000px` overlay, so a synthetic width
  shifts the clone horizontally relative to the crop origin. Only `windowHeight`
  may be enlarged; the container is top aligned so it is never centred
  vertically. Enlarging it stops short viewports clipping page bottoms.
- html2pdf **deep-clones the source node into its own container**, so any CSS
  scoped to an ancestor such as `.pdf-export-root` is silently dropped before
  rasterising. Page geometry must be set inline on the node itself.
- Do **not** pin the section height. `section.docx` is `display:flex` /
  `flex-flow:column`, so a fixed height makes children flex-shrink and overlap
  (this made the BO letterhead overlap its title). Use `min-height` only.
- Draw each page top-left at exact A4 width, never rescaled and never centred.
  docx-preview does not auto-paginate, so a section can exceed one sheet; such a
  page is continued onto further sheets by offsetting the same image upward
  (`addImage(img, 'JPEG', 0, -sheet * 297, 210, drawHeight)`). Earlier builds
  shrank tall pages to fit, which silently distorted them.

## A4 instead of US Letter (v10)

`pageSetup()` emitted `12240 x 15840` twips (US Letter) while the exporter
rendered A4 sheets, so Word and PDF disagreed. It now emits `11906 x 16838`.
Side margins stay at 900 twips, leaving **10106 twips of content width**; the
widest table in any document is 10080, so everything still fits. Any new table
must stay at or below 10080.

## BO table overhang (v11)

The 9-column BO table overhung the right margin by 12.4mm (190.6mm rendered
against 178.2mm available). Three separate causes, found only by measuring the
DOM rather than reasoning about the source:

1. `boCols` summed to 8080 while the table declared 10080. Word honours the grid,
   so the two disagreed. The grid now sums to exactly 10080.
2. The table was laid out **auto**, so unbreakable tokens ("Nationality", PAN,
   Aadhaar) set min-content widths that summed past the text block. The DOCX now
   sets `<w:tblLayout w:type="fixed"/>` via `TableLayoutType.FIXED`, which fixes
   Word. **docx-preview 0.4.0 does not map `tblLayout`** and writes an inline
   `table-layout: auto`, so `section.docx table{table-layout:fixed!important}`
   is required to beat that inline style in preview and PDF.
3. At the document's 11pt body text the content cannot fit: the widest
   unbreakable token per column measures ~180mm before padding, against 178mm
   available. The BO table therefore uses 9pt (`size: 18`) via the `boH`/`boC`/
   `boU` helpers with 60-twip cell margins, and `boCols` is derived from measured
   token widths so nothing breaks mid-word. A split PAN or DOB is not acceptable
   on a KYC document.

Do not add `word-break: break-word`. It hides overhang by breaking tokens
mid-word and was briefly mistaken for a fix; only `overflow-wrap: break-word` is
set, which breaks solely when a token genuinely cannot fit.

## Validation harness

Lives at `../pdf-harness` (outside the repo, persistent - do not use the OS temp
directory, it gets cleaned). Playwright drives the real app, PyMuPDF measures the
resulting PDFs.

- `run.mjs` produces PDFs for 11 scenarios covering both entity types, all three
  document kinds, scroll 0 to 4000, viewport widths 390 to 1680 and heights 620
  to 1050. It injects a test-only hook because `app.js` is an ES module and
  exposes no globals.
- `assert_geom.py` asserts A4 page size, ink bounds on all four edges, margin
  symmetry, and that the final sheet ends cleanly. It recognises continuation
  sheets (predecessor ran to the sheet edge) and exempts them from top, right and
  skew checks.
- `findoverhang.mjs` reports any element extending past the section content box.
- `measurecols.mjs` measures the widest unbreakable token per column and reports
  per-column slack; `tight: 0` means nothing will break mid-word.

Lesson: assert **all four** ink edges. The original check only looked at the
topmost dark pixel, which a shrunk-and-centred or overhanging page still passes.
