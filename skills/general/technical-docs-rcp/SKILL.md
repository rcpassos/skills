---
name: technical-docs-rcp
description: Write, revise, or validate clear, concise technical documentation against the reader's goal, sources, and language requirements.
disable-model-invocation: true
---

# Technical Docs

Turn the supplied input into documentation that lets a specific reader complete a task or understand a system.

For validation, follow steps 1–2, then **Review** below. For writing or revision, follow the full workflow.

## Workflow

1. **Frame the task.** Identify the reader, their goal, scope, and deliverable from the input and project context. Follow the requested language, locale, format, and explicit project style requirements; otherwise use Markdown. Apply the **Language standard** below to writing, revision, and validation. Ask only about gaps that materially change the result. Keep working assumptions in the handoff unless the reader needs them in the document.

2. **Ground the content.** Read relevant docs and inspect the code, configuration, tests, or supplied sources needed to support the document. Verify version-sensitive external details against current official documentation using available documentation tools. Distinguish implemented behavior from proposed requirements. Label unknowns and assumptions; keep unsupported claims out of factual instructions.

3. **Choose the structure.** Use the reader's goal to choose a tutorial, task guide, reference, or explanation. Put the common path first and add only useful sections. Keep prerequisites before the steps that need them; place alternatives and deeper detail afterward. Follow existing terminology and link related docs instead of repeating them.

4. **Write the document.** Lead with the task or fact the reader needs. Use descriptive headings and include only sections that serve the document's purpose. For procedures, include ordered steps, copyable examples, expected results, and recovery from likely failures. For reference material, include exact inputs, defaults, constraints, and outputs where relevant. For design proposals, identify decisions, tradeoffs, and open questions.

5. **Verify and deliver.** Walk through the document as its intended reader. Check examples, paths, links, and consistency against the sources; run examples when practical and authorized. Confirm the requested scope is covered and check every language rule below. Rewrite violations before delivery, preserving technical meaning and necessary detail. Edit the requested files, or return the document directly when no destination is established. Briefly report what was produced, what was verified, and any unresolved facts or unrun examples.

## Language standard

Apply these rules to authored prose. Preserve exact identifiers, commands, UI labels, and source quotations. Explain unfamiliar terms when the reader needs them. Translate the rules' intent into the requested language and locale.

- **Simple wording.** Use familiar, concrete words. Keep necessary technical terms and use one term consistently for each concept. Remove marketing language, hype, and buzzwords.
- **Short, direct sentences.** Express one main idea per sentence. Name the actor and action. Prefer verbs such as "configure" over phrases such as "perform the configuration of." Keep paragraphs focused on one topic.
- **Practical information.** Tell the reader what to do, when to do it, and how to recognize the result. Replace metaphors, cliches, and generalizations with specific behavior, conditions, or examples supported by the sources.
- **Supported claims.** State only what the evidence supports. Give the scope or conditions of a claim where they matter. Keep required actions, recommendations, permissions, and capabilities distinct. Preserve uncertainty and qualifications when shortening text.
- **Necessary words only.** Remove adjectives and adverbs that add no technical meaning. Delete repeated information; link to its authoritative location when needed. Retain prerequisites, constraints, warnings, and failure behavior needed to use the document correctly.
- **Reader content only.** Omit AI disclaimers and meta-commentary about drafting, research, or following instructions. Put verification limits in the handoff; retain limitations of the documented system in the document.
- **Direct openings and endings.** State the information directly. Omit conclusion or summary setup phrases such as "In conclusion," "In summary," and "To sum up." End when the reader has the information or next action they need.
- **Plain punctuation and constructions.** Use periods, commas, parentheses, or colons instead of em dashes. Replace paired "not only ... but also" constructions with a direct statement or separate sentences.

Examples (assuming the stated behavior is verified):

- "You can easily perform the configuration of the timeout parameter to 30 seconds" becomes "Set `timeout` to `30s`."
- "Our powerful integration seamlessly handles all failures" becomes "The integration retries failed requests three times." Use the narrower statement only if that retry policy is supported.
- "In summary, the service not only validates requests but also stores them" becomes "The service validates and stores requests."

## Review

Read the full in-scope document. Check accuracy, scope coverage, structure, prerequisites, examples, success criteria, recovery guidance, and links as relevant to its purpose. Apply every rule in **Language standard** to the prose, including repetition across sections. Verify suspected factual defects against sources; distinguish confirmed errors, unverifiable claims, language violations, and optional improvements. A shorter alternative alone is not evidence of a violation.

Report findings in order of reader impact, each with a document location, supporting evidence, and a concrete correction. For language violations, name the rule, quote the affected wording, and give a concise rewrite that preserves meaning. Group repeated instances of the same problem. State whether the document serves its reader and meets the language standard, identifying verification gaps and unrun examples. Insufficient evidence limits the verdict. Return the review without editing the document unless changes were requested.
