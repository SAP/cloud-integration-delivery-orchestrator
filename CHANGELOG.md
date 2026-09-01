# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [0.0.2] - 2026-09-01

Second verification pre-release — refreshes the image from the latest source and
aligns deployment naming ahead of the 1.0.0 GA target.

### Changed

- **Breaking (deployment):** renamed all MTA and XSUAA identifiers from `cpi-delivery`
  to `cloud-integration-delivery-orchestrator` — MTA ID, modules, resources, the XSUAA
  `xsappname`, role templates (`DeliveryOrchestratorAdministrator` / `Operator` / `Viewer`),
  and role collections (`Delivery Orchestrator Administrator` / `Operator` / `Viewer`).
  Existing deployments must reassign users to the new role collections in the BTP Cockpit.

### Added

- Home page empty-state: users without any assigned role now see guidance to request a
  Delivery Orchestrator role collection, instead of a blank page.

### Fixed

- Consolidated the destination service client onto the shared HTTP client for consistent
  timeout and TLS handling.

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
