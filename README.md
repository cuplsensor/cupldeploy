# cupldeploy

A simple way to deploy a system made up of:

* A web application (cuplfrontend and cuplbackend).
* The cuplTag hardware and firmware.

This is done with docker.

# Clone the repository

`git clone --recursive https://github.com/cuplsensor/cuplbackend`

# Set environment variables

```
chmod +x autogenpwd.sh
export ADMINAPI_CLIENTSECRET=$(./autogenpwd.sh)
export TAGTOKEN_CLIENTSECRET=$(./autogenpwd.sh)
export DB_PASS=$(./autogenpwd.sh)
export HASHIDS_SALT=$(./autogenpwd.sh)
export CSRF_SESSION_KEY=$(./autogenpwd.sh)
export SECRET_KEY=$(./autogenpwd.sh)
docker-compose config
```
