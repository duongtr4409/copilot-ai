---
name: system-design-reasoning
description: "Transform BA requirements into technical architecture with explicit reasoning and trade-offs. Use when decomposing into microservices/modules, designing frontend-backend-database flows, creating Mermaid diagrams with /draw, and producing architecture docs with #architecture-docs."
argument-hint: "BA requirements, NFRs, constraints, expected scale, and integrations | Yeu cau BA, NFR, rang buoc, quy mo, tich hop"
user-invocable: true
---

# System Design Reasoning (Suy luan Thiet ke He thong)

## Outcome | Dau ra
- EN: Convert abstract business requirements into executable architecture.
- VN: Chuyen hoa yeu cau nghiep vu tru tuong thanh kien truc ky thuat co the trien khai.
- EN: Make design reasoning explicit (why, trade-off, risk).
- VN: The hien ro ly do thiet ke (tai sao, danh doi, rui ro).
- EN: Produce architecture artifacts for implementation alignment.
- VN: Tao bo tai lieu kien truc de doi ngu trien khai dong bo.

## When To Use | Khi nao dung
- EN: BA requirements are business-level but not technical enough for implementation.
- VN: Yeu cau BA moi o muc nghiep vu, chua du chi tiet ky thuat de code.
- EN: Team needs module or microservice boundaries.
- VN: Doi ngu can xac dinh ranh gioi module hoac microservice.
- EN: Team needs end-to-end flow from Frontend to Backend to Database.
- VN: Doi ngu can luong du lieu dau-cuoi tu Frontend den Backend va Database.
- EN: Team needs diagrams and formal architecture docs.
- VN: Doi ngu can so do va tai lieu kien truc chuan hoa.

## Required Inputs | Dau vao bat buoc
- EN: BA requirements, user stories, business rules.
- VN: Yeu cau BA, user story, business rule.
- EN: NFRs (latency, throughput, availability, security, observability).
- VN: NFR (do tre, thong luong, san sang, bao mat, quan sat).
- EN: Constraints (stack, budget, timeline, team capability).
- VN: Rang buoc (cong nghe, chi phi, thoi gian, nang luc doi ngu).
- EN: Data constraints (consistency, privacy, retention, audit).
- VN: Rang buoc du lieu (nhat quan, rieng tu, luu tru, kiem toan).

## Procedure | Quy trinh
1. Requirement decomposition | Phan ra yeu cau
- EN: Group BA requirements into capabilities and bounded contexts.
- VN: Nhom yeu cau BA theo capability va bounded context.
- EN: Tag each capability with criticality, scale, and owner.
- VN: Dan nhan muc do quan trong, quy mo, va owner cho tung capability.

2. Choose partition strategy | Chon chien luoc phan tach
- EN: Prefer modular monolith if scope is early and ops maturity is limited.
- VN: Uu tien modular monolith neu pham vi con som va van hanh chua truong thanh.
- EN: Prefer microservices if domains need independent scaling/deployment.
- VN: Uu tien microservices neu domain can scale/trien khai doc lap.
- EN: Use hybrid when uncertainty is high.
- VN: Dung hybrid khi do bat dinh cao.

3. Define modules/services | Dinh nghia module/dich vu
- EN: Define purpose, responsibilities, non-responsibilities.
- VN: Dinh nghia muc dich, trach nhiem, va pham vi khong thuoc trach nhiem.
- EN: Define interfaces (API/events/errors), ownership, and dependency direction.
- VN: Dinh nghia giao dien (API/event/error), ownership, va huong phu thuoc.

4. Design data ownership and consistency | Thiet ke so huu du lieu va nhat quan
- EN: Map entities to service/module ownership.
- VN: Gan entity vao ownership cua module/dich vu.
- EN: Decide strong vs eventual consistency by business criticality.
- VN: Chon nhat quan manh hay eventual theo muc do quan trong nghiep vu.
- EN: Document compensating actions where eventual consistency is used.
- VN: Mo ta co che compensation khi dung eventual consistency.

