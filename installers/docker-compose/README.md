# Overview

This directory contains a Docker Compose specification that can be used to spin up an instance of Legend Studio and Legend Query.

# Assumptions

## Localhost

This installer assumes that localhost resolves to where the various Legend JVMs are running and localhost can be reached from a browser. 

If this is not the case, for e.g, Legend containers are being run on a machine accessible only by an IP or other names, all references to localhost, both in the .env and Gitlab OAuth configuration has to be changed.

## Gitlab.com

Out of the box, Legend Studio uses Gitlab for model version control. The installer uses the public gitlab.com instance.

If you do not wish to use gitlab.com, you can use any Gitlab instance. Make sure to change all the GITHUB variables in .env to point to your Gitlab instance.

# Usage Instructions

## Create a Gitlab.com account

Legend uses Gitlab as the identity provider. Create a user account at https://gitlab.com

## Create a Gitlab OAuth application

Create an OAuth application as described here https://docs.gitlab.com/ee/integration/oauth_provider.html

The OAuth application should be configured as follows :

- Redirect URI:

```
http://localhost:6300/callback
http://localhost:6100/api/auth/callback
http://localhost:6100/api/pac4j/login/callback
http://localhost:6201/depot-store/callback
http://localhost:6200/depot/callback
http://localhost:9000/studio/log.in/callback
http://localhost:9001/query/log.in/callback
```

- Enable the "Confidential" check box
- Enable these scopes: openid, profile, api

Save the application and record the application id and secret.

## Set the app id and secret of your Gitlab application

```
export GITLAB_APP_ID=<add your app id>
export GITLAB_APP_SECRET=<add your app secret>
```

## Studio 

Start Studio as follows. 

```
./docker-compose.sh --profile studio up -d
```

After a few minutes, the containers should pass their health checks and be marked as healthy.

```

./docker-compose.sh ps
       Name                     Command                  State                Ports
---------------------------------------------------------------------------------------------
legend-engine        /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:6300->6300/tcp
legend-mongodb       docker-entrypoint.sh --auth      Up             0.0.0.0:27017->27017/tcp
legend-sdlc          /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:6100->6100/tcp
legend-studio        /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:9000->9000/tcp
setup                /setup/setup.sh                  Exit 0```
```

### Use Studio

Once all containers are running, you should be able to access Studio at `http://localhost:9000/studio`

When accessing Studio for the first time, you will see a URL redirect, redirecting you to Gitlab.com to authorize the Legend OAuth application.

Once authorized, you should be able to start using Studio.


### Open a sample project 

In the opening page of Studio, select the "Legend Installer Demo" project or navigate directly to the project using this link http://localhost:9000/studio/setup/40061958

## Query

Run Query as follows :
```
./docker-compose.sh --profile query up -d
```

After a few minutes, the containers should pass their health checks and be marked as healthy.

```
./docker-compose.sh ps
       Name                     Command                  State                Ports
---------------------------------------------------------------------------------------------
legend-depot         /app/entrypoint.sh               Up (healthy)   0.0.0.0:6200->6200/tcp
legend-depot-store   /app/entrypoint.sh               Up (healthy)   0.0.0.0:6201->6201/tcp
legend-engine        /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:6300->6300/tcp
legend-mongodb       docker-entrypoint.sh --auth      Up             0.0.0.0:27017->27017/tcp
legend-query         /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:9001->9001/tcp
legend-sdlc          /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:6100->6100/tcp
legend-studio        /bin/sh -c java -cp /app/b ...   Up (healthy)   0.0.0.0:9000->9000/tcp
setup                /setup/setup.sh                  Exit 0

```

### Use Query

Once all containers are running, you should be able to access Studio at `http://localhost:9001/query`

When accessing Studio for the first time, you will see a URL redirect, redirecting you to Gitlab.com to authorize the Legend OAuth application.

Once authorized, you should be able to start using Query.

### Index Projects

Query allows you to access projects that have been indexed by the depot store server.

We have automatically indexed a project into the depot store. If this project is not visible in query, execute the following command.

```
curl -v -X GET "http://localhost:6201/depot-store/api/queue/PROD-1234/org.finos.legend.demo/legend-query/1.0.2?maxRetries=5" -H "accept: text/plain"
```

# Swagger 

Each component exposes a Swagger endpoint that can be used to explore the component's API.

| Component | Endpoint |
| ------ | ---------|
| Engine | http://localhost:6300/api/swagger# |
| SDLC Server | http://localhost:6100/api/swagger# |
| Depot Store Server | http://localhost:6201/depot-store/api/swagger# | 
| Depot Server | http://localhost:6200/depot/api/swagger# |

# Known Issues / Gotchas

In some cases, navigating to the Studio/Query web page can return an "unauthorized" error. This is usually because of stale cookies. Clear browser cookies for localhost (or domain name/IP that you are using) and try again.

