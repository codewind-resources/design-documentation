# The Codewind Operator

The Codewind-Operator is an evolution of Codewind remote deployment and attempts to remove some of the pain points and scalability barriers associated with deploying Codewind into Kubernetes for just one or multiple users.

## The problem

Previously setting up Codewind required logging into Kubernetes or Openshift and performing intricate steps to determine the cluster configuration before using the Codewind-CLI (cwctl).  The CWCTL command also had many subcommands and flags that were needed to install or remove Codewind or Keycloak.

![](media/operator/cwctl-diagram1.png)

Things get complicated to administer when adding additional Codewinds for users especially when they need to share a common Keycloak. The administrator needed to know the connectivity details for Keycloak or discover them using either the Openshift console or run kubectl commands with selectors before deploying each new Codewind.

![](media/operator/cwctl-diagram2.png)

Over time administration could gets unmanageble, the administrator carries the burden of deploying new Codewinds for users, connecting them up to Keycloak and providing the user with connectivity details. 

![](media/operator/cwctl-diagram3.png)


With the Codewind-Operator, this piece of software handles the deployment and management of new PFE, Gatekeeper, Keycloak and Performance containers on behalf of a user. It has knowledge of all the Kubernetes resources that are part of a typical Codewind remote deployment, configures them and deploy the components on demand.

Deployment of the Codewind-Operator is performed using `oc` or `kubectl` with either a standard yaml or using Codewind CLI `cwctl`.  This step will require the human operator to have already logged into their Kubernetes or Openshift but that is only for the purpose of installing the operator.

![](media/operator/cwctl-diagram4.png)

Once installed, the Codewind-Operator runs in a pod on the cluster and can monitor the entire cluster or just a single namespace. The Codewind-Operator has the ability to deploy all the necessary Kubernetes resources to support Codewind including Pods, Secrets, PVC, Roles and Role Bindings and it can configure all of them including adding any security Realms, User Accounts (if they dont already exist), Access Roles, and exchange of any public keys.

## How it works

Codewind-Operator monitors running instances and compares them against requested workload. By monitoring what has been requested compared to what is currently running on the cluster, the Codewind-Operator has an ability to provision new Codewinds or remove Codewinds or launch support pods like "Codewind-Performance" on demand.

To achieve this, the Codewind-Operator adds two new custom resources `Codewind` and `Keycloak`

By creating a custom resource in Kubernetes with a piece of yaml code such as:

```
apiVersion: codewind-operator.eclipse.org/v1alpha1
kind: Keycloak
metadata:
  name: devex
spec:
  size: 1
```

This YAML request creation of a new `KIND` of resource called `Keycloak` with the name `devex`. Thats it, the Codewind-Operator sees the request for the new Keycloak customer resouce, begins downloading the Docker image, creates the PVC, secrets and deploys a new Keycloak POD.

Follow-up commands such as:

`kubectl get pods` will show the new Keycloak pod as will `kubectl get keycloaks` which is a new command that understands the new `keycloaks` type.

## Deploying a new remote Codewind deployment

Similar to deploying a Keycloak service, deploying Codewind follows the same pattern:

```
apiVersion: codewind-operator.eclipse.org/v1alpha1
kind: Codewind
metadata:
  name: testuser1
spec:
  auth: devex
```

The command `kubectl get codewinds` should display the list of running Codewind deployments:

```
NAME        AGE   AUTH    URL
testuser1   31m   devex   https://codewind-k5886ir3.mycluster-757441.eu-gb.containers.mydomain.cloud
testuser2   22m   devex   https://codewind-k5856a25.mycluster-757441.eu-gb.containers.mydomain.cloud
testuser3   7s    devex   https://codewind-k18364r5.mycluster-757441.eu-gb.containers.mydomain.cloud
```

Running the command `kubectl get keycloaks` will show the Keycloak details`.

```
NAME        AGE   URL
devex       42m   https://codewind-keycloak-k18465r6.mycluster-757441.eu-gb.containers.mydomain.cloud
```

