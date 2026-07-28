# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this project adheres to [Semantic Versioning](https://semver.org/).

## [1.0.0] - 2026-07-28

### Added

- Initial public release
- MTA deployment descriptor for SAP BTP Cloud Foundry (Docker image module)
- XSUAA configuration with three role levels: Admin, Operator, Viewer
- Example MTA extension file for customer deployment configuration
- Opt-in SAP Cloud Logging integration (traces, metrics, logs via OTLP)
- Deployment documentation with prerequisites, setup, and troubleshooting

### Features (Application)

- Multi-tenant delivery orchestration for SAP Cloud Integration artifacts
- Transport request generation and import via TMS API
- Artifact deployment coordination across tenant landscapes
- Version comparison across tenants
- Tenant bootstrap wizard for rapid onboarding
- RBAC with Admin/Operator/Viewer role separation
- Real-time delivery progress tracking via WebSocket
- Audit logging for all delivery operations
