# Removing a remote Codewind deployment

The Codewind cli has been design to do almost all the steps required to remove a remote Codewind deployment by chaining together a number of Kubernetes operations.

There are two modes of use

1. Remove Codewind
2. Remove Keycloak

Since a Keycloak service can provide authentication services to many Codewind deployments in different namespaces, removing Keycloak must only be performed when you are sure it is no longer used.

###### Steps to remove Codewind only :

The cwctl remove remote command requires a namespace and workspaceID which are used to identify which Kubernetes resources to remove.

If you are unsure of the workspace ID it can be found as part of the ingress or Route url:

```
https://codewind-gatekeeper-k412oms7.apps.mycluster.X.X.X.X.nip.io
                            ^^^^^^^^
```

OR

in the Gatekeeper environment API

```json
{
  "auth_url": "https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io",
  "client_id": "codewind-k4137d9y",
  "workspace_id": "k4137d9y",    <----------
  "realm": "codewind",
  "codewind_version": "x.x.dev",
  "image_build_time": "20191211-081952"
}
```


```bash
cwctl remove remote --namespace {namespace} --workspace {workspaceID}
```

example :

```
cwctl remove remote --namespace myspace --workspace k4137d9y

INFO[0000] Running on openshift: true
INFO[0000] Checking namespace myspace exists
INFO[0000] Found 'myspace' namespace
INFO[0001] Removal summary:
INFO[0001] Codewind PFE Deployment: Removed
INFO[0001] Codewind PFE Service: Removed
INFO[0001] Codewind PFE PVC: Removed
INFO[0001] Codewind Performance Deployment: Removed
INFO[0001] Codewind Performance Service: Removed
INFO[0001] Codewind Gatekeeper Deployment: Removed
INFO[0001] Codewind Gatekeeper Service: Removed
INFO[0001] Codewind Gatekeeper Ingress: Removed
INFO[0001] Codewind Role Bindings: Removed
INFO[0001] Codewind Service Account: Removed
INFO[0001] Kubernetes namespace: CWCTL will not remove the namespace automatically, use 'kubectl delete namespace myspace' if you would like to remove it
```

**Notes:**

1. The namespace is name checked to be sure it exists
2. A WorkspaceID is used to identify which components need to be destroyed
3. The Kubernetes namespace is NOT removed. This is to safeguard any other applications in the namespace from being destroyed
4. The Codewind Cluster Roles are not removed since they may be shared with other Codewind deployments
5. Secrets are removed as part of the deployment removal

###### Steps to remove Keycloak only :

>WARNING: Removing Keycloak will break all remote deployments of Codewind that are using it for authentication services

When you are ready to remove keycloak use the command `cwctl remove keycloak` and supply the namespace and workspaceID of the Keycloak deployment you want to remove


```
cwctl remove keycloak  --namespace keycloak-only --workspace k412oms7

INFO[0000] Running on Openshift: true
INFO[0000] Checking namespace keycloak-only exists
INFO[0000] Found 'keycloak-only' namespace
INFO[0001] Removal summary:
INFO[0001] Keycloak Deployment: Removed
INFO[0001] Keycloak Service: Removed
INFO[0001] Keycloak PVC: Removed
INFO[0001] Keycloak Ingress: Removed
INFO[0001] Keycloak Secrets: Removed
INFO[0001] Keycloak Service Account: Removed
INFO[0001] Kubernetes namespace: CWCTL will not remove the namespace automatically, use 'kubectl delete namespace keycloak-only' if you would like to remove it
```

## Searching for all deployments of Codewind

Codewind and Keycloak deployments use labels to identify their components.

To get a list of all Codewind deployments on a cluster use the standard commands `kubectl` (Kubernetes)  or `oc`(Openshift) with a label selector eg :

```
kubectl get pods \
--selector=app=codewind-pfe \
--all-namespaces \
-o=jsonpath='{range .items[*]}{.status.startTime}{"\t"} {.metadata.labels.codewindWorkspace}{"\t"}{.metadata.namespace}{"\n"}{end}'

```

###### Which returns :

| Start Time           | WorkspaceID | Namespace      |
|----------------------|-------------|----------------|
| 2019-12-06T08:55:58Z | k3twyg0c    | codewind1      |
| 2019-12-11T12:07:41Z | k41903kr    | codewind-test  |
| 2019-12-09T07:54:15Z | k3y528xq    | codewind-test  |
| 2019-11-20T13:17:13Z | k37b98z0    | codewind-test  |
| 2019-12-02T23:33:33Z | k3p2kvu9    | codewind5-test |