5. Design Frontend-Backend-Database flow | Thiet ke luong Frontend-Backend-Database
- EN: Map critical user journeys to sync APIs and async events.
- VN: Anh xa user journey quan trong vao API dong bo va event bat dong bo.
- EN: Include timeout, retry, fallback, idempotency, and caching strategy.
- VN: Bao gom timeout, retry, fallback, idempotency, va cache strategy.

6. Visualize with /draw (Mermaid) | Truc quan hoa bang /draw (Mermaid)
- EN: Produce at least 3 diagrams.
- VN: Tao it nhat 3 so do.
- EN: Required diagrams: Context, Container/Component, and one critical Sequence flow.
- VN: Bat buoc co: Context, Container/Component, va Sequence cho 1 luong quan trong.

7. Document with #architecture-docs | Tai lieu hoa bang #architecture-docs
- EN: Write goals, constraints, topology, data flow, trade-offs, and risks.
- VN: Viet goals, constraints, topology, data flow, trade-offs, va risks.
- EN: Record ADR-style decisions (Decision, Options, Rationale, Consequences).
- VN: Ghi lai quyet dinh theo ADR (Decision, Options, Rationale, Consequences).
- EN: Trace decisions back to BA requirement IDs.
- VN: Truy vet moi quyet dinh ve requirement ID tu BA.

8. Quality gate before handoff | Cong gate chat luong truoc ban giao
- EN: Verify traceability from BA requirements to architecture components.
- VN: Kiem tra truy vet tu BA requirement den thanh phan kien truc.
- EN: Verify ownership/interface clarity, NFR coverage, and failure handling.
- VN: Kiem tra ro rang ownership/interface, bao phu NFR, va xu ly su co.
- EN: Output pass/fail with action items.
- VN: Dua ra ket qua pass/fail kem action item.

## Decision Branches | Nhanh quyet dinh
- Boundary decision:
- EN: If independent team ownership + independent scaling are required -> microservices.
- VN: Neu can ownership theo team va scale doc lap -> microservices.
- EN: If simplicity and delivery speed are priority -> modular monolith first.
- VN: Neu uu tien don gian va toc do giao hang -> modular monolith truoc.

- Consistency decision:
- EN: If financial/legal correctness is critical -> strong consistency.
- VN: Neu tinh dung tai chinh/phap ly la toi quan trong -> nhat quan manh.
- EN: If throughput and autonomy dominate -> eventual consistency + compensation.
- VN: Neu thong luong va tu chu la uu tien -> eventual consistency + compensation.

- Integration decision:
- EN: If immediate response UX is mandatory -> synchronous APIs.
- VN: Neu UX bat buoc phan hoi ngay -> API dong bo.
- EN: If resilience/decoupling is priority -> async events/messages.
- VN: Neu uu tien resilience/decoupling -> event/message bat dong bo.

## Completion Criteria | Tieu chi hoan thanh
- EN: Each major BA requirement maps to architecture elements.
- VN: Moi BA requirement quan trong duoc anh xa vao thanh phan kien truc.
- EN: Each architecture decision has rationale and trade-off.
- VN: Moi quyet dinh kien truc co ly do va danh doi ro rang.
- EN: Critical Frontend-Backend-Database flows are documented.
- VN: Luong Frontend-Backend-Database quan trong da duoc mo ta day du.
- EN: Mermaid diagrams and docs are consistent.
- VN: So do Mermaid va tai lieu kien truc nhat quan voi nhau.
- EN: Risks, assumptions, and open questions are explicit.
- VN: Rui ro, gia dinh, va cau hoi mo duoc neu ro.

## Output Contract | Mau dau ra bat buoc
When invoked, return these sections in order:
1. Requirement Decomposition | Phan ra yeu cau
2. Candidate Boundaries (Microservices/Modules) | Ranh gioi ung vien
3. Frontend-Backend-Database Data Flow | Luong du lieu FE-BE-DB
4. Architecture Decisions and Trade-offs | Quyet dinh va danh doi
5. Mermaid Plan for /draw | Ke hoach so do Mermaid
6. Documentation Plan for #architecture-docs | Ke hoach tai lieu kien truc
7. Quality Gate Result | Ket qua cong gate
8. Risks and Open Questions | Rui ro va cau hoi mo
