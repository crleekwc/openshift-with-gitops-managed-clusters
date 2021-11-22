# Single Node Gitops
This repository is created as a proof of concept on how you can use gitops to administer cluster wide configuration changes to your cluster. If you see anything in this repository that is best practice please reachout to me at chrilee@redhat.com or create and issue in this repository.

## Provide Cluster Wide Permissions For The Gitops Controller
The provided Cluster Role Binding provides cluster-admin to all of the service accounts that gitops uses. This will allow gitops to create resources in OpenShift managed name spaces. 
```
oc apply -f ClusterRoleBinding.openshift-gitops-cluster-admin.yaml
```

## Structure of the project
Currently this is the file structures that I have decided upon to use to distinguish the multiple environments. In this example you can see that the Applications folder is where you would store the base files for each Application to be configured. Applications in this context can mean an application or appliance or any resource that needs to be configured.
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


## References

