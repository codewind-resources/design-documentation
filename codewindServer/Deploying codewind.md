# Deploying Codewind

Before a remote deployment of Codewind can begin, login to your cluster using for example `oc login`. The account used needs to be an 'adminstrator account' with ability to create service accounts, cluster roles, routes/ingress, PV claims, pods, deployments and secrets. This account is only required for deplying instances. Users of the Codewind remote instance will communicate directly and not require any special Kubernetes accounts.  Codewind users authenticate using the deployed Keycloak service. 

## Installing a remote codewind-pfe with Keycloak

###### Steps for user of cwctl

cwctl command to Install Codewind on a remote cloud cluster.

`cwctl --insecure install remote \
—namespace {your_namespace} \
--kadminuser admin \
--kadminpass <keycloakPassword> \
--krealm codewind \
--kclient codewind \
--kdevuser developer \
--kdevpass <userPassword>`


The cli has been design to do almost all the steps required to deploy codewind remotely by chaining together a number of Kubernetes api commands.

###### An example deployment on Kubernetes :

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


## Deploying just Keycloak

When setting up a pilot or test deployment the previous section installs all components at once. This saves time and simplifies the install however there are some duplication of Keycloak pods which is not necessary.

The Codewind CLI has options to deploy just a Keycloak instance which can be shared and configured for use with many Codewind remote deployments.

To install just Keycloak the `--konly` flag can be added to the cli command.

eg:

```
cwctl --insecure install remote \
  --namespace keycloak-only  \
  --kadminuser admin \
  --kadminpass passw0rd  \
  --krealm codewind \
  --kclient codewind  \
  --kdevuser developer \
  --kdevpass passw0rd \
  --ingress "apps.mycluster.X.X.X.X.nip.io" \
  --konly

INFO[0000] Checking namespace keycloak-only exists
INFO[0000] Creating keycloak-only namespace
INFO[0000] Using namespace : keycloak-only
INFO[0000] Container images :
INFO[0000] eclipse/codewind-pfe-amd64:latest
INFO[0000] eclipse/codewind-performance-amd64:latest
INFO[0000] eclipse/codewind-keycloak-amd64:latest
INFO[0000] eclipse/codewind-gatekeeper-amd64:latest
INFO[0000] Running on openshift: true
INFO[0000] Using ingress domain: apps.mycluster.X.X.X.X.nip.io
INFO[0000] Creating service account definition 'keycloak-k412oms7'
INFO[0000] Creating codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io server Key
INFO[0000] Creating codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io server certificate 
INFO[0000] Creating Codewind Keycloak PVC
INFO[0000] Deploying Codewind Keycloak Secrets
INFO[0000] Deploying Codewind Keycloak TLS Secrets
INFO[0000] Waiting for pod: codewind-keycloak-k412oms7
INFO[0000] codewind-keycloak-k412oms7, phase: Pending Unschedulable
INFO[0009] codewind-keycloak-k412oms7, phase: Pending ContainersNotReady
INFO[0019] codewind-keycloak-k412oms7, phase: Running
INFO[0019] Waiting for Keycloak to start
.......
INFO[0053] Configuring Keycloak...
INFO[0053] https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io
INFO[0054] Creating Keycloak realm
INFO[0056] Keycloak is available at: https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io

```

Pieces to note from above :

1. The Keycloak server URL is displayed on completion
2. The Keycloak deployment is not populated with any users, roles or client configurations

## Deploying just Codewind

Once Keycloak has been deployed any further deployment of Codewind can make use of that instance of Keycloak.

The Codewind CLI can perform the necessary steps to configure an existing Keycloak instance for use with a new deployment of remote Codewind by using the flag `--kurl` and providing a URL of the Keycloak service.

eg :

