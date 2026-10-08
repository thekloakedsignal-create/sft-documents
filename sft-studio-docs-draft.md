# SFT Studio Pro — Documentation

## Overview

SFT Studio Pro is a production pipeline for building supervised fine-tuning (SFT) datasets. You define a project, generate or convert training examples, audit them for quality, and export formatted training and validation files.

The pipeline runs through distinct stages:

**Define → Generate or Convert → Review → QA → Registry → Compile → Format → Export**

Each stage is separate on purpose. A generated example is not committed to the dataset until it passes review and QA. A QA flag is a recommendation, not an automatic rejection. A batch is not part of the final output until you select it in the registry.

| Studio area | What it does |
|---|---|
| Project Definition | Stores the system prompt, persona, constraints, and instruction structure used by every example |
| Synthetic SFT Batch Generator | Creates candidate batches from the project definition |
| Review / Edit | Lets you inspect, correct, or reject individual examples |
| Document-to-SFT | Converts source documents into candidate examples |
| TRiAD Alignment | Optionally adds Freedom/Truth/Kindness alignment examples |
| AI Quality Assessor | Audits batches for structural and behavioral problems |
| Dataset Registry | Gates final inclusion by QA status |
| Compile & Export | Combines selected batches and creates the validation split |
| Template Formatter / Output Preview | Normalizes the target template and previews serialized output |
| JSON / JSONL Export | Produces the final training and validation files |
| Studio Expert | Project-aware help and troubleshooting |

---

## 1. Project Definition & Setup

The project is the context everything else runs against. It stores the name, dataset structure, system prompt, target persona, constraints, and instruction format. Generation, conversion, QA, and export all read from it.

**To set up a project:**

1. Create a new project and choose its dataset structure.
2. Fill in the system prompt, target persona, constraints, and task definition. These values are injected into every example you generate or convert later.
3. Review the project details and edit them until they describe the behavior you actually want the model to learn.
4. Save the project. It is now the working context for all downstream stages.

**Notes:**

- Define the target behavior, not just the output format. The generator and converter both aim at what you describe here.
- Choose the instruction structure before generating — every batch inherits it.
- Constraints you set here survive into generated examples, so put recurring rules (tone, length limits, persona behavior) at project level rather than repeating them per batch.

---

## 2. Synthetic SFT Batch Generator

The generator creates candidate examples against the current project definition. Each batch has its own description, special instructions, temporary constraints, template, and requested size — separate from the project-level settings.

**To generate a batch:**

1. Open the Synthetic SFT Batch Generator with your project loaded.
2. Enter a batch description and any batch-specific instructions or temporary constraints.
3. Choose the instruction format and the requested batch size.
4. Run the generation. The resulting examples are presented individually for inspection.

**Notes:**

- Generate separate batches for different topics, personas, styles, or source conditions. Keeping batches small and identifiable means a bad run contaminates only that batch, not the whole dataset.
- Nothing in a batch is committed to the final dataset yet. Approval happens in review, and inclusion happens in the registry.

---

## 3. Review, Edit & Approve Examples

Every batch — generated or converted — goes through manual review before it can move forward. Batch review shows the batch status, metadata, and controls for each individual example.

**To review a batch:**

1. Open the batch review view and read a representative sample of examples. Do not judge a batch by its count alone.
2. Open any example to see its full conversation. Edit examples that are useful but contain fixable problems.
3. Reject examples that teach the wrong behavior — editing cannot save a fundamentally wrong target.
4. Approve the batch only when it represents the intended behavior.

**Notes:**

- Edits are example-level; they do not alter the project definition or other batches.
- Rejected examples never enter the final dataset. Approved batches move on to QA.

---

## 4. Document-to-SFT Conversion

Existing material — manuals, literature, transcripts, and other text — can enter the same pipeline through the converter instead of the generator.

**To convert a document:**

1. Open Document-to-SFT and provide the source document or text.
2. Configure the document type, number of dataset turns, layout, and conversion format. These settings control how the source is interpreted as conversational training examples.
3. Run the conversion. The output becomes a batch.
4. Send the converted batch through the same lifecycle as a generated one: review → QA → registry.

**Notes:**

- Conversion is not acceptance. Converted material goes through the identical quality-control path as generated material, so there is one review standard regardless of where examples originated.

---

## 5. TRiAD Alignment (Optional)

TRiAD Alignment imports pre-generated ethical-dilemma examples built around a balance of Freedom, Truth, and Kindness (FTK). The examples are constructed around the tension between the three values rather than a ranked priority, which suits conversational moral dilemmas. This step is optional and complements your subject-matter data — it does not replace it.

**To import alignment examples:**

1. Open TRiAD Alignment.
2. Choose the alignment batch size.
3. Choose the applicable batch template.
4. Run the import. The import script incorporates your project's system prompt into each alignment example, so the alignment examples speak with the same persona as the rest of the dataset.
5. The imported batch goes to the Dataset Registry like any other batch.

---

## 6. AI Quality Assessor

The Quality Assessor audits every example in a selected registry batch. It checks for incomplete format, template inconsistency, schema-integrity failures, persona/target drift, corrupted text, and broken examples.

