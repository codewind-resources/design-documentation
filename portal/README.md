Flow of project data
====================

1.  Project starts on disk,

2.  Its edited by an IDE

3.  that in turn communicates with the cwctl (cli).

4.  cwctl sends files to the codewind server

5.  where it then gets built and deployed in a container

![A close up of a sign Description automatically
generated](./media/image1.tiff)
Adding a project to codewind
============================

For a project to be added (bind) to codewind, it must already exist
either locally on disk or, in the case of fully hosted, in a volume
associated with the

Steps for user of cwctl

1.  *cwctl* invoked to bind project to codewind

    a.  *cwctl project bind -p \<project location on disk\> -l
        \<language\> -t \<type\> -n \<project name\>*

    b.  *response of Project ID and Status*

> *Project ID:* 1db976c0-0475-11ea-a25d-c143077130ce
>
> *Status:* 200 OK

Internal flow

All actions that require access to the local filesystem will done by
cwctl rather than a bind mount back from the codewind-pfe container.
This removes any issues with permissions and the need to have the local
workspace running as a container

1.  cwctl calls '/api/v1/projects/bind/start' api in pfe

2.  '/api/v1/projects/bind/start' creates directory structure in
    codewind-pfe volume under codewind-workspace/\<name\>

3.  cwctl calls '/api/v1/projects/:id/upload for every file in the
    project

4.  '/api/v1/projects/:id/upload' receives each file and writes it to
    /codewind-workspace/cw-temp/\<projectname\>

5.  cwctl calls '/api/v1/projects/:id/bind/end when all files copied

6.  '/api/v1/projects/:id/bind/end' copies project from cw-temp to
    codewind-workspace and then notifies Turbine that a new project is
    ready to build and run

    ![A screenshot of a cell phone Description automatically
    generated](./media/image2.tiff){width="6.384259623797026in"
    height="2.172222222222222in"}

Making a code change
====================

Steps for user of cwctl

1.  *cwctl* called to sync changed files to codewind-pfe container

    a.  *cwctl project sync -p \<absolute path to project\> -I \<project
        ID\> -t \<milliseconds since epoch of last sync call\>*

Internal flow

The cli needs to be told when the local project is ready to be uploaded
to the codewind-pfe container. For example, by the filewatcher when it
detects a code change

1.  cwctl calls '/api/v1/projects/:id/upload' once for every file that
    has changed.

    a.  The list of files is worked out by comparing the last
        modification time of each file in the project against the passed
        in '-t' parameter. Any file with a newer time is uploaded. The
        file must be in the watch list.

    b.  Each file that is uploaded is written to
        /codewind-workspace/cw-temp/\<projectname\>

2.  cwctl calls '/api/v1/projects/:id/upload/end'.

3.  If a build is not in progress, changed files get copied to the
    build/run container created by turbine. A notification of changed
    files is also sent to turbine which will trigger a rebuild. If a
    build is already in progress, the copy to build container will wait
    until the build finishes

![A screenshot of a cell phone Description automatically
generated](./media/image3.tiff){width="6.263888888888889in"
height="4.795833333333333in"}

Upgrading to 0.6.0
==================

Steps for user of cwctl

1.  *cwctl* called to upgrade projects that were created in a previous
    release

    a.  *cwctl upgrade \--workspace \<old codewind-workspace dir\>*

Internal flow

Before 0.6.0, projects had to be created in a fixed location on the
users disk called codewind-workspace. 0.6.0 allows the user to have the
project created anywhere and also removes the need for it to be in a
container.

1.  cwctl will parse the passed in workspace looking for the project.inf
    files associated with a project

2.  For each project.inf found, cwctl will bind that project to codewind
    (see Adding a project to codewind)

Removing a project from codewind
================================

Steps for user of cwctl

1.  *Currently, cwctl is not required to remove a project*

Steps for users of api

1.  '/api/v1/projects/:id/unbind' needs to be called

Internal flow

Projects are not actually delete by codewind. The underlying source code
will still exist in the workspace.

1.  '/api/v1/projects/:id/unbind' will

    a.  stop all the logs streaming

    b.  inform turbine to delete the project

    c.  remove project from /codewind-workspace/\<project\>

Security in codewind
====================

Below is a high level view of the major parts of codewind when running
with a remote installation of codewind-pfe. This would allow a user to
develop an application on their desktop (with source code locally) but
build and run it on a remote cloud. When running, the gatekeeper service
will provide the ingress endpoint that will be used for all
communication to pfe.

![A screenshot of a cell phone Description automatically
generated](./media/image4.tiff){width="6.263888888888889in"
height="4.295833333333333in"}

Installing a remote code-pfe
============================

Steps for user of cwctl

1.  *cwctl* called to install pfe on a remote cloud installation

    a.  *cwctl --insecure install remote \\*

> *\--namespace {your\_namespace} \\*
>
> *\--kadminuser admin\\*
>
> *\--kadminpass \<keycloakPassword\> \\*
>
> *\--krealm codewind \\*
>
> *\--kclient codewind \\*
>
> *\--kdevuser developer \\*
>
> *\--kdevpass \<userPassword\>*

Internal flow

The cli has been design to do almost all the steps required to deploy
codewind remotely by chaining together a number of Kubernetes api
commands. This cli will perform the following steps :

1.  Create codewind-keycloak server key

2.  Create codewind-keycloak server certificate

3.  Deploy Codewind Keycloak secrets

4.  Deploy Codewind Keycloak TLS secrets

5.  Deploy Codewind Keycloak Ingress

6.  Create Keycloak realm

7.  Create Keycloak client

8.  Create Keycloak initial user

9.  Update Keycloak user password

10. Deploy Codewind service

11. Deploy Codewind Performance Dashboard

12. Deploy Codewind Gatekeeper resources

13. Create codewind-gatekeeper server key

14. Create codewind-gatekeeper server certificate

15. Deploy Codewind Gatekeeper secrets

16. Deploy Codewind Gatekeeper session secrets

17. Deploy Codewind Gatekeeper secrets

18. Deploy Codewind Gatekeeper session secrets

19. Deploy Codewind Gatekeeper TLS secrets

20. Deploy Codewind Gatekeeper deployment

21. Deploy Codewind Gatekeeper service

22. Deploy Codewind Gatekeeper Ingress
