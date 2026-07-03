# CANDIDATE-CONTRACT-DISCOVERY-001 — Mission C Backend Candidate Contract Discovery

Mission: Mission C
Role: BE-01 Backend Runtime Owner
Assignment: missions/mission-c/assignments/BE-01/ASSIGNMENT-001.md
Status: SUBMITTED
Implementation: NOT PERFORMED
Code Changes: NONE

---

## 1. Scope

This is a BE-01 discovery/review report only.

Objective: determine whether Product Template Candidate already has backend model, endpoint, or runtime support, then propose the backend contract direction for Mission C architecture phase.

No code was changed. No Prisma migration was created. Mission B receive flow was not altered. Promotion was not implemented. No assignments were created for other roles.

---

## 2. Files inspected

project-os:

- missions/mission-c/assignments/BE-01/ASSIGNMENT-001.md
- missions/mission-c/CHARTER-001.md

alpha-tech-server:

- prisma/schema.prisma
- routes/productRoutes.js
- src/modules/product/routes/templateProductSearchRoutes.js
- src/modules/product/controllers/templateProductSearchController.js
- src/modules/product/services/templateProductSearchService.js
- src/modules/product/repositories/productTemplateRepository.js

Searches performed:

- Candidate ProductTemplateCandidate product template candidate
- candidate
- ProductTemplate TemplateProduct productTemplate templateProduct
- admin product template routes templateBranch T01 create product template approve promote
- branchCode T01 product create template

---

## 3. Mission C baseline from Charter

Mission C goal: allow a branch user to create a Local Operational Product for immediate store use, while also creating a Product Template Candidate for later catalog review and promotion by an administrator.

Core flow:

- Search product
- No Operational Product and no Template Product found
- User creates Local Operational Product
- System creates branch Product immediately
- System creates BranchPrice
- System creates Product Template Candidate separately
- Branch can receive stock immediately
- Catalog administrator reviews Candidate later
- Approve / Reject / Revise / Merge Existing
- If approved, promote into central Template Catalog

Responsibility separation:

- Local Operational Product = immediate branch runtime use
- Product Template Candidate = catalog proposal created from real branch operation
- Template Catalog = central approved catalog

Mission C non-goals include: do not change Mission B receive commit flow, do not let branch users create Template Catalog directly, do not implement AI matching, do not implement automatic merge, do not implement broad cross-branch catalog governance, and do not implement advanced scoring or analytics.

---

## 4. Existing Candidate model or table

Result: No existing Product Template Candidate model/table was found.

Evidence:

- Repository search for Candidate / ProductTemplateCandidate returned no backend runtime model, route, controller, or service.
- prisma/schema.prisma has Product, ProductType, Brand, BranchPrice, StockBalance, StockMovement, ProductImage, Unit and related runtime models, but no Candidate model.
- Existing Product supports template clone tracking through templateProductId, but that is clone lineage, not a review candidate workflow.

Current schema facts:

- Product is used for both Template Catalog and Operational Product depending on productType.branchId.
- Product.templateProductId links an Operational Product clone back to a Template Product.
- ProductType.branchId identifies branch-scoped ProductType. Template Catalog currently uses Template Branch/ProductType pattern, currently T01.
- BranchPrice has productId + branchId unique pricing for branch runtime.

Conclusion: Mission C requires a new Candidate persistence model if candidates must be reviewable, stateful, auditable, and promotable later.

---

## 5. Existing Candidate routes/controllers/services

Result: No existing Product Template Candidate routes/controllers/services were found.

Existing related runtime support:

- GET /api/products/template/search
- src/modules/product/routes/templateProductSearchRoutes.js
- templateProductSearchController.searchTemplateProducts
- TemplateProductSearchService.searchTemplateProducts
- ProductTemplateRepository.searchTemplateProducts

This supports Template Catalog search from Template Branch T01. It is not a Candidate review flow.

Existing Local Operational Product runtime support:

- POST /api/products/pos/create-local
- routes/productRoutes.js:createLocalOperationalProduct

This creates the branch Operational Product and BranchPrice, but it does not create a Candidate.

No admin review/promotion endpoint was found for Product Template Candidate.

---

## 6. Required Prisma model proposal if missing

BE-01 recommends adding a dedicated ProductTemplateCandidate model instead of overloading Product.

Recommended enum name:

- ProductTemplateCandidateStatus

Recommended statuses:

- DRAFT
- SUBMITTED
- UNDER_REVIEW
- REJECTED
- NEEDS_REVISION
- MERGE_EXISTING
- APPROVED
- PROMOTED

Recommended ProductTemplateCandidate model responsibilities:

