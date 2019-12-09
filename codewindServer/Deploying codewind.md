# Deploying codewind

## Installing a remote codewind-pfe

###### Steps for user of cwctl

cwctl called to Install Codewind on a remote cloud installation

`cwctl --insecure install remote  —namespace {your_namespace}  --kadminuser admin --kadminpass <keycloakPassword>  --krealm codewind --kclient codewind --kdevuser developer --kdevpass <userPassword>`

The cli has been design to do almost all the steps required to deploy
codewind remotely by chaining together a number of Kubernetes api
commands. 

An example deployment on Kubernetes :

```
~ cwctl --insecure install remote \
  --namespace codewind  \
  --kadminuser admin \
  --kadminpass passw0rd  \
  --krealm codewind \
  --kclient codewind  \
  --kdevuser developer \
  --kdevpass passw0rd
INFO[0000] Checking namespace codewind exists
INFO[0000] Creating codewind namespace
INFO[0000] Using namespace : codewind
INFO[0000] Container images :
INFO[0000] eclipse/codewind-pfe-amd64:latest
INFO[0000] eclipse/codewind-performance-amd64:latest
INFO[0000] eclipse/codewind-keycloak-amd64:latest
INFO[0000] eclipse/codewind-gatekeeper-amd64:latest
INFO[0000] Running on openshift: false
INFO[0000] Attempting to discover Ingress Domain
INFO[0000] Using ingress domain: 10.99.117.86.nip.io
INFO[0000] Creating service account definition 'codewind-k3rdhvxk'
INFO[0000] Creating service account definition 'keycloak-k3rdhvxk'
INFO[0000] Creating codewind-keycloak-k3rdhvxk.10.99.117.86.nip.io server Key
INFO[0000] Creating codewind-keycloak-k3rdhvxk.10.99.117.86.nip.io server certificate
INFO[0000] Creating Codewind Keycloak PVC
INFO[0000] Deploying Codewind Keycloak Secrets
INFO[0000] Deploying Codewind Keycloak TLS Secrets
INFO[0000] Deploying Codewind Keycloak Ingress
INFO[0000] Waiting for pod: codewind-keycloak-k3rdhvxk
INFO[0000] codewind-keycloak-k3rdhvxk, phase: Pending Unschedulable
INFO[0001] codewind-keycloak-k3rdhvxk, phase: Pending ContainersNotReady
INFO[0005] codewind-keycloak-k3rdhvxk, phase: Running
INFO[0005] Waiting for Keycloak to start
..........
INFO[0028] Configuring Keycloak...
INFO[0028] https://codewind-keycloak-k3rdhvxk.10.99.117.86.nip.io
INFO[0029] Creating Keycloak realm
INFO[0030] Checking for Keycloak client 'codewind-k3rdhvxk'
INFO[0030] Creating Keycloak client
INFO[0030] Creating access role 'codewind-k3rdhvxk' in realm 'codewind'
INFO[0030] Creating Keycloak initial user
INFO[0030] Updating Keycloak user password
INFO[0030] Grant 'developer' access to this deployment
INFO[0030] Adding role 'codewind-k3rdhvxk' to user : 'c00750c7-4b10-4317-b328-e21c4c930913'
INFO[0030] Fetching client secret
INFO[0030] Checking if 'eclipse-codewind-x.x.dev' cluster access roles are installed
INFO[0030] Cluster roles 'eclipse-codewind-x.x.dev' already installed
INFO[0030] Checking if 'codewind-rolebinding-k3rdhvxk' role bindings exist
INFO[0030] Adding 'codewind-rolebinding-k3rdhvxk' role binding
INFO[0030] Creating and setting Codewind PVC codewind-pfe-pvc-k3rdhvxk to 1Gi
INFO[0030] Deploying Codewind Service
INFO[0030] Waiting for pod: codewind-pfe-k3rdhvxk
INFO[0030] codewind-pfe-k3rdhvxk, phase: Pending Unschedulable
INFO[0041] codewind-pfe-k3rdhvxk, phase: Pending ContainersNotReady
INFO[0084] codewind-pfe-k3rdhvxk, phase: Running
INFO[0084] Deploying Codewind Performance Dashboard
INFO[0084] Waiting for pod: codewind-performance-k3rdhvxk
INFO[0084] codewind-performance-k3rdhvxk, phase: Pending
INFO[0084] codewind-performance-k3rdhvxk, phase: Pending ContainersNotReady
INFO[0091] codewind-performance-k3rdhvxk, phase: Running
INFO[0091] Preparing Codewind Gatekeeper resources
INFO[0091] Creating codewind-gatekeeper-k3rdhvxk.10.99.117.86.nip.io server Key
INFO[0091] Creating codewind-gatekeeper-k3rdhvxk.10.99.117.86.nip.io server certificate
INFO[0092] Deploying Codewind Gatekeeper Secrets
INFO[0092] Deploying Codewind Gatekeeper Session Secrets
INFO[0092] Deploying Codewind Gatekeeper TLS Secrets
INFO[0092] Deploying Codewind Gatekeeper Deployments
INFO[0092] Deploying Codewind Gatekeeper Service
INFO[0092] Deploying Codewind Gatekeeper Ingress
INFO[0092] Waiting for pod: codewind-gatekeeper-k3rdhvxk
INFO[0092] codewind-gatekeeper-k3rdhvxk, phase: Pending
INFO[0092] codewind-gatekeeper-k3rdhvxk, phase: Pending ContainersNotReady
INFO[0100] codewind-gatekeeper-k3rdhvxk, phase: Running
INFO[0100] Waiting for Codewind Gatekeeper to start on https://codewind-gatekeeper-k3rdhvxk.10.99.117.86.nip.io
..
INFO[0101] Waiting for Codewind PFE to start
.
INFO[0101] Codewind is available at: https://codewind-gatekeeper-k3rdhvxk.10.99.117.86.nip.io
```


Pieces to note from above :

1.  The names of the docker images used for the deployment
2.  The keycloak server URL and the gatekeeper server URL is displayed
3.  The Keycloak client is displayed :  'codewind-k3rdhvxk'

All of the resources are deployed using a "workspaceID" this ID is the k3rdhvxk piece from the client ID and required when removing the remote deployment at a later date.

