# Reporting Codewind versions

Codewind now consists of several components each having their own binary.

1. cwctl
2. codewind-pfe
3. codewind-performance
4. codewind-keycloak
5. codewind-gatekeeper

Each of these needs to correctly be able to report its version back to the user to help with debugging mismatch of levels.

## Version command overview
```bash
NAME:
   main version - Get versions of deployed Codewind containers

USAGE:
   main version [command options] [arguments...]

OPTIONS:
   --conid value  The connection ID (default: "local")
   --all, -a      Get the codewind container versions for all connections
```
Note: 
* If the Codewind deployment has a self-signed certificate then the global `--insecure` flag will need to be used.
* If `--all` is used `--conid` will be ignored.

## Steps for cwctl user

Cwctl will require the `version` option as well as an optional `connectionID` if you want to get the versions of a deployed Codewind.

### Local Codewind deployment

```bash
cwctl version
```

Example output

```bash
> cwctl version
CWCTL VERSION: x.x.dev

CONNECTION ID 	PFE VERSION		        PERFORMANCE VERSION		   GATEKEEPER VERSION
local		    latest-20200130-164353	x.x.dev-20200130-164420

> cwctl --json version
{
    "CwctlVersion": "0.6.0"
    "PFEVersion": "0.6.0-20191203-132736"
    "PerformanceVersion": "0.6.0-20191203-132736"
    "GatekeeperVersion": "0.6.0-20191203-132736"
}
```



### Remote Codewind deployment
```bash
cwctl version --conid 
```

Example output

```bash
> cwctl --insecure version --conid XXXXXX
CWCTL VERSION: x.x.dev

CONNECTION ID 	PFE VERSION			        PERFORMANCE VERSION		    GATEKEEPER VERSION
k60ngbw9	    x.x.dev-20200108-171055		x.x.dev-20200110-162816		x.x.dev-20200108-171623

> cwctl --insecure --json version --conid XXXXXX
{
    "CwctlVersion": "x.x.x"
    "PFEVersion": "x.x.x.imageBuildTime"
    "PerformanceVersion": "x.x.x.imageBuildTime"
    "GatekeeperVersion": "x.x.x.imageBuildTime"
}
```

### Using the --all option
```bash
cwctl version --all
```

Example output

```bash
> cwctl --insecure version --all

CONNECTION ID 	PFE VERSION			            PERFORMANCE VERSION		    GATEKEEPER VERSION
K60NGBW9	    x.x.dev-20200108-171055		    x.x.dev-20200110-162816		x.x.dev-20200108-171623
K60ZDVIK	    x.x.dev-20200130-160716		    x.x.dev-20200130-161323		x.x.dev-20200109-205659
local		    latest-20200130-164353		    x.x.dev-20200130-164420

> cwctl --insecure --json version --all
{"cwctlVersion":"x.x.dev","connections":{"K60NGBW9":{"performanceVersion":"x.x.dev-20200110-162816","gatekeeperVersion":"x.x.dev-20200108-171623","PFEVersion":"x.x.dev-20200108-171055"},"K60ZDVIK":{"performanceVersion":"x.x.dev-20200130-161323","gatekeeperVersion":"x.x.dev-20200109-205659","PFEVersion":"x.x.dev-20200130-160716"},"local":{"performanceVersion":"x.x.dev-20200130-164420","PFEVersion":"latest-20200130-164353"}},"errors":{}}
```

## Deploying Codewind with versions

Cwctl can be passed an option release tag when installing.  This tag will specify the version of Codewind to install. If the tag is not supplied then the latest master images are installed.

`cwctl install <-tag tagname>`

Cwctl prevents a back level version of cwctl installing a later version of Codewind as this would not be a supported environment.
