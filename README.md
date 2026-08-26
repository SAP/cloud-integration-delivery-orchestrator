[![REUSE status](https://api.reuse.software/badge/github.com/SAP/cloud-integration-delivery-orchestrator)](https://api.reuse.software/info/github.com/SAP/cloud-integration-delivery-orchestrator)

# Cloud Integration Delivery Orchestrator

Orchestrate SAP Cloud Integration delivery across multi-tenant landscapes — replace repetitive manual steps with a single governed workflow.

## About This Project

Cloud Integration Delivery Orchestrator automates the transport, deployment, and governance of SAP Cloud Integration (CPI) artifacts across multiple tenants. It coordinates transport request generation, artifact export, and deployment into a single governed workflow via TMS and CPI OData APIs.

The project distributes a pre-built Docker image deployed to SAP BTP Cloud Foundry via MTA descriptor. No source code is published — only the compiled application image and deployment configuration.

## Requirements

| Requirement | Details |
|-------------|---------|
| SAP BTP CF environment | Subaccount with Cloud Foundry enabled |
| CF CLI | [Download](https://github.com/cloudfoundry/cli/releases) |
| Multiapps plugin | `cf install-plugin multiapps` |
| MTA Build Tool | `npm install -g mbt` |

### Required Entitlements

Ensure the following entitlements are assigned to your subaccount:

| Service | Plan | Required |
|---------|------|----------|
| Authorization and Trust Management (xsuaa) | application, apiaccess | Yes |
| PostgreSQL on SAP BTP | development / standard | Yes |
| Destination Service | lite | Yes |
| Connectivity Service | lite | Yes |
| SAP Cloud Logging | standard | No (opt-in) |

## Setup

### 1. Deploy

```bash
# Clone this repository
git clone https://github.com/SAP/cloud-integration-delivery-orchestrator.git
cd cloud-integration-delivery-orchestrator

# Copy and configure the extension file
cp example.mtaext my-landscape.mtaext
# Edit my-landscape.mtaext with your values (see Configuration below)

# Build MTA archive (descriptor only — image pulled from registry at deploy time)
mbt build

# Login to Cloud Foundry
cf login -a https://api.cf.<region>.hana.ondemand.com -o <org> -s <space>

# Deploy
cf deploy mta_archives/cloud-integration-delivery-orchestrator_1.0.0.mtar -e my-landscape.mtaext
```

The MTA deployer will:
1. Create/update service instances (XSUAA, PostgreSQL, Destination, Connectivity)
2. Pull the Docker image from the registry
3. Start the application with all service bindings

### 2. Configure

Copy `example.mtaext` and fill in your environment values:

```yaml
_schema-version: "3.1.0"
ID: cloud-integration-delivery-orchestrator-myorg
extends: cloud-integration-delivery-orchestrator

parameters:
  app-host-prefix: "your-orchestrator-cf-app-prefix"
  docker-image: "ghcr.io/sap/cloud-integration-delivery-orchestrator:latest"
```

#### Parameter Reference

| Parameter | Required | Description |
|-----------|----------|-------------|
| `app-host-prefix` | Yes | URL prefix (e.g. `orchestrator` → `orchestrator.cfapps.eu10.hana.ondemand.com`) |
| `docker-image` | Yes | Full image reference with tag |
| `db-service-plan` | No | PostgreSQL plan: `development` (default) or `standard` (production) |

#### Binding to Existing Service Instances

If you already have service instances, override the `service-name` in the resources section of your `.mtaext`:

```yaml
resources:
  - name: cloud-integration-delivery-orchestrator-db
    parameters:
      service-name: my-existing-postgresql
```

See `example.mtaext` for all available overrides.

### 3. Post-Deployment

#### Assign Role Collections

In **BTP Cockpit → Security → Users → Assign Role Collection**:

| Role Collection | Who | Access |
|----------------|-----|--------|
| Delivery Orchestrator Administrator | Platform administrators | Full access — manage tenants, rules, system config |
| Delivery Orchestrator Operator | DevOps engineers | Create/execute delivery requests, trigger version compare |
| Delivery Orchestrator Viewer | Stakeholders | Read-only access to all views |

#### Configure Destinations

Configure CPI tenant connections in the application's **System Configuration** view. Each CPI tenant requires:
- CPI OData endpoint — for artifact management
- TMS endpoint — for transport request operations

## Observability (SAP Cloud Logging)

Cloud Integration Delivery Orchestrator integrates with SAP Cloud Logging (CLS) for centralized log aggregation, distributed tracing, and metrics. This feature is **opt-in** and requires a `cloud-logging` entitlement.

### Enabling CLS

Add the following to your `.mtaext`:

```yaml
resources:
  - name: cloud-integration-delivery-orchestrator-cls
    active: true
```

On next deploy:
1. A `cloud-logging` service instance is created with OTLP ingestion enabled
2. Application logs are automatically shipped via CF syslog drain
3. Traces and metrics are exported via OTLP/gRPC with mTLS

### Accessing Dashboards

```bash
cf create-service-key cloud-integration-delivery-orchestrator-cls my-key
cf service-key cloud-integration-delivery-orchestrator-cls my-key
```

Open the `dashboards-endpoint` URL with `dashboards-username` / `dashboards-password` from the service key.

### Without CLS Entitlement

Simply omit the CLS activation. The application deploys normally — all tracing and metrics code uses no-op implementations.

## Upgrade

1. Update `docker-image` tag in your `.mtaext` (e.g. `ghcr.io/sap/cloud-integration-delivery-orchestrator:1.1.0`)
2. Rebuild: `mbt build`
3. Redeploy: `cf deploy mta_archives/cloud-integration-delivery-orchestrator_<version>.mtar -e my-landscape.mtaext`

Service bindings and database are preserved automatically.

## Uninstall

```bash
cf undeploy cloud-integration-delivery-orchestrator --delete-services --delete-service-keys
```

> **Warning**: `--delete-services` removes the PostgreSQL instance and all data. Omit this flag to preserve the database.

## Troubleshooting

| Symptom | Check |
|---------|-------|
| App fails to start | `cf logs cloud-integration-delivery-orchestrator --recent` — verify image is accessible and service bindings are correct |
| Login redirect fails | `cf env cloud-integration-delivery-orchestrator` — verify both XSUAA bindings exist (application + apiaccess) |
| Database errors | Migrations run on startup — check logs: `cf logs cloud-integration-delivery-orchestrator --recent \| grep migrat` |

## Support, Feedback, Contributing

This project is open to feature requests/suggestions, bug reports etc. via [GitHub issues](https://github.com/SAP/cloud-integration-delivery-orchestrator/issues). Contribution and feedback are encouraged and always welcome. For more information about how to contribute, the project structure, as well as additional contribution information, see our [Contribution Guidelines](CONTRIBUTING.md).

## Security / Disclosure
If you find any bug that may be a security problem, please follow our instructions at [in our security policy](https://github.com/SAP/cloud-integration-delivery-orchestrator/security/policy) on how to report it. Please do not create GitHub issues for security-related doubts or problems.

## Code of Conduct

We as members, contributors, and leaders pledge to make participation in our community a harassment-free experience for everyone. By participating in this project, you agree to abide by its [Code of Conduct](https://github.com/SAP/.github/blob/main/CODE_OF_CONDUCT.md) at all times.

## Licensing

Copyright 2026 SAP SE or an SAP affiliate company and cloud-integration-delivery-orchestrator contributors. Please see our [LICENSE](LICENSE) for copyright and license information. Detailed information including third-party components and their licensing/copyright information is available [via the REUSE tool](https://api.reuse.software/info/github.com/SAP/cloud-integration-delivery-orchestrator).
