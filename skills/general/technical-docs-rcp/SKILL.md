---
name: technical-docs-rcp
description: Write, revise, or validate technical documentation against the reader's goal and available sources.
disable-model-invocation: true
---

# Technical Docs

Turn the supplied input into documentation that lets a specific reader complete a task or understand a system.

For validation, follow steps 1–2, then **Review** below. For writing or revision, follow the full workflow.

## Workflow

1. **Frame the task.** Identify the reader, their goal, scope, and deliverable from the input and project context. Follow the requested language, format, and existing docs conventions; otherwise use Markdown. Ask only about gaps that materially change the result. State reasonable assumptions and continue where possible.

2. **Ground the content.** Read relevant docs and inspect the code, configuration, tests, or supplied sources needed to support the document. Verify version-sensitive external details against current official documentation using available documentation tools. Distinguish implemented behavior from proposed requirements. Label unknowns and assumptions; keep unsupported claims out of factual instructions.

3. **Choose the structure.** Use the reader's goal to choose a tutorial, task guide, reference, or explanation. Put the common path first and add only useful sections. Keep prerequisites before the steps that need them; place alternatives and deeper detail afterward. Follow existing terminology and link related docs instead of repeating them.

4. **Write the document.** Lead with its purpose and applicability. Use direct language, descriptive headings, and concise paragraphs. For procedures, include ordered steps, copyable examples, expected results, and recovery from likely failures. For reference material, include exact inputs, defaults, constraints, and outputs where relevant. For design proposals, identify decisions, tradeoffs, and open questions. Include only details supported by the input or verification.

5. **Verify and deliver.** Walk through the document as its intended reader. Check examples, paths, links, and consistency against the sources; run examples when practical and authorized. Confirm the requested scope is covered and trim repetition. Edit the requested files, or return the document directly when no destination is established. Briefly report what was produced, what was verified, and any unresolved facts or unrun examples.

## Review

Check accuracy, scope coverage, structure, terminology, prerequisites, examples, success criteria, recovery guidance, and links as relevant to the document's purpose. Verify suspected defects against sources; distinguish confirmed errors, unverifiable claims, and optional improvements.

Report findings in order of reader impact, each with a document location, supporting evidence, and a concrete correction. Conclude whether the document serves its intended reader, stating verification gaps and unrun examples. Insufficient evidence limits the verdict. Return the review without editing the document unless changes were requested.
