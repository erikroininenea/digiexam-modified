# Scope, ethics & legal notes

## Scope of this research

- **What was done:** static, read-only analysis of installed binaries and their
  embedded resources for the purpose of architecture documentation.
- **What was not done, and will not be documented here:**
  - no execution, patching or modification of any binary;
  - no attempt to disable, weaken or bypass exam lockdown, monitoring, or any other
    security functionality;
  - no publication of "how to defeat" techniques — hypotheses touching the lockdown's
    trust model (phase-1 §8.5) are recorded as *conceptual, unverified* and explicitly
    require a vendor-authorized lab.

DigiExam is exam-integrity software used in real examinations. Documenting *what the
software does on a user's machine* is legitimate transparency research; anything that
would materially help someone cheat or tamper during a live exam is out of scope for
this repository.

## Copyright handling of recovered material

The embedded frontend bundle and stylesheet, and the string dumps, are derived from
**DigiExam Solutions Sweden AB's proprietary, signed software**. Handling:

| Material | Local preservation | Published to this (public) repo? |
|---|---|---|
| Analysis reports, methodology, manifests | yes | **yes** (original researcher authorship) |
| Extracted `index-COO2fUbz.js` (10.5 MB minified bundle) | yes (`recovered-assets/frontend/`) | **no** — excluded by `.gitignore` |
| Extracted `index-CvNvudRa.css`, `index.html` | yes | **no** — excluded by `.gitignore` |
| String dumps of the binaries | yes (`recovered-assets/strings/`) | **no** — excluded by `.gitignore` |

Rationale: preserving extracted artifacts locally for study is normal research
practice; **republishing the vendor's proprietary code on a public repository is
distribution of copyrighted material** and is avoided. The
[`MANIFEST.md`](../recovered-assets/MANIFEST.md) provides sizes, SHA-256 hashes and
byte offsets sufficient for anyone with lawful access to the same installer to
reproduce the extraction exactly.

## Legal context (non-advice note)

EU/EEA software directive provisions permit decompilation/reproduction in restricted
cases (interoperability, private backup, study where lawful access exists); provisions
and jurisdictions differ, and this project's outputs (a public report with hashes and
offsets rather than redistributed code) were chosen to stay on the conservative side.
Nothing here is legal advice; the researcher remains responsible for compliance and
for honoring any vendor takedown request.

## Vendor contact / takedown

The signed vendor is **DigiExam Solutions Sweden AB** (org.nr 556869-1504). If the
vendor objects to any content in this repository, the affected material will be
removed on request.
