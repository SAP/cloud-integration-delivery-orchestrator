[![REUSE status](https://api.reuse.software/badge/github.com/SAP/cloud-integration-delivery-orchestrator)](https://api.reuse.software/info/github.com/SAP/cloud-integration-delivery-orchestrator)

# Cloud Integration Delivery Orchestrator

Orchestrate SAP Cloud Integration delivery across multi-tenant landscapes — replace repetitive manual steps with a single governed workflow.

## Try It on SAP BTP

Deploy the pre-built Docker image to SAP BTP Cloud Foundry with the supplied MTA descriptor.

**[Start setup :rocket:](#setup)**

> For evaluation, use a non-productive landscape.

**Need help?** Open a [GitHub Issue](https://github.com/SAP/cloud-integration-delivery-orchestrator/issues) — actively maintained by the team behind it.

## Who is this for?

This project is a fit if you:

- run **more than one SAP Cloud Integration tenant** and coordinate releases across them;
- use **SAP Cloud Transport Management** for transport and routing; and
- can deploy to a **BTP Cloud Foundry** subaccount with the required service entitlements.

## Setup

### Before you start

| Requirement | Details |
|-------------|---------|
| SAP BTP CF environment | Subaccount with Cloud Foundry enabled |
| CF CLI | [Download](https://github.com/cloudfoundry/cli/releases) |
| Multiapps plugin | `cf install-plugin multiapps` |
| MTA Build Tool | `npm install -g mbt` |

Ensure the following entitlements are assigned to your subaccount:

| Service | Plan | Required |
|---------|------|----------|
| Authorization and Trust Management Service (xsuaa) | application + apiaccess | Yes |
| PostgreSQL, Hyperscaler Option (postgresql-db) | development / standard | Yes |
| Destination Service (destination) | lite | Yes |
| Connectivity Service (connectivity) | lite | No (opt-in) |
| Cloud Logging (cloud-logging) | standard | No (opt-in) |
| SAP Alert Notification service (alert-notification) | free | No (opt-in) |

### 1. Configure your landscape

Clone the repository and create your landscape file:

```bash
git clone https://github.com/SAP/cloud-integration-delivery-orchestrator.git
cd cloud-integration-delivery-orchestrator

cp example.mtaext your-landscape.mtaext
# Edit your-landscape.mtaext
```

Minimal configuration in `your-landscape.mtaext`:

```yaml
_schema-version: "3.1.0"
ID: cloud-integration-delivery-orchestrator-myorg
extends: cloud-integration-delivery-orchestrator

parameters:
  # Required — URL prefix (e.g. orchestrator → orchestrator.cfapps.eu10.hana.ondemand.com)
  app-host-prefix: "your-orchestrator-cf-app-prefix"
  # Required — defaults to :latest; pin to a release tag if you prefer a fixed version
  docker-image: "ghcr.io/sap/cloud-integration-delivery-orchestrator:latest"
  # Optional — PostgreSQL plan: development (default) or standard (production)
  # db-service-plan: "standard"
```

If you already have service instances, override the `service-name` in the resources section of your `.mtaext`. See `example.mtaext` for all available overrides.

### 2. Build and deploy

```bash
mbt build

cf login -a https://api.cf.<region>.hana.ondemand.com -o <org> -s <space>

cf deploy mta_archives/cloud-integration-delivery-orchestrator_1.0.0.mtar -e your-landscape.mtaext
```

The MTA archive file name follows the `version` in `mta.yaml` (currently `1.0.0`). The deployer will:

1. Create or update service instances (XSUAA, PostgreSQL, Destination, etc.)
2. Pull the Docker image from the registry
3. Start the application with all service bindings

### 3. Complete setup

- [ ] **Assign a role collection** in **BTP Cockpit → Security → Role Collections**, then assign it to your user:

| Role Collection | Who | Access |
|----------------|-----|--------|
| Delivery Orchestrator Administrator | Platform administrators | Full access — manage tenants, rules, system config |
| Delivery Orchestrator Operator | DevOps engineers | Create/execute delivery requests, trigger version compare |
| Delivery Orchestrator Viewer | Stakeholders | Read-only access to all views |

- [ ] **Open the application** at `https://<app-host-prefix>.cfapps.<region>.hana.ondemand.com` and sign in.
- [ ] **Configure System Configuration** — the mandatory setting is the **TMS destination** for your transport landscape. CPI tenant connections are configured separately when you register tenants.
- [ ] Run **Tenant Bootstrap** for a new tenant to validate prerequisites before the first delivery.

**You are ready** when you can log in, see the home page, and open System Configuration.

## Troubleshooting

<!-- Reserved for validated deployment issues. Report problems via GitHub Issues in the meantime. -->

## About This Project

Cloud Integration Delivery Orchestrator automates the transport, deployment, and governance of SAP Cloud Integration (CPI) artifacts across multiple tenants. It coordinates transport request generation, artifact export, and deployment into a single governed workflow via TMS and CPI OData APIs.

The project distributes a pre-built Docker image deployed to SAP BTP Cloud Foundry via an MTA descriptor. This repository holds the deployment configuration (MTA descriptor, XSUAA security, extension templates); the application source code lives in the two repositories linked below.

## Source Code

The published Docker image is built from two open-source repositories:

| Repository | Component | Stack |
|------------|-----------|-------|
| [cloud-integration-delivery-orchestrator-srv](https://github.com/SAP/cloud-integration-delivery-orchestrator-srv) | Backend service (REST API, WebSocket, XSUAA auth) | Go, Gin, GORM, PostgreSQL |
| [cloud-integration-delivery-orchestrator-ui](https://github.com/SAP/cloud-integration-delivery-orchestrator-ui) | Web frontend (embedded into the backend image at build time) | Vue 3, TypeScript, Vite, UI5 Web Components |

## Optional services

Three services are inactive by default in `mta.yaml`. Enable them in your `.mtaext` when needed, then redeploy.

### Connectivity Service

Not required for core delivery workflows. Enable it when your landscape needs on-premise connectivity through **SAP Cloud Connector** — for example, when GitHub is **GitHub Enterprise Server (GHES)** on your corporate network. In that case, Cloud Connector is required to reach GHES, which in turn enables **GitHub sync** and **BPMN Visual Diff** against repositories hosted there.

```yaml
resources:
  - name: cloud-integration-delivery-orchestrator-conn
    active: true
```

### SAP Cloud Logging

Centralized log aggregation, distributed tracing, and metrics. Requires a `cloud-logging` entitlement.

```yaml
resources:
  - name: cloud-integration-delivery-orchestrator-cls
    active: true
```

Without this entitlement, omit the block — the application deploys normally with no-op tracing and metrics.

### SAP Alert Notification service

Delivery event notifications (e.g. Email, Slack, Microsoft Teams). Requires an `alert-notification` entitlement.

```yaml
resources:
  - name: cloud-integration-delivery-orchestrator-ans
    active: true
```

## Upgrade

Most deployments keep `docker-image` on `:latest`. To pick up a new release:

1. Rebuild: `mbt build`
2. Redeploy: `cf deploy mta_archives/cloud-integration-delivery-orchestrator_<version>.mtar -e your-landscape.mtaext`

If you pin a specific image tag in `.mtaext`, update that tag before redeploying. Service bindings and database are preserved automatically.

## Uninstall

```bash
cf undeploy cloud-integration-delivery-orchestrator --delete-services --delete-service-keys
```

> **Warning**: `--delete-services` removes the PostgreSQL instance and all data. Omit this flag to preserve the database.

## Support, Feedback, Contributing

This project is open to feature requests/suggestions, bug reports etc. via [GitHub issues](https://github.com/SAP/cloud-integration-delivery-orchestrator/issues). Contribution and feedback are encouraged and always welcome. For more information about how to contribute, the project structure, as well as additional contribution information, see our [Contribution Guidelines](CONTRIBUTING.md).

## Security / Disclosure

If you find any bug that may be a security problem, please follow our instructions at [in our security policy](https://github.com/SAP/cloud-integration-delivery-orchestrator/security/policy) on how to report it. Please do not create GitHub issues for security-related doubts or problems.

## Code of Conduct

We as members, contributors, and leaders pledge to make participation in our community a harassment-free experience for everyone. By participating in this project, you agree to abide by its [Code of Conduct](https://github.com/SAP/.github/blob/main/CODE_OF_CONDUCT.md) at all times.

## Licensing

Copyright 2026 SAP SE or an SAP affiliate company and cloud-integration-delivery-orchestrator contributors. Please see our [LICENSE](LICENSE) for copyright and license information. Detailed information including third-party components and their licensing/copyright information is available [via the REUSE tool](https://api.reuse.software/info/github.com/SAP/cloud-integration-delivery-orchestrator).
