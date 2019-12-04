# Reporting Codewind versions

Codewind now consists of several components each having their own binary.

1. cwctl
2. codewind-pfe
3. codewind-performance
4. codewind-keycloak
5. codewind-gatekeeper

Each of these needs to correctly be able to report its version back to the user to help with debugging mismatch of levels.

## Steps for cwctl user

Cwctl will require the `version` option as well as an optional `connectionID` if you want to get the versions of a remotely deployed Codewind.

Command

```bash
cwctl version --insecure --connID <value>
```

Output

```bash
{
    "CwctlVersion": "x.x.x"
    "PFEVersion": "x.x.x.imageBuildTime"
    "PerformanceVersion": "x.x.x.imageBuildTime"
    "GatekeeperVersion": "x.x.x.imageBuildTime"
}
```

Example

```bash
{
    "CwctlVersion": "0.6.0"
    "PFEVersion": "0.6.0-20191203-132736"
    "PerformanceVersion": "0.6.0-20191203-132736"
    "GatekeeperVersion": "0.6.0-20191203-132736"
}
```

### If versions don't match

If cwctl detects that the version of itself is not at the same level as Codewind PFE, we should output a warning to the user and advise them to update.  This is critical as the cwctl may be attempting to invoke commands that are not available in old levels.

## Deploying Codewind with versions

To make things simpler when deploying, cwctl can be passed an option release tag when installing.  This tag will specify the version of Codewind to install. If the tag is not supplied then the latest master images are installed.

`cwctl install <-tag tagname>`

We should prevent a back level version of cwctl installing a later version of Codewind as this would not be a supported environment.
