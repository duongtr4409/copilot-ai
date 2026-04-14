---
name: schema-data-migration
description: "Manage database lifecycle by designing SQL schemas or NoSQL document models and generating Liquibase changelog-first migrations with versioned scripts (for example V1__initial_schema.sql) to keep Dev, Staging, and Production synchronized. Use for schema design, migration planning, rollout safety, and rollback readiness."
argument-hint: "Domain requirements, DB type (SQL or NoSQL), current state, target state, Liquibase constraints | Yeu cau nghiep vu, loai DB (SQL hoac NoSQL), hien trang, muc tieu, rang buoc Liquibase"
user-invocable: true
---

# Schema and Data Migration (Thiet ke va Di tru Du lieu)

## Outcome | Dau ra
- EN: Produce target schema design for SQL tables or NoSQL documents.
- VN: Tao thiet ke schema muc tieu cho bang SQL hoac document NoSQL.
- EN: Produce versioned migration scripts that can be applied consistently across environments.
- VN: Tao script migration co version de ap dung dong bo giua cac moi truong.
- EN: Provide rollout, validation, and rollback plan for Dev, Staging, Production.
- VN: Dua ra ke hoach rollout, kiem thu, va rollback cho Dev, Staging, Production.

## When To Use | Khi nao dung
- EN: New project needs initial schema bootstrap.
- VN: Du an moi can khoi tao schema ban dau.
- EN: Existing system needs schema evolution without environment drift.
- VN: He thong hien tai can nang cap schema ma khong lech moi truong.
- EN: Team needs deterministic migration artifacts for CI/CD and release.
- VN: Doi ngu can migration artifact xac dinh de dung trong CI/CD va release.

## Required Inputs | Dau vao bat buoc
- EN: Business requirements, entities, and data retention/security rules.
- VN: Yeu cau nghiep vu, entity, va quy dinh luu tru/bao mat du lieu.
- EN: DB engine and version (PostgreSQL/MySQL/SQL Server/MongoDB/etc.).
- VN: He quan tri CSDL va phien ban (PostgreSQL/MySQL/SQL Server/MongoDB/... ).
- EN: Current schema state and target state.
- VN: Trang thai schema hien tai va schema muc tieu.
- EN: Migration window, uptime expectations, and data volume.
- VN: Cua so migration, yeu cau uptime, va quy mo du lieu.

## Resources | Tai nguyen dinh kem
- Liquibase master changelog template: [db.changelog-master.yaml.template](./assets/db.changelog-master.yaml.template)
- SQL bootstrap template: [V1__initial_schema.sql.template](./assets/V1__initial_schema.sql.template)
- NoSQL bootstrap template: [V1__initial_documents.js.template](./assets/V1__initial_documents.js.template)
- Validation checklist: [migration-quality-checklist.md](./references/migration-quality-checklist.md)

## Migration Conventions | Quy uoc migration
- EN: Default framework: Liquibase changelog-first.
- VN: Framework mac dinh: Liquibase changelog-first.
- EN: Keep a master changelog and add one changeSet per migration objective.
- VN: Duy tri master changelog va them mot changeSet cho moi muc tieu migration.
- EN: SQL naming convention: V{version}__{description}.sql (example: V1__initial_schema.sql), then reference file from Liquibase changeSet.
- VN: Quy uoc dat ten SQL: V{version}__{description}.sql (vi du: V1__initial_schema.sql), sau do tham chieu file trong Liquibase changeSet.
- EN: NoSQL naming convention: V{version}__{description}.js or tool-native migration file.
- VN: Quy uoc dat ten NoSQL: V{version}__{description}.js hoac format migration cua tool.
- EN: One migration objective per version to keep deployment reversible and auditable.
- VN: Moi version chi nen gom mot muc tieu migration de de rollback va kiem toan.

## Procedure | Quy trinh
1. Baseline and target definition | Xac dinh hien trang va muc tieu
- EN: Capture as-is schema or collection model and identify gaps to target state.
- VN: Thu thap schema/model hien tai va xac dinh khoang cach den muc tieu.
- EN: Label each change as create, alter, backfill, index, or cleanup.
- VN: Dan nhan moi thay doi theo nhom create, alter, backfill, index, cleanup.

2. Data model design | Thiet ke mo hinh du lieu
- EN (SQL): Define tables, keys, constraints, and indexes.
- VN (SQL): Dinh nghia bang, khoa, rang buoc, va chi muc.
- EN (NoSQL): Define document shape, partition strategy, and query indexes.
- VN (NoSQL): Dinh nghia cau truc document, chien luoc partition, va query index.
- EN: Explicitly document ownership and write/read patterns.
- VN: Mo ta ro ownership va mau doc/ghi du lieu.

3. Migration strategy selection | Chon chien luoc migration
- EN: Prefer expand-then-contract for backward-compatible releases.
- VN: Uu tien expand-then-contract cho release tuong thich nguoc.
- EN: For breaking changes, split migration into pre-deploy, deploy, and post-deploy phases.
- VN: Voi thay doi breaking, tach migration thanh pre-deploy, deploy, post-deploy.
- EN: Plan backfill separately for large datasets to reduce lock pressure.
- VN: Tach backfill rieng cho du lieu lon de giam lock pressure.

4. Generate migration scripts | Tao script migration
- EN: Generate initial schema script (V1__initial_schema.sql) for greenfield SQL projects.
- VN: Tao script khoi tao schema (V1__initial_schema.sql) cho du an SQL moi.
- EN: Generate incremental scripts per change unit (V2..., V3...).
- VN: Tao script tang dan theo tung nhom thay doi (V2..., V3...).
- EN: Generate or update Liquibase master changelog and register each migration as a separate changeSet.
- VN: Tao hoac cap nhat Liquibase master changelog va dang ky moi migration thanh changeSet rieng.
- EN: Ensure scripts and changeSets are deterministic, ordered, and safe for CI execution.
- VN: Dam bao script va changeSet xac dinh, dung thu tu, va an toan cho CI.

