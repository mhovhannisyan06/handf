# HanDF

HanDF turns phone photos of handwritten Armenian math lecture notes into clean, compilable LaTeX PDFs, so students don't have to retype text and formulas.

**Status:** week 2 — test set and baselines done; next: fine-tuning.

## How it works (planned pipeline)

1. Photo of a notes page
2. Split the page into formula parts and Armenian text parts
3. Formulas → image-to-LaTeX model; Armenian text → Gemini
4. Assemble everything into one LaTeX document
5. Compile with XeLaTeX (Armenian font) → PDF

## Test set (my own data)

No public dataset has real phone photos of Armenian math notes, so I collected one:

- **40 pages** of real lecture notes (calculus, algebra, geometry, financial math), from different people and lighting
- **52 formulas** cropped from 10 pages, with correct LaTeX for each (drafted with AI, checked and corrected by hand)
- **30 lines** of handwritten Armenian text, with the correct transcription

## Baseline results

### Formulas (52 handwritten formulas)

| Model | Exact match | Avg char error (CER) |
|---|---|---|
| pix2tex | 0.0% | 99.9% |
| pix2tex + image cleaning | 0.0% | 98.1% |
| Pix2Text-MFR | **15.4%** | 40.7% |
| Pix2Text-MFR + image cleaning | 5.8% | 53.9% |
| Gemini (flash-lite) | **59.6%** | 10.2% |

Before comparing, LaTeX is normalized (spaces, `x^{2}` = `x^2`, `\cos{\alpha}` = `\cos\alpha`).

### Armenian text (30 handwritten lines, Gemini)

| Metric | Result |
|---|---|
| Character error rate (CER) | 24.4% |
| Lines read exactly right | 23.3% |

## What I learned

- **pix2tex cannot read handwriting at all** (0%): it was trained on printed formulas.
- **Pix2Text-MFR is the best open model** (15.4%), but far behind Gemini (59.6%).
- **Cleaning the background did not help**: it made Pix2Text-MFR worse (15.4% → 5.8%), probably because thin strokes get lost.
- **Gemini's typical formula mistakes:** similar-looking symbols (γ→y, b→6), small marks (prime ′, vector arrow), small fractions (1/4 → 114).
- **Gemini on Armenian text sometimes invents** a meaningful but wrong sentence, so the final PDF should flag uncertain lines for the student to check.

## Next steps

1. Fine-tune Pix2Text-MFR on the MathWriting dataset and compare with the baseline above
2. Build the page pipeline (page → parts → LaTeX → PDF)
3. Streamlit demo: upload a photo, download a PDF

## Files

| File | What it does |
|---|---|
| `01_rename_photos.ipynb` | Organizes the photos in Google Drive |
| `baseline.ipynb` | Formula baseline: pix2tex, Pix2Text-MFR, Gemini |
| `TextGeminiRead.ipynb` | Armenian text evaluation with Gemini |

Data (photos, crops, labels) is in Google Drive: `handf_data/`.
