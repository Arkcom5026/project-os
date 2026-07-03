# CHARTER-001 — Mission C: Local Product Evolution

Mission: Mission C
Status: OPEN / ARCHITECTURE PHASE
Execution mode: Local implementation later. Git is used for role communication and shared understanding only.

## Mission Goal

Allow a branch user to create a Local Operational Product for immediate store use, while also creating a Product Template Candidate for later catalog review and promotion by an administrator.

## Core Flow

```txt
Search product
-> No Operational Product and no Template Product found
-> User creates Local Operational Product
-> System creates branch Product immediately
-> System creates BranchPrice
-> System creates Product Template Candidate separately
-> Branch can receive stock immediately
-> Catalog administrator reviews Candidate later
-> Approve / Reject / Revise / Merge Existing
-> If approved, promote into central Template Catalog
```

## Responsibility Separation

### Local Operational Product

Purpose:

```txt
Immediate branch runtime use
```

Owns:

```txt
Product used by the branch
BranchPrice
Barcode settings
Stock intake
POS runtime visibility
```

Owner:

```txt
Branch / store runtime
```

### Product Template Candidate

Purpose:

```txt
Catalog proposal created from real branch operation
```

Owns:

```txt
Candidate metadata
Proposed name
Brand
ProductType
Unit
Images if available
Source branch product reference
Review state
```

Owner:

```txt
Creator branch + Catalog Administrator
```

### Template Catalog

Purpose:

```txt
Central approved catalog
```

Owns:

```txt
Approved reusable product template
Template search / clone source
Catalog governance
```

Owner:

```txt
Catalog Administrator
```

## Candidate Lifecycle

```txt
Draft
-> Submitted
-> Under Review
-> Reject / Revise / Merge Existing / Approve
-> Promote
-> Template Catalog
```

## Doctrine

### Operational-first, Catalog-evolution

Branches can create Operational Product data to continue real work immediately. Central Catalog quality is maintained by routing new catalog suggestions through Product Template Candidate review instead of allowing branches to create Template Catalog records directly.

## Mission C Non-goals

```txt
Do not change Mission B receive commit flow.
Do not let branch users create Template Catalog directly.
Do not implement AI matching.
Do not implement automatic merge.
Do not implement broad cross-branch catalog governance.
Do not implement advanced scoring or analytics.
```

## Architecture Phase Deliverables

```txt
FE-02: UX flow and candidate review experience
BE-01: Candidate model/API/state machine proposal
FE-01: Frontend runtime impact and store/component proposal
ROLE-ARCH: consolidate reports into Mission C Architecture Lock
```

## Local Development Rule

Implementation will be done locally and delivered as a `.zip` for manual placement. Git is used only as the shared communication and role-reading space until the user explicitly approves code integration.