- Candidate identity and timestamps
- Review status
- sourceBranchId
- sourceOperationalProductId
- createdByEmployeeId
- reviewedByEmployeeId
- proposedName
- proposed mode/noSN/trackSerialNumber
- categoryId
- productTypeId
- brandId
- unitId
- warrantyDays
- productConfig
- proposedImages snapshot or relation strategy
- reviewNote
- rejectionReason
- revisionRequest
- mergedIntoTemplateProductId
- promotedTemplateProductId
- promotedAt

Recommended indexes:

- sourceBranchId
- sourceOperationalProductId
- status
- productTypeId
- brandId
- promotedTemplateProductId

Minimum viable fields for Phase 1:

- id
- status
- sourceBranchId
- sourceOperationalProductId
- createdByEmployeeId
- proposedName
- proposedMode
- proposedNoSN
- proposedTrackSerialNumber
- categoryId/productTypeId/brandId/unitId
- reviewNote
- createdAt/updatedAt

Notes:

- Candidate should not be represented by Product.templateProductId because that field means clone lineage from Template Product to Operational Product.
- Candidate should reference sourceOperationalProductId to preserve branch runtime origin.
- Candidate should not create Template Catalog Product until promotion is explicitly approved.

---

## 7. Required API proposal

Recommended route namespace:

- /api/products/template-candidates

Reason: Candidate belongs to Product Template governance, but is separate from Template Catalog search and branch Product runtime.

Recommended branch/runtime creation endpoint:

- POST /api/products/template-candidates

Purpose: create a Candidate proposal tied to a branch-created Operational Product.

Recommended create request responsibilities:

- sourceOperationalProductId
- proposedName
- productTypeId
- brandId
- unitId
- mode/noSN/trackSerialNumber
- images or image reference snapshot if approved

Backend should derive:

- sourceBranchId from req.user.branchId
- createdByEmployeeId from req.user.employeeId or req.user.id depending current auth context

Recommended success response responsibilities:

- success
- created
- candidate.id
- candidate.status
- candidate.sourceBranchId
- candidate.sourceOperationalProductId
- candidate.proposedName

Optional combined Mission C create endpoint:

- POST /api/products/pos/create-local-with-candidate

This would create Local Operational Product + BranchPrice + ProductTemplateCandidate in one transaction.

BE-01 recommendation: prefer separate Candidate creation first unless ROLE-ARCH requires atomic local+candidate creation.

Reason: Mission B create-local already works and should not be destabilized. A separate endpoint preserves Mission B and keeps Candidate evolution isolated.

Recommended admin/review endpoints:

- GET /api/products/template-candidates
- GET /api/products/template-candidates/:id
- PATCH /api/products/template-candidates/:id/review
- POST /api/products/template-candidates/:id/reject
- POST /api/products/template-candidates/:id/request-revision
- POST /api/products/template-candidates/:id/merge-existing
- POST /api/products/template-candidates/:id/approve
- POST /api/products/template-candidates/:id/promote

Promotion should be admin/catalog-governed only.

---

## 8. Candidate state machine proposal

Recommended lifecycle:

DRAFT -> SUBMITTED -> UNDER_REVIEW -> REJECTED / NEEDS_REVISION / MERGE_EXISTING / APPROVED -> PROMOTED -> Template Catalog

Recommended allowed transitions:

- DRAFT -> SUBMITTED
- SUBMITTED -> UNDER_REVIEW
- SUBMITTED -> REJECTED
- SUBMITTED -> NEEDS_REVISION
- UNDER_REVIEW -> REJECTED
- UNDER_REVIEW -> NEEDS_REVISION
- UNDER_REVIEW -> MERGE_EXISTING
- UNDER_REVIEW -> APPROVED
- NEEDS_REVISION -> SUBMITTED
- APPROVED -> PROMOTED

Terminal states:

- REJECTED
- MERGE_EXISTING
- PROMOTED

Important rule: APPROVED is not the same as PROMOTED.

Reason: approval is a governance decision; promotion is a catalog write operation that creates or links the central Template Catalog product.

---

## 9. Relationship to Local Operational Product

Current Local Operational Product support exists:

- POST /api/products/pos/create-local

Current behavior:

- Creates branch Product immediately
- Creates BranchPrice immediately
- Uses authenticated branch context
- Rejects templateProductId
- Does not create stock rows
- Does not create Candidate

Mission C relationship proposal:

- Local Operational Product remains the source of truth for branch runtime.
- Product Template Candidate references the Local Operational Product as source evidence.
- Template Catalog remains separate until approved promotion.

Recommended source relation:

- ProductTemplateCandidate.sourceOperationalProductId -> Product.id

Important separation rule:

- Creating a Candidate must not block branch receiving stock.
- Candidate review must not mutate the branch Operational Product unless explicitly assigned later.

