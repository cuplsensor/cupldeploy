## GitHub Workflows

There are 3 workflows in [main.yml](.github/workflows/main.yml). These are run by a [GitHub Action](https://github.com/features/actions) whenever a commit is pushed to GitHub. The workflows build and run cuplfrontend / cuplbackend on the AWS / DigitalOcean infrastructure described above. 

![GitHub Workflows](/docs/ghworkflows.png)

Application versions are specified by commit hashes of Git submodules in this parent repository. In this way, cupldeploy documents which submodule versions are compatible with one another.

![GitHub Submodules](/docs/ghsubmodules.png)

### Frontend Workflow

1. GitHub Actions launches a Linux [Hosted Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources).
2. The hosted runner checks out this repository (cupldeploy) including its submodules (cuplfrontend).

3. NPM is Node Package Manager. ``npm install`` installs Node.js packages that cuplfrontend requires. Among these are [React JS](https://reactjs.org/) and [Chart JS](https://www.chartjs.org/). The full list is given by [package.json](https://github.com/cuplsensor/cuplfrontend/blob/master/reactapp/package.json).

4. The hosted runner builds the cuplfrontend web application with ``npm run build``. A production-optimized version of the app is created in the build folder.

5. The GitHub Action [S3-Sync-Action](https://github.com/jakejarvis/s3-sync-action) copies files from the build folder into an Amazon S3 bucket. The cuplfrontend app can be served directly from the bucket, after web access is enabled. However, the domain name will be very long, which is no good for cupl. 

   In order to:

   * Reduce page load times.
   * Use a short, custom domain name together with an [Amazon-supplied SSL certificate](https://aws.amazon.com/certificate-manager/).

   an [Amazon Cloudfront](https://aws.amazon.com/cloudfront/) [CDN](https://en.wikipedia.org/wiki/Content_delivery_network) instance is placed 'in front' of the S3 bucket, from the user's perspective.

6. The hosted runner invalidates the Cloudfront cache. The latest cuplfrontend build is propagated to the CDN from S3.

### Backend Workflow

1. GitHub Actions launches a Linux [Hosted Runner](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#supported-runners-and-hardware-resources).
2. The hosted runner configures an SSH connection to the Droplet.
3. The hosted runner uses SSH to create a [Docker context](https://docs.docker.com/engine/context/working-with-contexts/). When this is employed, docker commands run remotely on the Droplet.
4. ``docker-compose --context remote pull`` causes 3 Docker images defined in [docker-compose.yml](docker-compose.yml) to be downloaded from DockerHub.
5. ``docker-compose --context remote up --build -d`` builds each Docker image into a container. 
6. Finally ``up`` in the previous command causes all containers to run ``-d`` daemonised on the Droplet.
 
Docker builds are configured from a list of environment variables. These are stored in this repository as encrypted GitHub Secrets.