#


# Setup for Windows - WSL

## First time setup

1. Make sure Docker Desktop is installed. If not, install it from here: https://www.docker.com/products/docker-desktop/
2. Run this command to check if current windows user is in docker-users group `net localgroup docker-users`. If not present, add the user by running this command in an elevated command prompt `net localgroup docker-users <<windows-user>> /ADD`. Post that, sign out and sign back in.
3. Ensure Ubuntu wsl environment exists. If not, install it using the command `wsl --install -d Ubuntu`
4. Start Docker Desktop application and keep it running in the background
5. In wsl terminal, check if docker executable is working by running the commad `docker --version`
6. Create Gitlab OAuth application using [above](https://github.com/gs-ssh16/legend/blob/master/installers/docker-compose/README.md#create-a-gitlabcom-account) documentation and note down client_id and secret
7. In wsl terminal, run below commands: <br/>
          `cd ~` <br/>
          `git clone https://github.com/gs-ssh16/legend` <br/>
          `cd ./legend/installers/docker-compose` <br/>
          `sudo chown -R $(whoami):$(whoami) *`

## Running legend applications
1. Start Docker Desktop application and keep it running in the background
2. In wsl terminal, run below commands: <br/>
          `cd ~/legend/installers/docker-compose` <br/>
          `export GITLAB_APP_ID=<<add your app id>>` <br/>
          `export GITLAB_APP_SECRET=<<add your app secret>>` <br/>
          `./docker-compose.sh --profile studio up -d`
4. It takes few minutes to start up all servers
5. Status can be checked using this command `./docker-compose.sh ps`
6. Once all applications are running, you should be able to access Studio at http://localhost:9000/studio
7. Applications can be terminated with this command `./docker-compose.sh --profile studio down`


## Query Application

### One time setup on Gitlab project
1. Add [settings.xml](https://gitlab.com/finosfoundation/legend/showcase/legend-showcase-northwind-data/-/blob/master/settings.xml?ref_type=heads) file to the project
2. Add [deploy_snapshot and deploy_release](https://gitlab.com/finosfoundation/legend/showcase/legend-showcase-northwind-data/-/blob/master/.gitlab-ci.yml?ref_type=heads#L49-64) jobs to .gitlab-ci.yml file of the project

### One time setup on deployment side
1. [Create](https://gitlab.com/-/profile/personal_access_tokens) a Gitlab personal access token with read_api scope
2. Add a new repository and server in respective sections in docker-compose/depot-store/config/settings.xml file like below, with <<GITLAB_PROJECT_ID>> and <<PERSONAL_ACCESS_TOKEN>> replaced
   ```
   <repository>
      <id><<GITLAB_PROJECT_ID>></id>
      <url>https://gitlab.com/api/v4/projects/<<GITLAB_PROJECT_ID>>/-/packages/maven</url>
      <releases>
          <enabled>true</enabled>
          <updatePolicy>never</updatePolicy>
      </releases>
      <snapshots>
          <enabled>true</enabled>
          <updatePolicy>always</updatePolicy>
      </snapshots>
   </repository>
   ```
   ```
   <server>
     <id><<GITLAB_PROJECT_ID>></id>
     <username>Private-Token</username>
     <password><<PERSONAL_ACCESS_TOKEN>></password>
   </server>
   ```

### Running Query Applications
1. There is a docker-compose profile `query` similar to `studio` profile. To start it, run below commands in a wsl terminal: <br/>
   `cd ~/legend/installers/docker-compose` <br/>
   `export GITLAB_APP_ID=<<add your app id>>` <br/>
   `export GITLAB_APP_SECRET=<<add your app secret>>` <br/>
   `export DEPOT_STORE_ADMIN_USER=<<add your gitlab user handle>>` <br/>
   `./docker-compose.sh --profile query up -d`
2. Once all applications are running, you should be able to access Query at http://localhost:9001/query
3. In order to see latest updates to master (or) new versions of projects in Query, Depot server needs to be notified once the Gitlab pipeline completes. To notify, hit below endpoints <br/>
   Notify master-SNAPSHOT -> `http://localhost:6201/depot-store/api/queue/PROD-<<GITLAB_PROJECT_ID>>/<<GROUP_ID>>/<<ARTIFACT_ID>>/master-SNAPSHOT`  <br/>
   Notify version -> `http://localhost:6201/depot-store/api/queue/PROD-<<GITLAB_PROJECT_ID>>/<<GROUP_ID>>/<<ARTIFACT_ID>>/<<version>>`
4. To check available versions of a project on Depot server, hit this endpoint - `http://localhost:6200/depot/api/projects/<<GROUP_ID>>/<<ARTIFACT_ID>>/versions?snapshots=true`