5. Validate and test | Kiem tra va thu nghiem
- EN: Apply migrations on clean Dev database and on snapshot of real-like data.
- VN: Chay migration tren DB Dev sach va tren ban sao du lieu gan thuc te.
- EN: Validate schema diff, query performance, and data correctness.
- VN: Kiem tra schema diff, hieu nang truy van, va tinh dung du lieu.
- EN: Add verification queries and health checks per migration.
- VN: Them truy van verify va health check cho tung migration.
- EN: Run the migration checklist before environment promotion.
- VN: Chay checklist migration truoc khi promote moi truong.

6. Rollout plan for environments | Ke hoach rollout theo moi truong
- EN: Promote in order Dev -> Staging -> Production with checkpoint approvals.
- VN: Trien khai theo thu tu Dev -> Staging -> Production kem checkpoint phe duyet.
- EN: Track Liquibase history, checksums, contexts, and labels in release notes.
- VN: Luu lich su Liquibase, checksum, context, va label trong release notes.

7. Rollback and recovery design | Thiet ke rollback va phuc hoi
- EN: Define rollback path for each version (where feasible) and fallback strategy when not reversible.
- VN: Dinh nghia rollback cho tung version (neu kha thi) va fallback khi khong the dao nguoc.
- EN: Include backup/restore checkpoint for high-risk migrations.
- VN: Bao gom moc backup/restore cho migration rui ro cao.

8. Quality gate before handoff | Cong gate chat luong truoc ban giao
- EN: Confirm cross-environment determinism and no drift.
- VN: Xac nhan tinh xac dinh va khong drift giua moi truong.
- EN: Confirm compatibility with application release plan.
- VN: Xac nhan tuong thich voi ke hoach release ung dung.
- EN: Confirm observability: logs, metrics, and alert thresholds during migration.
- VN: Xac nhan kha nang quan sat: log, metric, nguong canh bao khi migration.

## Decision Branches | Nhanh quyet dinh
- Greenfield vs Brownfield:
- EN: If no existing schema, generate full baseline migration first.
- VN: Neu chua co schema, tao baseline migration day du truoc.
- EN: If existing production data exists, generate delta migrations only.
- VN: Neu da co du lieu production, chi tao migration dang delta.

- SQL vs NoSQL:
- EN: SQL when transactional integrity and relational constraints dominate.
- VN: Chon SQL khi can tinh toan ven giao dich va rang buoc quan he.
- EN: NoSQL when flexible schema and horizontal scale dominate.
- VN: Chon NoSQL khi can linh hoat schema va scale ngang.

- Online vs Offline migration:
- EN: Online migration if uptime is strict and change can be phased.
- VN: Online migration neu uptime nghiem ngat va co the tach giai doan.
- EN: Offline migration if change is highly invasive and downtime is acceptable.
- VN: Offline migration neu thay doi qua xam lan va downtime duoc chap nhan.

- Backward compatibility:
- EN: If old and new app versions run concurrently, avoid destructive changes first.
- VN: Neu app cu va moi chay song song, tranh thao tac pha huy o giai doan dau.

- SQL-first default:
- EN: If DB type is not specified, default to SQL schema design and Liquibase-managed SQL migration files.
- VN: Neu chua xac dinh loai DB, mac dinh thiet ke schema SQL va migration SQL duoc quan ly boi Liquibase.
- EN: Use NoSQL guidance only when workload or requirements explicitly justify document-first modeling.
- VN: Chi dung huong dan NoSQL khi workload hoac yeu cau thuc su can mo hinh document-first.

## Completion Criteria | Tieu chi hoan thanh
- EN: Migration scripts are versioned, ordered, and executable end-to-end.
- VN: Script migration co version, dung thu tu, va chay tron ven end-to-end.
- EN: Liquibase changelog validates successfully and checksum history is clean.
- VN: Liquibase changelog validate thanh cong va lich su checksum sach.
- EN: Dev, Staging, and Production strategy is explicit and synchronized.
- VN: Chien luoc Dev, Staging, Production ro rang va dong bo.
- EN: Data correctness, performance impact, and rollback readiness are validated.
- VN: Tinh dung du lieu, tac dong hieu nang, va kha nang rollback da duoc kiem chung.
- EN: Drift prevention controls are documented.
- VN: Co tai lieu hoa co che ngan drift.

## Output Contract | Mau dau ra bat buoc
When invoked, return these sections in order:
1. Data Context and Requirements | Ngu canh du lieu va yeu cau
2. Target Schema or Document Design | Thiet ke schema/document muc tieu
3. Migration Strategy and Version Plan | Chien luoc migration va ke hoach version
4. Generated Migration Scripts | Cac script migration duoc tao
5. Environment Rollout Plan (Dev/Staging/Production) | Ke hoach rollout theo moi truong
6. Validation and Rollback Plan | Ke hoach kiem tra va rollback
7. Quality Gate Result | Ket qua cong gate
8. Risks and Open Questions | Rui ro va cau hoi mo

## Example Prompts | Vi du prompt
- /schema-data-migration Design PostgreSQL schema for order, payment, shipment and generate V1__initial_schema.sql.
- /schema-data-migration Evolve existing MongoDB collections for audit logging and generate versioned migration scripts.
- /schema-data-migration Plan zero-downtime migration for splitting user_profile into profile and preferences tables.