**To run a quality audit:**

1. Select a registry batch and run the audit.
2. Review the flagged examples. Each flag shows the specific problem detected.
3. For each flagged example, choose one of four actions:
   - **Edit** — fix the problem and keep the example.
   - **Ignore** — dismiss the flag when the audit is wrong or the example is acceptable as-is.
   - **Reject** — remove the example from the batch.
   - **Regenerate Flagged** — replace flagged examples with fresh ones, then re-score.
4. Repeat regeneration and rescoring until the batch meets your quality threshold.

**Notes:**

- A flag is a recommendation for human examination, not an automatic rejection. You make the final call on every flagged example.
- **Deduplicate Dataset** can delete all flagged examples automatically. It saves time but bypasses human review, so it is not the recommended default.

---

## 7. Dataset Registry

The registry is the control room for final dataset assembly. Every batch is listed with its source, template, example count, and QA status.

**To assemble a dataset:**

1. Open the Dataset Registry and review the batch list.
2. Select the batches you want in the final output. Only QA-checked batches are selectable — unaudited batches are greyed out.
3. Your selections preserve per-batch provenance through the compile stage.

**Notes:**

- The QA gate is enforced in the interface: a batch that has not completed the audit cannot be selected. This keeps unaudited material out of final output by construction, not by discipline.

---

## 8. Compile & Export SFT

Compilation combines your selected batches into a final dataset and automatically partitions approximately 10% into a validation set.

**To compile:**

1. Open Compile & Export with your registry selections made.
2. Check the breakdown: included batches, total examples, training count, and validation count. Example: a 115-example selection produces a 103-example training set and a 12-example validation set.
3. Choose the target export template format.
4. Compile. The stage aligns the selected data to the target template so independently created batches are made consistent rather than assumed to match.

---

## 9. Template Reformatting & Output Preview

The same dataset can be reformatted for a different training pipeline without regenerating examples. This is useful when you move between model frameworks that expect different serialization.

**To reformat and preview:**

1. Choose the target export template (e.g., ChatML).
2. Apply it to the selected batches.
3. Open the Output Preview and inspect the serialized structure — for ChatML, you will see the exact `<|im_start|>` / `<|im_end|>` token layout that will be exported, including any think/reasoning blocks.
4. Export when the structure is correct.

**Notes:**

- The preview shows the machine-readable form exactly as it will be written to file. Check it before exporting — catching a template mismatch here is cheaper than catching it in training.

---

## 10. Final Export (JSON / JSONL)

The export area produces separate files for the training set and the validation subset, in either format:

- **JSON** — a single structured document. Best for human inspection, archival, debugging, and manual review.
- **JSONL** — one JSON object per line. The format most training pipelines consume directly.

**To export:**

1. Choose JSON or JSONL for each subset.
2. Download the files, or use copy-to-clipboard to move JSONL directly into another training environment when you do not need a file download.

The result is a complete package: a QA-screened training set, a separate validation set, both in the target template format.

---

## 11. Studio Expert / App Support

The Studio Expert is a project-aware chatbot for help inside the studio. Because it can see the current project context, it gives advice grounded in the work you have underway instead of making you re-explain the project.

**It can help with:**

- Defining personas and system prompts
- Creating strong training examples
- Generating and configuring batches
- Document conversion
- Interpreting quality-audit results
- Dataset assembly and template selection
- Export and troubleshooting

Use it when you are unsure what a control does, why the auditor flagged something, or how to structure a batch for a particular goal.

---

## 12. Recommended End-to-End Workflow

For a new dataset, this sequence keeps the process clean, auditable, and reversible:

| # | Step | Action |
|---|---|---|
| 1 | Create the project | Define the task, persona, system behavior, constraints, and instruction structure |
| 2 | Generate or convert | Create synthetic batches or convert source material into candidate examples |
| 3 | Review | Inspect examples; edit the fixable; reject the wrong-behavior ones |
| 4 | Optional TRiAD | Add FTK-balanced alignment examples if desired |
| 5 | Run QA | Audit every example for corruption, template/schema failure, and drift |
| 6 | Resolve flags | Edit, ignore, reject, or regenerate flagged examples |
| 7 | Select in Registry | Include only QA-checked batches |
| 8 | Compile | Combine selections and create the automatic validation split |
| 9 | Align template | Apply the target training template consistently |
| 10 | Preview | Inspect the formatted output |
| 11 | Export | Download or copy training and validation JSON/JSONL |

You can re-enter at any stage; nothing is destructive until you approve, compile, and export.

---

## Appendix — What the Studio Produces

By the end of the workflow, the studio has turned source material and/or generated batches into a controlled dataset with explicit provenance and quality status:

- Project definition (task, persona, system prompt, constraints, template)
- Generation batches and/or converted document batches
- Example-level edits and review decisions
- AI QA scores and individual quality flags
- The QA-checked Dataset Registry
- The selected final dataset
- The automatic validation subset
- The template-formatted output preview
- Training JSON and JSONL
- Validation JSON and JSONL

SFT Studio Pro is a dataset production pipeline, not just a generator.