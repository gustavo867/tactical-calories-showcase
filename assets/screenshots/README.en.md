# Screenshot notes

[Back to the showcase](../../README.en.md) · [Português](README.md)

These images are existing application development/validation captures, copied without visual alterations. They show the Brazilian Portuguese interface and illustrative entries. They were visually inspected before inclusion; no email addresses, credentials, or account identifiers were visible in the selected captures.

The images come from different development iterations. Navigation labels and available actions may differ between them; they are not a single synchronized release capture. In particular, the dashboard is an earlier iteration, while the offline detail demonstrates a later flow.

| Showcase file                                                  | Capture path in the private project                | What it shows                                                                |
| -------------------------------------------------------------- | -------------------------------------------------- | ---------------------------------------------------------------------------- |
| [daily-dashboard-desktop.png](daily-dashboard-desktop.png)     | `docs/screenshots/meal-cards/desktop.png`          | Calorie balance, macro progress, meals, and hydration section                |
| [meal-actions-desktop.png](meal-actions-desktop.png)           | `docs/screenshots/issue-100-diario-desktop.png`    | Copy meal, save favorite, meal notes, and water intake                       |
| [food-search-mobile.png](food-search-mobile.png)               | `docs/validacao/issue-102/issue-102-busca-390.png` | Food search and selection summary                                            |
| [favorite-meals-mobile.png](favorite-meals-mobile.png)         | `docs/screenshots/refeicoes-favoritas/mobile.png`  | Saved meal combination and management actions                                |
| [offline-food-entry-mobile.png](offline-food-entry-mobile.png) | `docs/evidence/issue-116-offline-mobile.png`       | Offline individual entry alongside a selection requiring online confirmation |

The offline capture includes a long food-detail page and a separate action for recording an individual entry offline. The presence of a selection on that page does not mean batch confirmation is supported offline.

The source paths document provenance; the private repository is not required to view any image in this showcase. No source repository history, test reports, or user-data exports were copied.

## Captures added for v0.7.0

The captures below were inspected before publication. Catalog and My foods files were retrieved through `gh` from the confirmed product `main`, commit `8f5a586480bca3ac2c049ad40b0c7d7531c4a734`. The two meal-copy captures are the local originals from PR #136, also published in the comment linked below. Illustrative dates and names belong to test scenarios; they do not indicate the release date.

| Showcase file                                                          | Product source                                          | Content                                                                    |
| ---------------------------------------------------------------------- | ------------------------------------------------------- | -------------------------------------------------------------------------- |
| [catalog-tilapia-desktop.png](catalog-tilapia-desktop.png)             | `docs/evidence/issue-133-ampliacao/tilapia-desktop.png` | Prepared tilapia, USDA attribution, and 50 g preview on desktop            |
| [catalog-tilapia-mobile.png](catalog-tilapia-mobile.png)               | `docs/evidence/issue-133-ampliacao/tilapia-mobile.png`  | The same flow on mobile                                                    |
| [catalog-mustard-desktop.png](catalog-mustard-desktop.png)             | `docs/evidence/issue-133/mostarda-desktop.png`          | Prepared mustard with USDA attribution on desktop                          |
| [catalog-mustard-mobile.png](catalog-mustard-mobile.png)               | `docs/evidence/issue-133/mostarda-mobile.png`           | The same flow on mobile                                                    |
| [copy-yesterday-desktop.png](copy-yesterday-desktop.png)               | `.codex/tmp/issue-132-shortcut.png`, PR #136            | Shortcut in the selected date’s lunch card                                 |
| [copy-yesterday-review-desktop.png](copy-yesterday-review-desktop.png) | `.codex/tmp/issue-132-final.png`, PR #136               | Review with source October 1 and destination October 2, 2026               |
| [my-foods-desktop.png](my-foods-desktop.png)                           | `docs/validation/assets/issue-134.png`                  | Editing owned public and private items; deletion only for the private item |

Private source references: [yesterday’s meal copy](https://github.com/gustavo867/tactical-calories/pull/136#issuecomment-6695937164), [My foods](https://github.com/gustavo867/tactical-calories/pull/137#issuecomment-6696234174), and [promotion selection](https://github.com/gustavo867/tactical-calories/pull/142#issuecomment-6716315339). Full PNGs are included in this showcase and do not depend on those links for viewing. The images demonstrate validation screens, without establishing hosted deployment status, continuous interaction, or physical-device use.
