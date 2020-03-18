# UNDER DEVELOPMENT - Design could change #

# Linking Codewind Projects (Networking)

## Design decisions/assumptions
* Environment variables will be used to connect projects
* The URL of a project will never change (See issue #1) - for initial implementation
* The user will always specify a name for the enrvironment variable - for initial implementation but could be random later
* The users container/pod will restart when a link is created/updated
    * Either kicking of another build (which should complete quickly due to Docker caching) or implementing a restart and pickup env function
* envs will be added to the containers in different ways depending on the build tool/architecture
    * Local Docker through Codewind: env file
    * K8s through Codewind (Helm): configmap
    * Appsody local: env file (--docker-options)
    * Appsody K8s: env file or configmap (tbd)
    * odo K8s: tbd

### Issues that need to be addressed/fixed
1. In Remote the Ingress address for the Codewind templates changes on a project change

### Implementation notes:
* A project is type "LOCAL" if the POST request does not contain a parentPFEURL AND the target projectID is exists on the same PFE.
* A project type is "REMOTE" if a parentPFEURL is given, PFE cannot know whether the given URL is pointing to a Codewind application.

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
                projectURL: "internalURL", // Such as Docker network address
                envName: "JAVA_APP_URL",
                type: "LOCAL",
            },
            {
                projectId: "xxxxx-xxxx-xxxx-xxxxx",
                projectURL: "externalURL", // Such as Ingress address
                envName: "JAVA_APP_URL",
                parentPFEURL: "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io",
                type: "REMOTE",
            }
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
                "type": "LOCAL",
            },
            {
                "projectId": "xxxxx-xxxx-xxxx-xxxxx",
                "projectURL": "externalURL",
                "envName": "JAVA_APP_URL",
                "parentPFEURL": "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io",
                "type": "REMOTE",
            }
        ]
    }
```

```json
POST /api/v1/links

LOCAL : Don't send a parentPFEURL
BODY {
    "projectId": "xxxxx-xxxx-xxxx-xxxxx",
    "projectURL": "internalURL",
    "envName": "JAVA_APP_URL",
    "type": "LOCAL",
}

RES
202 : Added successfully
    : BODY null

40x : Target Project ID does not exist on the same PFE
    : BODY null


REMOTE :
BODY {
    "projectId": "xxxxx-xxxx-xxxx-xxxxx",
    "projectURL": "externalURL",
    "envName": "JAVA_APP_URL",
    "parentPFEURL": "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io",
    "type": "REMOTE",
}

RES
202 : Added successfully
    : BODY null
```

```json
PUT /api/v1/links/:env
BODY {
    "env": "JAVA_APP_URL",
    "url": "appbaseURL or host+port (aka what we use to talk to performance)",
}

RES
202 : Updated successfully
    : BODY null
```

```json
DELETE /api/v1/links/:env
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
cwctl project link get
    --id value,         -i value    the project id
    --conid value                   the connection id
```

```bash
cwctl project link create
    --id value,         -i value    the project id
    --conid value                   the connection id
    --targetID value,     -t value    the project id of the project to add a link to
    --targetConID value               the connection id of the project to add a link to
    --env value         -e value    the name of the environment variable to use
    --url value         -u value    the url of the project to add a link to
```

```bash
cwctl project link update
    --id value,         -i value    the project id
    --conid value                   the connection id
    --targetID value,     -t value    the project id of the project to update a link to
    --targetConID value               the connection id of the project to update a link to
    --env value         -e value    the name of the environment variable to use
    --url value         -u value    the url of the project to update a link to
```

```bash
cwctl project link delete
    --id value,         -i value    the project id
    --conid value                   the connection id
    --targetID value,     -t value    the project id of the linked project to delete
    --targetConID value               the connection id of the linked project to delete
```
