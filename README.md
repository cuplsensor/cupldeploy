# cupldeploy

Deploys a system made up of:

* A web application comprised of [cuplfrontend](https://github.com/cuplsensor/cuplfrontend) and [cuplbackend](https://github.com/cuplsensor/cuplbackend).
* [cuplTag](https://github.com/cuplsensor/cupltag) hardware and firmware.

## System Diagram 

![Diagram showing cuplfrontend cuplbackend and the database](docs/cupldeploy_system_diagram.png)

DNAME is short for DEPLOYMENT NAME `d3` or `latest`

### Frontend Diagram

The frontend web application is hosted at a domain, for example: [latest.f.cupl.uk](https://latest.f.cupl.uk). 

The domain is registered with [Amazon Route53](https://docs.aws.amazon.com/route53/?id=docs_gateway). 

An **A Record** routes traffic to a [Amazon CloudFront distribution](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-working-with.html). CloudFront is a [Content Delivery Network](https://en.wikipedia.org/wiki/Content_delivery_network), which stores copies of **files** from a given **origin** at edge locations worldwide. 

* The files are a production optimized build of the **cuplfrontend** [static web application](https://en.wikipedia.org/wiki/Static_web_page). 
* The origin is an [Amazon S3 Bucket](https://aws.amazon.com/s3/), a *pay-for-what-you-use* web folder with near infinite capacity.

CloudFront reduces latency in file access, compared to just hosting from S3. It is also easy to add an Amazon-provided SSL certificate, by [requesting one](https://aws.amazon.com/premiumsupport/knowledge-center/install-ssl-cloudfront/) in Amazon Certificate Manager. The frontend should be served over HTTPS in a production environment. 

The frontend web application provides a Graphical User Interface to **cuplbackend**. Data are stored and retrieved via calls to its web-based API.

### Backend Diagram

The backend web application is hosted at a domain name, for example: [latest.b.cupl.uk](https://latest.b.cupl.uk). 

The domain is registered with the DigitalOcean DNS. An **A record** routes traffic to the IP address of a DigitalOcean Droplet, which is a virtual machine running Linux. 

Services are packaged into [Docker](https://en.wikipedia.org/wiki/Docker_(software)) containers. This is done for isolation: one service is not able to interfere with another. Each *sees* its own Linux installation and has to install its own dependencies, thereby avoiding [dependency hell](https://en.wikipedia.org/wiki/Dependency_hell). 

The 3 services run on a single Docker instance using [Docker Compose](https://docs.docker.com/compose/).  Each is defined in [docker-compose.yml](docker-compose.yml):

1. [Nginx-certbot](https://hub.docker.com/r/staticfloat/nginx-certbot/) runs the web server, **Nginx**. This acts as a *reverse proxy*. It rewrites requests received over TCP/IP into the [WSGI](https://en.wikipedia.org/wiki/Web_Server_Gateway_Interface) protocol, which is standard for Python web applications.
2. The [cuplbackend](https://hub.docker.com/r/cupl/backend) web application. It is built atop of the [Flask](https://flask.palletsprojects.com/en/1.1.x/) framework. The application exposes [two HTTPS APIs](https://cupl.readthedocs.io/projects/backend/en/latest/docs/api/index.html). The interface is text only: data are read and written as [JSON](https://en.wikipedia.org/wiki/JSON). Data are persisted in an external PostgreSQL database.
3. A [Redis](https://hub.docker.com/_/redis) instance. A cuplbackend dependency named [Flask-Limiter](https://flask-limiter.readthedocs.io/en/stable/) uses this to record and block API requests. 

## GitHub Workflows

There are 3 workflows in [main.yml](.github/workflows/main.yml). These are run by a [GitHub Action](https://github.com/features/actions) whenever a commit is pushed to GitHub. The workflows build and run cuplfrontend / cuplbackend on the AWS / DigitalOcean infrastructure described above. 

![GitHub Workflows](docs/ghworkflows.png)

Application versions are specified by commit hashes of Git submodules in this parent repository. In this way, cupldeploy documents which submodule versions are compatible with one another.

![GitHub Submodules](docs/ghsubmodules.png)

### Frontend Workflow

### Backend Workflow

1. GitHub Actions launches a Ubuntu Linux [Hosted Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources).
2. The hosted runner configures an SSH connection to the Droplet.
3. The hosted runner uses SSH to create a [Docker context](https://docs.docker.com/engine/context/working-with-contexts/). When this is employed, docker commands run remotely on the Droplet instead of locally on the runner.
4. The command ``docker-compose --context remote pull`` causes 3 Docker images defined in [docker-compose.yml](docker-compose.yml) to be downloaded from DockerHub.
5. The command ``docker-compose --context remote up --build -d`` builds each Docker images into a containers. 
6. Finally ``up`` in the previous command causes all containers to run ``-d`` daemonised. 
 
Docker builds are configured from a list of environment variables. These are stored in this repository as encrypted GitHub Secrets.

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