Candidate should preserve copied metadata from the Operational Product at creation time so review remains stable even if branch product changes later.

---

## 10. Promotion path to Template Catalog

Current Template Catalog implementation pattern:

- Template Catalog is represented by Product records under Template Branch/ProductType.
- Current Template Branch code is T01.

Existing search path:

- GET /api/products/template/search
- Finds Template Branch by branchCode T01
- Searches Product where product.productType.branchId = templateBranch.id

Promotion proposal:

1. Admin approves Candidate.
2. Promotion service finds Template Branch T01.
3. Promotion service resolves or creates the matching Template ProductType in T01 based on Candidate product type/category governance.
4. Promotion service creates Product under Template Branch ProductType.
5. Product.templateProductId remains null because the promoted record is itself a Template Product.
6. Candidate.promotedTemplateProductId is set to the created Template Product id.
7. Candidate.status becomes PROMOTED.
8. Branch Operational Product may optionally later set templateProductId to the promoted Template Product id, but this should be a separate decision because it changes lineage semantics.

Merge Existing proposal:

If candidate duplicates an existing Template Product, admin may choose MERGE_EXISTING and set mergedIntoTemplateProductId to the existing Template Product id. No new Template Product is created.

Important risk:

No discovered canonical admin template creation/promote service exists today. Promotion should be implemented only after ROLE-ARCH locks the Candidate model and admin governance boundary.

---

## 11. Known backend risks

1. No existing Candidate persistence model/table.
2. No existing Candidate API or review controller/service.
3. No existing admin promotion runtime was discovered.
4. Current Template Catalog is encoded as Product under Template Branch T01; this works but must be handled carefully because Product also powers branch Operational Product runtime.
5. ProductType/Brand/Unit governance during promotion needs ROLE-ARCH decision.
6. Candidate image handling needs a decision: snapshot JSON, relation table, or reuse ProductImage references.
7. Separate Candidate creation may cause partial success if Local Product create succeeds but Candidate creation fails.
8. Atomic create-local-with-candidate reduces partial success but risks touching stable Mission B create-local flow.
9. Candidate status transitions require role/permission boundaries; this BE-01 discovery does not define final RBAC.
10. Duplicate detection/merge scoring is explicitly not part of Mission C non-goals, so backend should not implement AI matching or automatic merge.

---

## 12. Backend gaps

- ProductTemplateCandidate Prisma model missing.
- Candidate status enum missing.
- Candidate routes/controllers/services missing.
- Candidate create endpoint missing.
- Candidate list/detail/review endpoints missing.
- Candidate promotion service missing.
- Admin/catalog governance boundary not yet implemented in backend.
- No migration exists for Candidate persistence.

---

## 13. PASS / NEEDS_DECISION

NEEDS_DECISION

Reason:

Discovery completed successfully, but Mission C cannot proceed to backend implementation without architecture decisions on Candidate model shape, API namespace, creation coupling strategy, promotion behavior, and admin governance boundary.

BE-01 recommended decisions for ROLE-ARCH:

1. Approve dedicated ProductTemplateCandidate model.
2. Choose separate Candidate creation endpoint vs atomic create-local-with-candidate.
3. Lock Candidate status enum and allowed transitions.
4. Decide image storage strategy for Candidate.
5. Decide promotion governance: how ProductType/Brand/Unit are resolved in Template Branch T01.
6. Decide whether branch Operational Product should be linked to promoted Template Product after promotion.

---

## 14. Recommended next owner

ROLE-ARCH

Recommended next action:

Consolidate BE-01 Candidate contract discovery with FE/UX reports and issue a Mission C Architecture Lock or implementation assignment.

After ROLE-ARCH locks architecture, next implementation owner may be BE-01 for a minimal backend Candidate model/API implementation assignment.

---

## 15. Completion response

Report path:
missions/mission-c/inbox/BE-01/CANDIDATE-CONTRACT-DISCOVERY-001.md

PASS/NEEDS_DECISION:
NEEDS_DECISION

Files inspected:
project-os/missions/mission-c/assignments/BE-01/ASSIGNMENT-001.md
project-os/missions/mission-c/CHARTER-001.md
alpha-tech-server/prisma/schema.prisma
alpha-tech-server/routes/productRoutes.js
alpha-tech-server/src/modules/product/routes/templateProductSearchRoutes.js
alpha-tech-server/src/modules/product/controllers/templateProductSearchController.js
alpha-tech-server/src/modules/product/services/templateProductSearchService.js
alpha-tech-server/src/modules/product/repositories/productTemplateRepository.js

Backend gaps:
ProductTemplateCandidate model/table missing; Candidate routes/controllers/services missing; Candidate create/review/promote APIs missing; promotion governance not yet locked.

Recommended next owner:
ROLE-ARCH
