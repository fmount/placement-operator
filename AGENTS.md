# AGENTS.md - placement-operator

## Project overview

placement-operator is a Kubernetes operator that manages
[OpenStack Placement](https://docs.openstack.org/placement/latest/) (the
resource tracking and allocation service: resource providers, inventories,
traits, allocations, and resource classes) on OpenShift/Kubernetes. It is
part of the
[openstack-k8s-operators](https://github.com/openstack-k8s-operators) project.

Key Placement domain concepts: **resource providers**, **inventories**
(available resources per provider), **traits** (qualitative capabilities),
**allocations** (resource consumption tracking), **resource classes**
(VCPU, MEMORY_MB, DISK_GB, custom classes).

Go module: `github.com/openstack-k8s-operators/placement-operator`
API group: `placement.openstack.org`
API version: `v1beta1`

## Tech stack

| Layer | Technology |
|-------|------------|
| Language | Go (modules, multi-module workspace via `go.work`) |
| Scaffolding | [Kubebuilder v4](https://book.kubebuilder.io/) + [Operator SDK](https://sdk.operatorframework.io/) |
| CRD generation | controller-gen (DeepCopy, CRDs, RBAC, webhooks) |
| Config management | Kustomize |
| Packaging | OLM bundle |
| Testing | Ginkgo/Gomega + envtest (functional), KUTTL (integration) |
| Linting | golangci-lint (`.golangci.yaml`) |
| CI | Zuul (`zuul.d/`), Prow (`.ci-operator.yaml`), GitHub Actions |

## Custom Resources

| Kind | Purpose |
|------|---------|
| `PlacementAPI` | Manages the Placement API deployment (httpd/WSGI). This is the only CR -- there is no top-level "Placement" CR. |

The `PlacementAPI` CR has defaulting and validating admission webhooks.

## Directory structure

| Directory | Contents |
|-----------|----------|
| `api/v1beta1/` | CRD types (`placementapi_types.go`), conditions, webhook markers |
| `cmd/` | `main.go` entry point |
| `internal/controller/` | Reconciler: `placementapi_controller.go` |
| `internal/placement/` | Placement resource builders (deployment, volumes, config) |
| `internal/webhook/` | Webhook implementation |
| `templates/` | Config files mounted into pods via `OPERATOR_TEMPLATES` env var |
| `config/crd,rbac,manager,webhook/` | Generated Kubernetes manifests (CRDs, RBAC, deployment, webhooks) |
| `config/samples/` | Example CRs (Kustomize overlays). Includes TLS variant. |
| `test/functional/` | envtest-based Ginkgo/Gomega tests |
| `test/kuttl/` | KUTTL integration tests |
| `hack/` | Helper scripts (CRD schema checker, local webhook runner) |

## Build commands

After modifying Go code, always run: `make generate manifests fmt vet`.

## Code style guidelines

- Follow standard openstack-k8s-operators conventions and lib-common patterns.
- Use `lib-common` modules for conditions, endpoints, TLS, storage, and other
  cross-cutting concerns rather than re-implementing them.
- CRD types go in `api/v1beta1/`. Controller logic goes in
  `internal/controller/`. Resource-building helpers go in `internal/placement/`.
- Config templates are plain files in `templates/` -- they are mounted at
  runtime via the `OPERATOR_TEMPLATES` environment variable.
- Webhook logic is split between the kubebuilder markers in `api/v1beta1/` and
  the implementation in `internal/webhook/`.

## Testing

- Functional tests use the envtest framework with Ginkgo/Gomega and live in
  `test/functional/`.
- KUTTL integration tests live in `test/kuttl/`.
- Run all functional tests: `make test`.
- When adding a new field or feature, add corresponding test cases in
  `test/functional/` and update fixture data accordingly.

## Key dependencies

- [lib-common](https://github.com/openstack-k8s-operators/lib-common): shared modules for conditions, endpoints, database, TLS, secrets, etc.
- [infra-operator](https://github.com/openstack-k8s-operators/infra-operator): topology APIs.
- [mariadb-operator](https://github.com/openstack-k8s-operators/mariadb-operator): database provisioning.
- [keystone-operator](https://github.com/openstack-k8s-operators/keystone-operator): identity service registration.
