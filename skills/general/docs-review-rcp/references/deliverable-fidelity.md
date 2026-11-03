# Deliverable fidelity and visual review

Read when the audit includes Word/PDF compilations or diagram sources and exports. Keep all temporary copies, extracted media, renderer profiles, PDFs, and page images outside the reviewed repository. Inspect the existing deliverable before any temporary re-render; the existing export is part of the evidence.

## Structural extraction

Establish the compilation manifest from documentation or generation configuration. If none exists, infer the relationships that can be demonstrated and record uncertain inclusion boundaries. Complementary runbooks and pending registers need not be reproduced in a client Word compilation unless the project requires them there.

For DOCX, extract paragraphs and tables in document order, including nested tables and relevant text boxes, hyperlinks/relationship targets, headings, notes, headers, and footers. Include footnotes/endnotes when present and relevant. Basic paragraph-only extraction misses material evidence. Use available extraction tools or inspect the DOCX package's XML and relationships; account for any content the chosen extractor omits.

Compare each intended source section, ADR, table, assumption, and diagram with the compilation. Check additions and omissions, stale values, changed conditions, and loss of proposal/approval qualifications. Trace discrepancies to the correct source version rather than assuming the newest file was used. Check internal bookmarks, external hyperlinks, unresolved Markdown links, and referenced identifiers.

For PDF, use available text/layout extraction to locate sections and tables, retaining the existing PDF as the visual source. Text extraction establishes content evidence, not page layout quality.

## Render and inspect

Discover the document/PDF rendering tools available in the environment. Prefer an existing renderer that can produce page images; otherwise use an available office converter for a temporary DOCX-to-PDF copy and a PDF rasterizer. Keep renderer output and any writable office profile in the temporary directory. These tool choices do not require another installed skill.

Record the renderer and any missing fonts or rendering differences that could affect conclusions. Confirm that the complete document was rendered, determine the actual page count, and inspect every page at a resolution that makes body text, tables, and diagrams readable. Overview/contact sheets can help locate problems but do not establish readability when their content is too small. Revisit suspicious areas at full size.

Look for:

- Clipped tables, split rows that lose meaning, missing cells, and columns too narrow to read.
- Overlapping, truncated, or missing text and diagrams.
- Unreadable labels, stretched images, broken characters, and missing fonts.
- Inappropriate page breaks, stranded headings/captions, and header/footer collisions.
- Contents/index accuracy, heading hierarchy, numbering, captions, navigation, and page references.

Record verified page numbers with the relevant heading/table/figure. Distinguish rendered page index from a printed page label when they differ. Do not infer pagination from DOCX XML or assume a field is up to date. If rendering fails or pages cannot be inspected, continue supported content checks and explicitly list the missing visual coverage.

## Diagram consistency

For each in-scope BPMN or other model, inspect the source, existing image export, and embedded deliverable image. Check the model/export index and trace identifiers, participants, activities, branches, message flows, and legend/captions. Confirm that the embedded image corresponds to the intended model/version and that labels remain readable at their delivered size.

For BPMN, compare modeled responsibilities, gateways, events, failure paths, and recovery/manual operations with prose and runbooks. A missing branch is a defect only when required by a documented flow or claimed guarantee; model detail depends on its stated purpose.

When regeneration helps validate an export, render a temporary source copy outside the repository. Identify which original export was inspected and which image was newly rendered. Without a usable model renderer, inspect the source structure and available exports, and leave unverifiable graphical correspondence pending.

## Coverage evidence

Track each deliverable's extraction coverage, render result, total pages, inspected pages, and uninspected elements. Track each diagram's source, export, embedded counterpart, and the comparisons actually completed. Conclude semantic fidelity and visual quality separately; one does not prove the other.
