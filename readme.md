```
Example usage:
build.py --gen-pw

build.py --backend-db    "use"    --keycloak-db "a" \
         --root-db       "secure" --keycloak-pw "password" \
         --client-secret "please"
```

> Requires python 3.6 or later (because of template format strings)

> The script works on macOS and Linux. On windows the script executes correctly but the docker containers cannot be run properly because docker on windows doesn't properly support all network features that are required.

> Depending on the version of docker-compose or the operating system you are running you might need to comment out the last line in the docker-compose file. Sometimes setting the gateway on an ipam network is not possible.

If you want to use this repository by cloning it make sure to also clone the submodules alongside with it:

```
git clone https://git.jannikwibker.dev/PSE/deployment --recursive
```

# Overview

The build script will do all the work for you, it will:
- build all docker containers
- generate passwords
- set the keycloak client secret

After that all docker containers are ready to use with docker-compose.

To start the containers go to the smart_tv_system directory and run `docker-compose up -d`.

Default config files are provided.

The config files used by the software are located in ``./backend/build/resources/config``.

# Usage

```
./build.py -g
```

This will run the script and tell it to generate passwords.

If you don't want to use pre-generated passwords you can use the following flags:

```
./build.py
--backend-db "<password>" \
--keycloak-db "<password>" \
--root-db "<password>" \
--keycloak-pw "<password" \
--client-secret "<client secret (uuid)>"
```

After that you'll need to create a keycloak user.
To do this go to http://localhost:8081, use "admin" as the username and look for "keycloak_password" in the script output for the password.

> The passwords can also be found in text files in the smart_tv_system directory as these are the files being used for docker secrets.

After logging in head over to "Users" -> "Add user" (top right). Type in a username, hit "Save", go to the user, go to "Credentials", enter a password, uncheck "Temporary" and hit "Reset password".

The user still needs the appropriate roles, head over to "Role Mapping" and add "app_admin" or "app_editor" as a role.

Now you're all set and can use the system however you like.

The frontend is available at: http://localhost:3000<br />
The dashboard is avaialable at: http://localhost:3001<br />
The backend is available at: http://localhost:8080 (normally not ever needed)<br />
The database is available at: http://localhost:5432 (normally not ever needed)<br />
The IDP is available at: http://localhost:8081

> There is an nginx container which forwards requests to the individual docker containers (or staticly built versions of the frontend and dashboard)


# Potential problems

Sometimes building the keycloak container doesn't go to well because keycloak fails to initialize some database related things.

In the case that after running `docker-compose up -d` only 2 containers are running a short while afterwards this has most likely happened.

If this happens recreating the containers will most likely fix the issue, but for this to work correctly you have to clear docker images so that docker cannot use its cache (docker ignores the --no-cache flag for some things):

```
docker-compose down
docker image rm localbackend:0.0.1
docker image rm localnginx:0.0.1
docker image rm localkeycloak:0.0.1
docker image rm postgresql:0.0.1
```

Alternatively `docker image prune` should also work.

After that just rerun the script and start up the docker containers again.
