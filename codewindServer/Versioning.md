## How to report versioning back to user

Codewind now consists of several components each having their own binary.

1. cwctl
2. codewind-pfe
3. codewind-performance
4. codewind-keycloak
5. codewind-gateway

Each of these needs to correctly be able to report its version back to the user to help with debugging mis match of levels.

###### Steps for user of cwctl

cwctl will require the -version option as well as an optional connectionID if you want to get the versions of a remotely deployed codewind.

`cwctl -version <-connID value>` 

output

`cwctl version x.x.x`<br/>
`codewind-pfe-amd64 x.x.x.imageID`<br/>
`codewind-performance-amd64 x.x.x.imageID`<br/>
`codewind-keycloak-amd64 x.x.x.imageID`<br/>
`codewind-gatekeeper-amd64 x.x.x.imageID`<br/>

Example

`cwctl version 0.6.0`<br/>
`codewind-pfe-amd64 0.6.0.23bd1a58b015`<br/>
`codewind-performance-amd64 0.6.0.9d6e9b75db3d`<br/>
`codewind-keycloak-amd64 0.6.0.d234ada371ed`<br/>
`codewind-gatekeeper-amd64 0.6.0.da86e6ba6ca1`<br/>

### What if the versions don't match?

If cwctl detects that the version of itself is not at the same level as codewind-pfe, we should output a warning to the user and advise them to update.  This is critical as the cwctl may be attempting to invoke commands that are not available in old levels

## Deploying codewind with versions

To make things simpler when deploying, cwctl can be passed an option release tag when installing.  This tag will specfiy the version of codewind to install. If the tag is not supplied then the latest master images are installed.

`cwctl install <-tag tagname>`

We should prevent a back level version of cwctl installing a later version of codewind as this would not be a supported environment.

