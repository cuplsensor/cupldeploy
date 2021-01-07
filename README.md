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

# Create a droplet
Follow instructions https://danielwachtel.com/devops/deploying-multiple-dockerized-apps-digitalocean-docker-compose-contexts

## Add a domain name to the droplet
Set the Github secret CUPL_DEPLOY_LATEST_HOST to this name.

## Log in as root via SSH
```ssh root@$CUPL_DEPLOY_LATEST_HOST```

## Add a non-root user
e.g. deployer. Set the Github secret CUPL_DEPLOY_LATEST_USER to this name.

## Create an SSH key pair for the non-root user
There should be no pass phrase. Copy the entire contents of the private key into a Github secret named CUPL_DEPLOY_LATEST_KEY. 

Copy the public key to the droplet with: 

```ssh-copy-id -i ~/.ssh/deployerkey $CUPL_DEPLOY_LATEST_USER@$CUPL_DEPLOY_LATEST_HOST```





