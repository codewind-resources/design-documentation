# Codewind Server Design documentation

The Codewind server consists of two main elements.   These are the API frontend (portal), and the build and run engine (turbine) .



## Flow of project data

A users project data typically flows in one direction when using Codewind.

1. Project starts on disk,
2. Its edited by an IDE
3. that in turn communicates with the cwctl (cli).
4. cwctl sends files to the codewind server
5. where it then gets built and deployed in a container

![](Codewind%20Server%20Design%20documentation/image1.png)


## Design for adding a project to codewind 
For a project to be added (bind) to codewind, it must already exist
either locally on disk or, in the case of fully hosted, in a volume
associated with the IDE.

###### Steps for user of cwctl

cwctl nvoked to bind project to codewind

`cwctl project bind -p <project location on disk> -l <language> -t <type> -n <project name>`

response of Project ID and Status

`Project ID:/ 1db976c0-0475-11ea-a25d-c143077130ce/Status:/ 200 OK`

###### Internal flow  

All actions that require access to the local filesystem will done by
cwctl rather than a bind mount back from the codewind-pfe container.
This removes any issues with permissions and the need to have the local
workspace running as a container

1. cwctl calls `/api/v1/projects/bind/start`api in portal

2. `/api/v1/projects/bind/start` creates directory structure in
codewind-pfe volume under `codewind-workspace/<projectname>`

3. cwctl  makes one call per file  to `/api/v1/projects/:id/upload` for every file in the project

4.` /api/v1/projects/:id/upload` receives each file and then writes it to
`/codewind-workspace/cw-temp/<projectname>`

5. cwctl calls `/api/v1/projects/:id/bind/end`when all files copied

6. `/api/v1/projects/:id/bind/end` copies project from `codewind-workspace/cw-temp/<projectname>` to`codewind-workspace/<projectname>` and then notifies Turbine that a new project is ready to build and run

![](Codewind%20Server%20Design%20documentation/image2.png)


## How does a delta code change work
If a user makes a code change, upon saving that file, codewind should automatically build and run the application with that code change

The cli needs to be told when the local project is ready to be uploaded
to the codewind-pfe container. For example, by the file watcher daemon when it detects a code change

###### Steps for user of cwctl

cwctl called to sync changed files to codewind-pfe container

`cwctl project sync -p <absolute path to project> -I <projectID> -t <milliseconds since epoch of last sync call>`

###### Internal flow

1. cwctl calls `/api/v1/projects/:id/upload` once for every file that
has changed.

The list of files is worked out by comparing the last modification time of each file in the project against the passed in '-t' parameter. Any file with a newer time is uploaded. The file must be in the watch list.
Each file that is uploaded is written to `/codewind-workspace/cw-temp/<projectname>`

2. cwctl calls `/api/v1/projects/:id/upload/end`

3. If a build is not in progress, changed files get copied to the build/run container created by turbine. A notification of changed files is also sent to turbine which will trigger a rebuild. If a build is already in progress, the copy to build container will wait until the build finishes

![](Codewind%20Server%20Design%20documentation/image3.png)

## Upgrading to 0.6.0
For 0.6.0, the restriction  for having to have all user projects in a `codewind-workspace`has been removed.  This means that projects created before 0.6.0 will need to be upgraded to the latest version

###### Steps for user of cwctl

cwctl called to upgrade projects that were created in a previous
release

`cwctl upgrade --workspace <old codewind-workspace dir>`

###### Internal flow

1. cwctl will parse the passed in workspace looking for the project.inf
files associated with a project

2. For each project.inf found, cwctl will bind that project to codewind
(see Adding a project to codewind)

## User want to remove a project from codewind
A user need to be able to remove a project from codewind if they no longer want to use it

###### Steps for user of cwctl

Currently, cwctl is not required to remove a project

###### Steps for users of api

`/api/v1/projects/:id/unbind` needs to be called

###### Internal flow

Projects are not actually delete by codewind. The underlying source code
will still exist in the workspace.

Calling `/api/v1/projects/:id/unbind`will
	* stop all the logs streaming
	* inform turbine to delete the project
	* remove project from `/codewind-workspace/<projectname>`

