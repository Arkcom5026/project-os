# UX-REVIEW-001 — Mission C UX Review

Mission: Mission C  
Role: FE-02 UX Owner  
Assignment: ASSIGNMENT-001  
Status: UX guidance report  
Implementation: NOT APPROVED — no code changes

---

## PASS / NEEDS_DECISION

```txt
PASS WITH ROLE-ARCH DECISIONS REQUIRED
```

FE-02 supports Mission C direction: Local Operational Product should be usable immediately by the branch, while Product Template Candidate review happens later as a catalog governance workflow.

This preserves Mission B receive flow and avoids blocking branch operations.

---

## 1. What should the branch operator see after creating a local product?

The branch operator should see immediate operational success, not an approval-style request state.

Recommended operator-facing meaning:

```txt
สร้างสินค้าของร้านเรียบร้อยแล้ว
พร้อมใช้งานและรับสินค้าได้ทันที
ระบบจะส่งข้อมูลให้ผู้ดูแล Catalog ตรวจสอบภายหลัง
```

Primary UX goals:

1. Confirm the Local Operational Product was created.
2. Confirm BranchPrice / required price data is ready or clearly show what is still missing.
3. Keep the operator on the QuickStock receive path.
4. Make clear that Catalog review does not block stock intake.
5. Avoid forcing the operator to understand Template governance.

The operator should not be redirected into a catalog administration workflow.

---

## 2. How should the system explain that catalog review will happen later?

Catalog review should be explained as a background improvement process, not as permission to use the product.

Recommended explanation:

```txt
สินค้านี้ใช้งานในร้านได้ทันที
ระบบส่งข้อมูลเป็น Candidate ให้ผู้ดูแล Catalog ตรวจสอบภายหลัง
```

Recommended microcopy:

```txt
พร้อมใช้งานในร้าน
ส่งตรวจ Catalog ภายหลัง
ไม่ต้องรออนุมัติเพื่อรับสินค้า
```

Avoid wording:

```txt
รออนุมัติ
ส่งคำขอ
Pending Approval
ยังใช้ไม่ได้จนกว่าจะอนุมัติ
```

Reason:

Those phrases make branch staff believe the created product cannot be used, which conflicts with Mission C doctrine: Operational-first, Catalog-evolution.

---

## 3. What wording is clear for branch staff?

Preferred Thai wording should be short and operational.

Recommended labels:

```txt
สร้างสินค้า Local ของร้าน
สร้างสินค้าของร้านเรียบร้อยแล้ว
พร้อมรับสินค้า
ส่งข้อมูลให้ผู้ดูแล Catalog ตรวจสอบ
ตรวจสอบภายหลัง
```

Recommended helper text:

```txt
ใช้เมื่อไม่พบสินค้าในระบบกลางหรือสินค้าในร้าน
ระบบจะสร้างสินค้าให้ร้านใช้งานทันที และส่งข้อมูลให้ผู้ดูแล Catalog ตรวจสอบภายหลัง
```

Recommended post-create success message:

```txt
สร้างสินค้า Local เรียบร้อยแล้ว พร้อมรับสินค้า
ระบบส่งข้อมูลให้ผู้ดูแล Catalog ตรวจสอบภายหลัง
```

Use carefully:

```txt
Template Candidate
Catalog Candidate
Operational Product
```

These terms are useful for internal/admin surfaces, but branch-facing UI should use simpler wording first.

---

## 4. What should Catalog Admin review later?

Catalog Admin should review the Candidate as a catalog proposal, not as branch runtime data that blocks store work.

Recommended admin review fields:

```txt
Candidate status
Source branch
Source Operational Product reference
Created by / created at
Product name
Brand
ProductType
Unit
Serial tracking / noSN
Mode / product configuration
Images if available
Duplicate or similar Template candidates
Potential existing Template match
Branch usage evidence if available
```

Recommended admin decisions:

```txt
Approve and promote to Template Catalog
Reject candidate
Request revision
Merge with existing Template
Mark duplicate
Keep as branch-local only
```

Admin should be able to distinguish:

```txt
Local Operational Product remains valid for branch runtime
Candidate review affects central catalog governance only
```

---

## 5. UX risks ROLE-ARCH should decide before implementation

### Risk 1 — Approval wording may block branch behavior mentally

If the UI says “waiting for approval,” branch staff may stop receiving stock even though Local Operational Product is already valid.

ROLE-ARCH should lock wording doctrine:

```txt
Catalog review must never sound like branch operation approval.
```

### Risk 2 — Candidate status visibility to branch

Question:

```txt
Should branch staff see Candidate status after creation?
```

FE-02 recommendation:

Show only a simple background status if needed, such as:

```txt
ส่งตรวจ Catalog แล้ว
```

Do not make status prominent in QuickStock unless it affects branch work.

### Risk 3 — Duplicate candidate expectations

If multiple branches create similar Local Products, users may expect automatic merging.

Mission C non-goals exclude automatic merge. UI should not imply automatic catalog cleanup.

### Risk 4 — Admin review workload

If every Local Product creates a Candidate, Catalog Admin may receive noisy or low-quality submissions.

ROLE-ARCH should decide whether Candidate creation is:

```txt
always automatic
optional with branch checkbox
automatic only when minimum metadata quality is met
```

FE-02 prefers automatic Candidate creation initially, but with clear admin filtering/status.

### Risk 5 — Branch-created names may be informal

Branch staff may type practical names that are good enough for local receiving but not suitable for central catalog.

Admin review must support revision before promotion.

### Risk 6 — Candidate promotion should not mutate branch runtime unexpectedly

If a Candidate becomes Template Catalog later, the existing branch Operational Product should not be silently overwritten in ways that affect price, barcode, stock, or local operations.

ROLE-ARCH should define whether promotion only links metadata or also suggests updates.

---

## 6. UX recommendation

FE-02 recommends this user-facing model:

```txt
Branch operator creates Local Product
-> Product is ready immediately
-> Stock intake can continue
-> System quietly creates Catalog Candidate
-> Catalog Admin reviews later
```

Recommended branch UI principle:

```txt
Operational success first. Catalog governance second.
```

Recommended branch success message:

```txt
สร้างสินค้า Local เรียบร้อยแล้ว พร้อมรับสินค้า
ระบบส่งข้อมูลให้ผู้ดูแล Catalog ตรวจสอบภายหลัง
```

Recommended admin UI principle:

```txt
Candidate review is catalog governance, not stock operation approval.
```

---

## 7. Mission B boundary confirmation

Mission C should not change Mission B receive commit flow.

Mission B remains:

```txt
Operational Product
-> Barcode / Queue
-> /api/quick-stock/existing
-> Branch runtime stock result
```

Mission C adds a parallel catalog-evolution side effect after Local Product creation:

```txt
Local Operational Product created
-> Product Template Candidate created separately
-> Admin reviews later
```

---

## 8. Recommended next owner

```txt
ROLE-ARCH
```

Recommended next action:

```txt
Lock Mission C UX doctrine and decide Candidate visibility, automatic creation policy, and promotion/linking behavior before BE/FE implementation proposals.
```

---

## Completion Response

Report path:

```txt
missions/mission-c/inbox/FE-02/UX-REVIEW-001.md
```

PASS/NEEDS_DECISION:

```txt
PASS WITH ROLE-ARCH DECISIONS REQUIRED
```

UX recommendation:

```txt
Local Product should be immediately usable by the branch. Catalog review should happen later as a background governance process and must not sound like operational approval.
```

Risks:

```txt
Approval wording confusion, candidate status visibility, duplicate candidates, admin workload, informal branch naming, and promotion side effects need ROLE-ARCH decisions.
```

Recommended next owner:

```txt
ROLE-ARCH
```
