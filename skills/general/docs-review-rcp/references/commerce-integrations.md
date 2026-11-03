# Commerce architecture and integrations

Read for commerce documentation audits. Apply only the systems and requirements present in the source map; the names below are examples from the originating review, not an assumed stack.

## Requirements and authority

Check the original RFP across every in-scope category and priority. Where it uses B2C, B2B, ER, BO, and IM, retain those categories and their original meaning. Confirm each requirement independently of the coverage matrix; a lower priority alone does not exclude it or move it into a later phase.

Reconcile client architecture/ADRs, process models, go-live runbooks, and pending records with internal coverage, QA preparation, integration build specs, and scope-expansion notes. Follow source references back to the original requirement or approved change. An internal build spec can itself be wrong.

## Domain checks

| Area | Evidence to establish |
| --- | --- |
| Systems and ownership | Responsibility and authoritative data for the commerce platform (e.g. BigCommerce), integration layer (MuleSoft), CRM (Salesforce), ERP (SAP), PSP (Adyen), hospitality system (Opera), and other actual services. Trace handoffs and the party responsible when they fail. |
| Catalog and delivery | Products versus variants, physical versus digital goods, Postal/Email delivery options, mixed carts, checkout/address requirements, stock, and fulfillment. Confirm that the actual model supports the promised behavior. |
| Channels and commercial rules | Channels, customer groups, B2B Edition, price lists, promotions, currency, and plan/license availability. Separate platform capability from entitlement and project configuration. |
| Identity | Customer/staff identities, SSO, permissions, tenant/company boundaries, and seller identification throughout a transaction. |
| Payment to fulfillment | Payment events and states, order states, invoicing, voucher creation/issuance, release to the customer, and prerequisites between these transitions. Distinguish planned reconciliation from verified guarantees. |
| Reversals | Cancellation, credit notes, refunds, stock restoration, and already-used vouchers. Trace partial failure, duplicate events, and who can authorize or perform a manual correction. |
| Customer communications | Emails, languages, currencies, consent, and analytics: trigger, responsible sender/service, source of locale and consent, and claimed behavior. |
| Integration operations | Authentication/authorization, secrets handling, idempotency keys and scope, ordering, retries, reconciliation, observability, recovery, and manual runbooks. Trace guarantees to a contract or demonstrated mechanism. |

For each flow, trace the business event through authoritative state transitions and external calls, including failure and recovery paths. A diagram's happy path alone cannot establish a recovery guarantee.

## MuleSoft contracts

When the project defines RAML as its contract, treat the relevant RAML and included datatypes/traits as authoritative for API shape. Read the integration README to resolve files, includes, and version; cite the operation's method/path, datatype, and contract version.

Compare prose and diagrams against the actual request/response fields and operation semantics. Looking up documents or voucher codes does not establish an operation to create invoices, issue vouchers, or release them. If an issuance contract is absent, determine whether it is an acknowledged pending delivery or contradicts a claim that the integration is already specified.

RAML establishes the declared interface; deployment, entitlement, and successful operation need their own evidence. An incomplete or inaccessible contract leaves a validation pending, with the exact missing operation or datatype named.
