# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [0.0.2] - 2026-09-03

### Changed

- **Breaking (deployment):** renamed all MTA and XSUAA identifiers from `cpi-delivery`
  to `cloud-integration-delivery-orchestrator` — MTA ID, modules, resources, the XSUAA
  `xsappname`, role templates (`DeliveryOrchestratorAdministrator` / `Operator` / `Viewer`),
  and role collections (`Delivery Orchestrator Administrator` / `Operator` / `Viewer`).
  Existing deployments must reassign users to the new role collections in the BTP Cockpit.
- **Go module rename:** `mmt-delivery` → `github.com/SAP/cloud-integration-delivery-orchestrator-srv`,
  aligning with the public GitHub repo URL (Go ecosystem standard).
- **Notifier interface redesign:** method names now describe events, not channels
  (`OnApprovalRequested` / `OnStatusChanged` / `OnDeliveryComment`); recipient list parameter
  removed — routing is an implementation detail.
- **`defaultNotifier` → `jiraNotifier`:** renamed to reflect current responsibility
  (only Jira comment has active logic; email methods are no-ops pending ANS).

### Added

- **GitHub App authentication** — one-click registration via App Manifest flow as an
  alternative to PAT-based auth. Includes dual-mode support (PAT / GitHub App), App-mode
  repo discovery, install-pending deep-link, disconnect/uninstall mechanism, and App
  settings URL display.
- **Jira config panel** — dedicated System Config panel with destination name, resolved
  endpoint URL, enabled toggle, test connection, and BTP Destination guidance text.
  Replaces the generic Integration Registry row for Jira.
- **Git sync resilience** — orphan snapshot recovery, Re-sync mechanism, and empty repo
  bootstrap support.
- **Delivery UX improvements (RFC 026)** — DR name uniqueness (409 conflict), status
  filtering, included-tenants connectivity check.
- Home page empty-state: users without any assigned role now see guidance to request a
  Delivery Orchestrator role collection.

### Removed

- **Integration Registry** — generic `IntegrationConfig` model, CRUD API, and UI table
  removed. Jira has a dedicated config; cookie_service and github legacy rows cleaned up.
- **Email/SMTP notifications** — `pkg/notify/email.go`, gomail dependency, and SMTP
  configuration removed. Email notifications will be re-implemented via SAP Alert
  Notification Service (RFC 027 Phase 4).
- **Check All Connectivity** — button, `ConnectivityCheckResult` model, and result
  caching removed. Individual Test Connection buttons remain (pure real-time, no persist).

### Fixed

- Consolidated the destination service client onto the shared HTTP client for consistent
  timeout and TLS handling.
- Backend is now source of truth for auth-mode field ownership in Git config
  (prevents client from overwriting callback-authored fields in GitHub App mode).

## [0.0.1] - 2026-08-25

First verification pre-release. Validates the end-to-end ghcr.io publishing pipeline
(GitHub Actions build → artifact attestation → environment approval → image push) with the
complete initial feature set.

### Added

- Multi-tenant delivery orchestration for SAP Cloud Integration artifacts
- Transport request generation and import via TMS API
- Version comparison across tenants
- Tenant bootstrap wizard
- RBAC with Admin / Operator / Viewer roles
- Real-time delivery progress tracking (WebSocket)
- Opt-in SAP Cloud Logging integration (traces, metrics, logs)
- MTA-based one-command deployment to SAP BTP Cloud Foundry
