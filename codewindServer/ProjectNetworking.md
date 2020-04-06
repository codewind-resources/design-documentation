# UNDER DEVELOPMENT - Design could change #

# Linking Codewind Projects (Networking)

## Intended user experience
Access URL's pointing to Codewind projects using environment variables in their code:
```javascript
const databaseURL = process.env.DATABASE; // DATABASE points to a project under the covers
```

## Design decisions/assumptions
* Environment variables will be used to connect projects
* ~~The URL of a project will never change (See issue #1) - for initial implementation~~ Use of link proxy voids this
* The user will always specify a name for the enrvironment variable - for initial implementation but could be random later
* The users container/pod will restart when a link is created/updated
    * Either kicking of another build (which should complete quickly due to Docker caching) or implementing a restart and pickup env function
* envs will be added to the containers in different ways depending on the build tool/architecture
    * Local Docker through Codewind: env file
    * K8s through Codewind (Helm): configmap
    * Appsody local: env file (--docker-options)
    * Appsody K8s: env file or configmap (tbd)
    * odo K8s: tbd
* The URLs put into containers will point back to a proxy running in PFE so that PFE can direct the request to the application's current URL
    * For example `NEW_URL=http://codewind-pfe:9090/links/proxy/:idOfProjectToLinkTo`

### Issues that need to be addressed/fixed
1. ~~In Remote the Ingress address for the Codewind templates changes on a project change~~
2. PFEs need to be able to share ProjectLists (at least the external URLs of projects)

## PFE
### Project links object
```javascript
project: {
    id: xxxxx-xxxx-xxxx-xxxxx,
    name: javaApp,
    links: {
        filePath: "connectionFileLocation.env",
        _links: [
            {
                projectId: "xxxxx-xxxx-xxxx-xxxxx",
                projectURL: "http://codewind-pfe/links/proxy/xxxxx-xxxx-xxxx-xxxxx", // URL pointing back to the PFE link proxy with application id
                envName: "JAVA_APP_URL",
            },
        ]
    }
}
```

### API
Base url: `/api/v1/links`

```json
GET /api/v1/links
BODY null

RES
200 : Successful
    : BODY {
        "connectedProjects": [
            {
                "projectId": "xxxxx-xxxx-xxxx-xxxxx",
                "projectURL": "internalURL",
                "envName": "JAVA_APP_URL",
            },
        ]
    }
```

```json
POST /api/v1/links

LOCAL : Don't send a parentPFEURL or projectURL
BODY {
    "targetProjectID": "xxxxx-xxxx-xxxx-xxxxx",
    "envName": "JAVA_APP_URL"
}

RES
202 : Added successfully
    : BODY null

40x : Target Project ID does not exist on the same PFE
    : BODY null
```

```json
PUT /api/v1/links/
BODY {
    "envName": "OLD_ENV_NAME",
    "updatedEnvName": "NEW_ENV_NAME"
}

RES
202 : Updated successfully
    : BODY null
```

```json
DELETE /api/v1/links/
BODY null

RES
202 : Deleted successfully
    : BODY null
```

## cwctl

Command:
```bash
cwctl project link
```

```bash
cwctl project link list
    --id value,             -i value    the project id
```

```bash
cwctl project link create
    --id value,             -i value    the project id
    --targetID value,       -t value    the project id of the project to add a link to
    --env value             -e value    the name of the environment variable to use
```

```bash
cwctl project link rename
    --id value,         -i value    the project id
    --env value         -e value    the name of the environment variable to use
    --newEnv value                  the new name for the environment variable
```

```bash
cwctl project link delete
    --id value,         -i value    the project id
    --env value         -e value    the name of the environment variable to delete
```
