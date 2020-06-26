# Helm Releases

// TODO(hidde): short conclusive API description / outline

## Specification

A **helmrelease** object defines the source of a Helm chart by referencing an object
managed by [source-controller](https://github.com/fluxcd/source-controller), the interval
reconciliation should happen at, and a set of options to control the settings of the
automated Helm actions that are being performed.

```go
type HelmReleaseSpec struct {
	// ReleaseName used for the Helm release. Defaults to a composition of
	// '[TargetNamespace-]Name'.
	// +optional
	ReleaseName string `json:"releaseName,omitempty"`

	// TargetNamespace to target when performing operations for the HelmRelease.
	// Defaults to the namespace of the HelmRelease.
	// +optional
	TargetNamespace string `json:"targetNamespace,omitempty"`

	// SourceRef of the Helm chart source.
	// +required
	SourceRef corev1.TypedLocalObjectReference `json:"sourceRef"`

	// DependsOn holds a list of HelmReleases that must be ready before this
	// release can be installed.
	// +optional
	DependsOn []string `json:"dependsOn,omitempty"`

	// Interval at which to reconcile the release.
	// +required
	Interval metav1.Duration `json:"interval"`

	// Suspend tells the reconciler to suspend reconciliations for
	// this Helm release. Defaults to false.
	// +optional
	Suspend bool `json:"suspend,omitempty"`

	// Timeout for Helm actions performed. Defaults to 'Interval' duration.
	// +optional
	Timeout *metav1.Duration `json:"timeout,omitempty"`

	// Wait tells the reconciler to wait with marking a Helm action as
	// successful until all resources are in a ready state.
	// If set, it will wait for as long as 'Timeout'.
	// +optional
	Wait bool `json:"wait,omitempty"`

	// MaxHistory is the number of revisions saved by Helm for this release.
	// Use '0' for an unlimited number of revisions; defaults to '10'.
	// +optional
	MaxHistory *int `json:"maxHistory,omitempty"`

	// Install holds the configuration for performing Helm install actions.
	// +optional
	Install *struct {
		// Timeout for the Helm install action.
		// Defaults to the main 'Timeout' duration.
		// +optional
		Timeout *metav1.Duration `json:"timeout,omitempty"`

		// Wait tells the reconciler to wait with marking the Helm install as
		// successful until all resources are in a ready state. When set, it will
		// wait for as long as 'Timeout'. Defaults to the main 'Wait' value.
		// +optional
		Wait bool `json:"wait,omitempty"`

		// SkipCRDs will mark this Helm release to skip the creation of CRDs during
		// the installation.
		// +optional
		SkipCRDs bool `json:"skipCRDs,omitempty"`
	} `json:"install,omitempty"`
	
	// Upgrade holds the configuration for performing Helm upgrade actions.
	// +optional
	Upgrade *struct {
		// Timeout for the Helm upgrade action. Defaults to the main
		// 'Timeout' duration.
		// +optional
		Timeout *metav1.Duration `json:"timeout,omitempty"`

		// Wait tells the reconciler to wait with marking the Helm upgrade as
		// successful until all resources are in a ready state. When set, it will
		// wait for as long as 'Timeout'. Defaults to the main 'Wait' value.
		// +optional
		Wait bool `json:"wait,omitempty"`

		// ResetValues resets all values to the chart defaults, so that the
		// configuration applied is what is defined in the resource values.
		// Defaults to true to ensure declarative behaviour.
		// +optional
		ResetValues *bool `json:"resetValues,omitempty"`
	} `json:"upgrade,omitempty"`

	// Test holds the configuration for enabling and performing Helm
	// test actions.
	// +optional
	Test *struct {
		// Enable tells the reconciler to perform a test action
		// on the configured condition reasons. Defaults to false.
		// +optional
		Enable bool `json:"enable,omitempty"`

		// OnConditionReason defines on what condition reasons a Helm test
		// should be performed. Defaults to ['InstallSucceeded', 'UpgradeSucceeded'].
		// +optional
		OnConditionReason []string `json:"onConditionReason,omitempty"`

		// Timeout for the Helm test action. Defaults to the main
		// 'Timeout' duration.
		// +optional
		Timeout *metav1.Duration `json:"timeout,omitempty"`
	} `json:"test,omitempty"`

	// Rollback holds the configuration for enabling and performing Helm
	// rollback actions.
	// +optional
	Rollback *struct {
		// Enable tells the reconciler to perform a rollback action
		// on the configured condition reasons. Defaults to false.
		// +optional
		Enable bool `json:"enable,omitempty"`
		
		// Retry tells the reconciler to retry upgrades on a rolled back release
		// until 'MaxRetries' is reached. 
		Retry bool `json:"retry,omitempty"`

		// MaxRetries is the maximum amount of retries that should be attempted
		// for a rolled back release. Defaults to '5' when omitted, use '0' for
		// an unlimited amount of retries.
		// +optional
		MaxRetries *int `json:"maxRetries,omitempty"`

		// OnConditionReason defines on what condition reasons a Helm rollback
		// should happen. Defaults to ['UpgradeFailed', 'TestFailed'].
		// +optional
		OnConditionReason []string `json:"onConditionReason,omitempty"`

		// Timeout for the individual operations of a Helm rollback action.
		// Defaults to the main 'Timeout' duration.
		// +optional
		Timeout *metav1.Duration `json:"timeout,omitempty"`

		// DisableHooks prevents hooks from running during rollback. Defaults
		// to false.
		// +optional
		DisableHooks bool `json:"disableHooks,omitempty"`

		// Force resource update through delete/recreation if needed. Defaults
		// to false.
		// +optional
		Force bool `json:"force,omitempty"`

		// Recreate performs pod restarts for the resource if applicable. Defaults
		// to false.
		// +optional
		Recreate bool `json:"recreate,omitempty"`
	} `json:"rollback,omitempty"`

	// ValuesFromSources holds the references to values from sources (ConfigMaps,
	// Secrets, ExternalSourceRefs).
	// +optional
	ValuesFrom []ValuesFromSource `json:"valuesFrom,omitempty"`

	// Values holds the values for this Helm release.
	// +optional
	Values map[string]interface{} `json:"values,omitempty"`
}

type ValuesFromSource struct {
	// The reference to a config map with release values.
	// +optional
	ConfigMapKeyRef *OptionalConfigMapKeySelector `json:"configMapKeyRef,omitempty"`

	// The reference to a secret with release values.
	// +optional
	SecretKeyRef *OptionalSecretKeySelector `json:"secretKeyRef,omitempty"`

	// The reference to an external source with release values.
	// +optional
	ExternalSourceRef *ExternalSourceSelector `json:"externalSourceRef,omitempty"`
}
```

Status condition types:

```go
const (
	// ReadyCondition represents the fact that the HelmRelease
	// has been successfully enrolled on the cluster.
	ReadyCondition string = "Ready"
)
```

Status condition reasons:

```go
const (
	// InstallSucceededReason represents the fact that the Helm install for the release succeed.
	InstallSucceededReason string = "InstallSucceeded"

	// InstallFailedReason represents the fact that the Helm install for the release failed.
	InstallFailedReason string = "InstallFailed"

	// UpgradeSucceededReason represents the fact that the Helm upgrade for the release succeed.
	UpgradeSucceededReason string = "UpgradeSucceeded"

	// UpgradeFailedReason represents the fact that the Helm upgrade for the release failed.
	UpgradeFailedReason string = "UpgradeFailed"

	// TestSucceededReason represents the fact that the Helm test for the release succeeded.
	TestSucceededReason string = "TestSucceeded"

	// TestFailedReason represents the fact that the Helm test for the release failed.
	TestFailedReason string = "TestFailed"
	
	// RollbackSucceededReason represents the fact that the Helm rollback for the release succeeded.
	RollbackSucceededReason string = "RollbackSucceeded"
	
	// RollbackFailedReason represent the fact that the Helm rollback for the release failed.
	RollbackFailedReason string = "RollbackFailed"

	// ArtifactFailedReason represents the fact that the artifact download for the release failed.
	ArtifactFailedReason string = "ArtifactFailed"

	// DependencyNotReady represents the fact that the one of the dependencies is not ready.
	DependencyNotReadyReason string = "DependencyNotReady"

	// InitializedReason represents the fact that a given resource has been initialized.
	InitializedReason string = "Initialized"

	// ProgressingReason represents the fact that the reconciliation for the resource is underway.
	ProgressingReason string = "Progressing"

	// SuspendedReason represents the fact that the HelmRelease reconciliation is suspended.
	SuspendedReason string = "Suspended"
)
```

## Source reference

The `HelmRelease` `spec.sourceRef` is a reference to an object managed by
[source-controller](https://github.com/fluxcd/source-controller). When the source
[revision](https://github.com/fluxcd/source-controller/blob/master/docs/spec/v1alpha1/common.md#source-status) 
changes, it generates a Kubernetes event that triggers a new release.

Supported source types:

* [HelmChart](https://github.com/fluxcd/source-controller/blob/master/docs/spec/v1alpha1/helmcharts.md)

## Reconciliation

The `HelmRelease` `spec.interval` tells the reconciler at which interval to reconcile the release. The
interval time units are `s`, `m` and `h` e.g. `interval: 5m`, the minimum value should be over 60 seconds.

The reconciliation can be suspended by setting `spec.suspend` to `true`.

The reconciler can be told to reconcile the `HelmRelease` outside of the specified interval
by annotating the object with:

```go
const (
	// ReconcileAtAnnotation is the annotation used for triggering a
	// reconciliation outside of the specified schedule.
	ReconcileAtAnnotation string = "helm.fluxcd.io/reconcileAt"
)
```

On-demand execution example:

```bash
kubectl annotate --overwrite helmrelease/podinfo helm.fluxcd.io/reconcileAt="$(date +%s)"
```