```
cwctl --insecure install remote \
  --namespace codewind-only  \
  --kadminuser admin \
  --kadminpass passw0rd  \
  --krealm codewind \
  --kclient codewind  \
  --kdevuser developer \
  --kdevpass passw0rd \
  --ingress "apps.mycluster.X.X.X.X.nip.io" --kurl https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io



INFO[0000] Checking namespace codewind-only exists
INFO[0000] Creating codewind-only namespace
INFO[0000] Using namespace : codewind-only
INFO[0000] Container images :
INFO[0000] eclipse/codewind-pfe-amd64:latest
INFO[0000] eclipse/codewind-performance-amd64:latest
INFO[0000] eclipse/codewind-keycloak-amd64:latest
INFO[0000] eclipse/codewind-gatekeeper-amd64:latest
INFO[0000] Running on openshift: true
INFO[0000] Using ingress domain: apps.mycluster.X.X.X.X.nip.io
INFO[0000] Creating service account definition 'codewind-k4137d9y'
INFO[0000] Waiting for Keycloak to start
.
INFO[0000] Configuring Keycloak...
INFO[0000] https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io
INFO[0001] Updating existing Keycloak realm 'codewind'
INFO[0001] Checking for Keycloak client 'codewind-k4137d9y'
INFO[0001] Creating Keycloak client
INFO[0001] Creating access role 'codewind-k4137d9y' in realm 'codewind'
INFO[0001] Creating Keycloak initial user
INFO[0001] Updating Keycloak user password
INFO[0002] Grant 'developer' access to this deployment
INFO[0002] Adding role 'codewind-k4137d9y' to user : '02f80936-d109-4133-8f51-2d75b4974454'
INFO[0002] Fetching client secret
INFO[0002] Checking if 'eclipse-codewind-x.x.dev' cluster access roles are installed
INFO[0002] Cluster roles 'eclipse-codewind-x.x.dev' already installed - updating
INFO[0002] Cluster roles 'eclipse-codewind-x.x.dev' updated complete
INFO[0002] Checking if 'codewind-rolebinding-k4137d9y' role bindings exist
INFO[0002] Adding 'codewind-rolebinding-k4137d9y' role binding
INFO[0002] Creating and setting Codewind PVC codewind-pfe-pvc-k4137d9y to 1Gi
INFO[0002] Deploying Codewind Service
INFO[0003] Waiting for pod: codewind-pfe-k4137d9y
INFO[0003] codewind-pfe-k4137d9y, phase: Pending Unschedulable
INFO[0013] codewind-pfe-k4137d9y, phase: Pending ContainersNotReady
INFO[0098] codewind-pfe-k4137d9y, phase: Running
INFO[0098] Deploying Codewind Performance Dashboard
INFO[0098] Waiting for pod: codewind-performance-k4137d9y
INFO[0098] codewind-performance-k4137d9y, phase: Pending
INFO[0098] codewind-performance-k4137d9y, phase: Pending ContainersNotReady
INFO[0125] codewind-performance-k4137d9y, phase: Running
INFO[0125] Preparing Codewind Gatekeeper resource
INFO[0125] Creating codewind-gatekeeper-k4137d9y.apps.mycluster.X.X.X.X.nip.io server Key
INFO[0125] Creating codewind-gatekeeper-k4137d9y.apps.mycluster.X.X.X.X.nip.io server certificate
INFO[0125] Deploying Codewind Gatekeeper Secrets
INFO[0125] Deploying Codewind Gatekeeper Session Secrets
INFO[0125] Deploying Codewind Gatekeeper TLS Secrets
INFO[0125] Deploying Codewind Gatekeeper Deployment
INFO[0125] Deploying Codewind Gatekeeper Service
INFO[0125] Deploying Codewind Gatekeeper Route
INFO[0125] Waiting for pod: codewind-gatekeeper-k4137d9y
INFO[0126] codewind-gatekeeper-k4137d9y, phase: Pending
INFO[0126] codewind-gatekeeper-k4137d9y, phase: Pending ContainersNotReady
INFO[0139] codewind-gatekeeper-k4137d9y, phase: Running
INFO[0139] Waiting for Codewind Gatekeeper to start on https://codewind-gatekeeper-k4137d9y.apps.mycluster.X.X.X.X.nip.io
..
INFO[0143] Waiting for Codewind PFE to start
.
INFO[0143] Codewind is available at: https://codewind-gatekeeper-k4137d9y.apps.mycluster.X.X.X.X.nip.io

```

Pieces to note from above :

1. The `--kurl` flag includes the URL of a Keycloak service previously installed
2. The Keycloak deployment is populated with :
    - A realm if the requested one does not exist
    - A user if the requested one does not exist
    - An access role for the deployment
    - Grant the user access to the deployment
3. Deploy Codewind PFE and Codewind Performance pods
4. Deploy Gatekeeper configured to use the Keycloak instance


## Gatekeeper Environment API

**GET: /api/v1/gatekeeper/environment**

An environment API is available in Gatekeeper to return connectivity details that clients such as the Codewind VSCode Plugin or Codewind Eclipse Plugins use when configuring remote connections lists. eg :

```json
{
  "auth_url": "https://codewind-keycloak-k412oms7.apps.mycluster.X.X.X.X.nip.io",
  "client_id": "codewind-k4137d9y",
  "workspace_id": "k4137d9y",
  "realm": "codewind",
  "codewind_version": "x.x.dev",
  "image_build_time": "20191211-081952"
}
```
