# Codewind Push Registries

## Current Design

When building a project on a remote deployment, Codewind requires a registry to push its built images to.
Currently, with a remote connection, we offer the user an "Image Registry Manager" through their chosen IDE.

Behind the scenes, this is calling the `/api/v1/imagepushregistry` and `api/v1/registrysecrets` APIs on that deployed PFE.
The details of the registry are persisted inside a `settings.json`, which exists in the filesystem of the PFE container.

It is therefore required of a user to supply the details of a valid repository, before using Codewind remote.
Additionally, if using a public store (such as a public namespace on docker.io), these images will be publicly available.

## Codewind Registry Proposal

It is proposed to add a default registry into the cluster Codewind is deployed to.

This default would use the standard registry [image](https://hub.docker.com/_/registry) provided by Docker. It would be deployed into its own namespace when `cwctl remote install` is called.
We could therefore add a flag to not install the registry.
A user would still be able to select their chosen registry, by posting to `/api/v1/imagepushregistry` on the deployed PFE.
