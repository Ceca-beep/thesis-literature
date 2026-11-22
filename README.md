# Literature for the bachelor's thesis

*Clinical Note Intelligence: An NLP Pipeline for Structured Information Extraction and Prescription Anomaly Detection in Electronic Health Records*

| What | Where |
|---|---|
| Complete bibliographic information (30 sources) | [`references.bib`](references.bib) |
| Notes: what is relevant in each source and what it is used for | [`notes/literature_notes.md`](notes/literature_notes.md) |
| PDFs and links (`<bibkey>.pdf`) | [`papers/`](papers/README.md) |

## Workflow
1. Download the PDF of a source into `papers/` as `<bibkey>.pdf`.
2. Read it, tick its box in the notes, and correct or extend the entry with your own findings (especially items marked ⚠).
3. Cite it in the thesis with `\cite{bibkey}`; the thesis uses `\bibliography{references}` (or copy the file over `myBib.bib`).

## Data and privacy
MIMIC-III and the n2c2 2018 data need credentialed access and a data use agreement. Do not commit any of that data, and do not send MIMIC note text to hosted LLM APIs (PhysioNet restriction).

## Method
The literature was collected and first annotated with Claude Code; the notes were then restructured with Claude. All entries must be verified against the sources by the author.
