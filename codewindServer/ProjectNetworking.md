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

## PFE
### Project links object
```javascript
project: {
    id: xxxxx-xxxx-xxxx-xxxxx,
    name: javaApp,
    links: [
        {
            id: "connectionID/projectID", // See GET request for more information
            projectId: "xxxxx-xxxx-xxxx-xxxxx",
            projectName: "javaApp",
            env: "JAVA_APP_URL",
            url: "urlToProject.com",
            connectionID: "K60NGBW9",
            connectionURL: "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io"
        },
        {   ...connectedProject2    },
        {   ...connectedProject3    },
    ]
}
```

### API
Base url: `/api/v1/links`

```json
GET /api/v1/links // Gets all the linked projects
BODY null

RES
200 : Successful
    : BODY {
        "connectedProjects": [
            {
                "id": "connectionID/projectID", // e.g. K60NGBW9/xxxxx-xxxx-xxxx-xxxxx in this form as a projectID is only unique per connection
                "projectId": "xxxxx-xxxx-xxxx-xxxxx",
                "projectName": "javaApp",
                "env": "JAVA_APP_URL", // The env that will be put in the container
                "url": "urlToProject.com", // The value for the env
                "connectionID": "K60NGBW9",
                "connectionURL": "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io"
            },
            {   ...connectedProject2    },
            {   ...connectedProject3    },
        ]
    }
```

```json
POST /api/v1/links // Adds a link
BODY {
    "projectId": "xxxxx-xxxx-xxxx-xxxxx",
    "projectName": "javaApp",
    "env": "JAVA_APP_URL", // The env that will be put in the container
    "url": "appbaseURL or host+port (aka what we use to talk to performance)", // The value for the env
    "connectionID": "K60NGBW9",
    "connectionURL": "https://codewind-gatekeeper-k56r8d8f.apps.exact-mongrel-icp-mst.9.20.195.90.nip.io"
}

RES
202 : Added successfully
    : BODY null
```

```json
PUT /api/v1/links/:id // Updates the env and value of a link
BODY {
    "env": "JAVA_APP_URL", // The env that will be put in the container
    "url": "appbaseURL or host+port (aka what we use to talk to performance)", // The value for the env
}

RES
202 : Updated successfully
    : BODY null
```

```json
DELETE /api/v1/links/:id // Deletes a link
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
