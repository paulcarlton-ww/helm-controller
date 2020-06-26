# Helm Controller

The Helm Controller is a Kubernetes operator, allowing one to declaratively manage Helm chart
releases with Kubernetes manifests.

## Motivation

The main goal is to provide an automated operator that can perform Helm actions (e.g.
install, upgrade, rollback, test) and continuously reconcile the state of Helm releases.

When provisioning a new cluster, one may wish to install Helm releases in a specific order, for
example because one relies on a service mesh admission controller managed by a `HelmRelease` and
the proxy injector must be functional before deploying applications into the mesh, or when
[Custom Resource Definitions are managed in a separate chart](https://helm.sh/docs/chart_best_practices/custom_resource_definitions/#method-2-separate-charts)
and this chart needs to be installed first.

// TODO(hidde): look into service account isolation / impersonation in the context of Helm.

When dealing with an incident, one may wish to suspend the reconciliation of some Helm releases,
without having to stop the reconciler and affect the whole cluster.

## Design

TBA
