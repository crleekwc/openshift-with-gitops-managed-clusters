# OpenShift With Gitops Managed Clusters
This repository is created as a proof of concept on how you can use gitops to administer cluster wide configuration changes to your cluster. If you see anything in this repository that is best practice please reachout to me at chrilee@redhat.com or create an issue in this repository.

## Provide Cluster Wide Permissions For The Gitops Controller
The provided Cluster Role Binding provides cluster-admin to all of the service accounts that gitops uses. This will allow gitops to create resources in OpenShift managed namespaces. 
```
oc apply -f ClusterRoleBinding.openshift-gitops-cluster-admin.yaml
```

## Structure Of The Repository
Currently this is the file structures that I have decided upon to use to distinguish the multiple environments. In this example you can see that the Applications folder is where you would store the base files for each Application to be configured. Applications in this context could mean an application or appliance or any resource that needs to be configured. The Projects folder is where you store projects, projects in this context means our cluster environment we want to configure.
```
Repository
	- ClusterRoleBinding.yaml
	- README.md
	- Applications
		○ <application name>
			§ kustomize.yaml
			§ Base <--- The base configuration file that can be customized
	- Projects
		○ <environment name>
			§ kustomize.yaml <--- Manifests that collects all of the desired apps to install in the environment
			§ Overlays
				□ <app name>
					® Overlay
						◊ Kustomize.yaml <--- configuration customization for the each individual app
```

## How To Test Configurations
Run the following kustomize command to test if it will build successfully.
```
kustomize build Projects/sample-environment
```

### Sample Ouput:
```
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
    argocd.argoproj.io/sync-wave: "-1"
  name: openshift-local-storage
---
apiVersion: local.storage.openshift.io/v1
kind: LocalVolume
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
    argocd.argoproj.io/sync-wave: "1"
  name: image-registry-local-storage
  namespace: openshift-local-storage
spec:
  logLevel: Debug
  managementState: Managed
  storageClassDevices:
  - devicePaths:
    - /dev/sdb
    fsType: xfs
    storageClassName: image-registry
    volumeMode: Filesystem
---
apiVersion: local.storage.openshift.io/v1
kind: LocalVolume
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
    argocd.argoproj.io/sync-wave: "1"
  name: monitoring-local-storage
  namespace: openshift-local-storage
spec:
  logLevel: Debug
  managementState: Managed
  storageClassDevices:
  - devicePaths:
    - /dev/sdc
    fsType: xfs
    storageClassName: monitoring
    volumeMode: Filesystem
---
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
    argocd.argoproj.io/sync-wave: "0"
  name: local-operator-group
  namespace: openshift-local-storage
spec:
  targetNamespaces:
  - openshift-local-storage
---
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
    argocd.argoproj.io/sync-wave: "0"
  name: local-storage-operator
  namespace: openshift-local-storage
spec:
  channel: stable
  installPlanApproval: Automatic
  name: local-storage-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
```
## Skeleton branch
I left a skeleton branch available, of just one application(local storage operator) and a sample environment, as a starting point for anyone that wants to use this repository to start their own project.

## Things TODO
- Figure out way that SealedSecrets will integrate into this methodology.
 

## References
- https://github.com/redhat-cop/gitops-catalog
- https://github.com/gitops-working-group/gitops-working-group
- https://github.com/redhat-developer/openshift-gitops-getting-started
- https://docs.google.com/document/d/1147S5yOdj5Golj3IrTBeeci2E1CjAkieGCcl0w90BS8/edit#heading=h.w8ey4tssu45h