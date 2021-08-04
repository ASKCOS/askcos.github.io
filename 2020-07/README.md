# ASKCOS
Software package for the prediction of feasible synthetic routes towards a desired compound and associated tasks related to synthesis planning. Originally developed under the DARPA Make-It program and now being developed under the [MLPDS Consortium](http://mlpds.mit.edu).

# 2020.07 Release Notes

## askcos-core

Developer notes:
* Refactored MCTS code to decouple pure python code from celery infrastructure
* Added Makefile to facilitate building docker images
* Renamed `makeit` module to `askcos`
* Revert to default coordinate generator for RDKit drawing
* Use askcos-base as base image for Docker build

Bug fixes:
* Running the main tree.builder.py file as a script or the tree builder unit test with multiprocessing did not work
* Atomic identity should not change in a tree builder reaction prediction
* Catch general selectivity error when target is not recovered
* Tree builder returns the incorrect supporting template
* Fix template db lookup in tree builder to include template_set
* Catch error when retro template cant be parsed by rdchiral when reversed
* Do not use isomeric smiles for TFFP prediction

## askcos-site

User notes:  
* New general selectivity model available from the Interactive Path Planner UI and as an API endpoint
* Redesigned and consolidated forward prediction UI combining reaction condition prediction, forward synthesis prediction, and impurity prediction
* Drawing tool added to Interactive Path Planner UI
* The Interactive Path Planner now saves last used settings locally in the browser, and more visualization settings are exposed to the user
* Users can now initiate a tree builder search from the Interactive Path Planner
* Option added to automatically redirect to the Interactive Path Planner UI upon completion of tree builder (using new UI)
* Show building block source in the Interactive Path Planner and tree visualization UIs
* Reaction precedents for new template sets can be viewed in the UI
* Redesigned page for viewing and adding banned chemicals and reactions
* Add api v2 drawing endpoint and new drawing page
* Ask for confirmation before clearing IPP results
* Refine user interface and upgrade to Bootstrap 4
* Add option to completely disable third-party name resolution

Developer notes:
* Enabled versioning for the API and introduces API v2
* API endpoint created for the atom mapping tool
* API endpoint created for the impurity predictor
* Three new scoring coordinator specific workers have been created to handle template-free prediction, template-based prediction, and fast filter evaluation
* Reconfigure the Docker image so that new templates can be added without the image needing to be rebuilt
* Chemical historian information has been migrated into the mongodb
* Make it easier to use retrained models and template sets
* Use tokens to authenticate users that make API calls
* Added Makefile to facilitate building docker images
* Added option to retain atom mapping for forward prediction API calls
* Reduce initial website load time
* Optimize CI build time
* Add info about how to build image and use —local
* Use SUPPORT_EMAILS environment variable for support form
* Enable support for TensorFlow Serving model versions from TFServingAPIModel
* Add modularity for new template relevance models to use different input sizes
* Improve loading of cached IPP options to handle new options
* Rename blacklist to banlist across askcos-site
* Pass template prioritizer version to tree builder
* Download new versions of browser cached static files on upgrades

Bug fixes:
* Remove broken links from the old context pages
* Forward Predictor API may return -Infinity in JSON response
* Fix available worker calculation on Celery status page
* Handle loss of connection error more gracefully during async polling in client
* Fix issue preventing regioselectivity check from being disabled in IPP
* Remove hard-coded template set names in retro api (v2) endpoint
* Properly pass cluster settings in `tb_c_worker.get_top_precursors`
* Prevent logout/login loop
* Re-enable multi-select in IPP to remove multiple nodes
* Fix backup issue by reverting to drf-jwt 1.14
* Remove custom css for browsable api due to Bootstrap version incompatibility

## askcos-deploy

User notes:  
* Additional data can now be added/appended to collections in the mongodb via the deploy script without clearing the collection first
* Add new docker volume based backup and restore commands to deploy.sh
* Add option to completely disable third-party name resolution
* Compare user customizable environment files when deploying
* Expose SUPPORT_EMAILS as an environment variable on deployment
* Add functionality in health_check.py to fix tree builder timeout

Developer notes:
* Updates to k8 configuration
* Change mongo db index types for better performance
* Update deployment for repository split

Bug fixes:
* Add pull-images step to deploy-http command
* Use COMPOSE_PROJECT_NAME from .env in backup.sh and restore.sh

# Quickstart Using Google Cloud

```
# (1) Create a Google Cloud instance
#     - recommended specs: 8 vCPUs, 64 GB memory
#     - select Ubuntu 18.04 LTS Minimal
#     - upgrade to a 100 GB disk
#     - allow HTTP and HTTPS traffic

# (2) Install docker
#     - https://docs.docker.com/engine/install/ubuntu/
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common -y
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io -y
sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

# (3) Install docker-compose
#     - https://docs.docker.com/compose/install/
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

# (4) Clone askcos-deploy
git clone https://github.com/ASKCOS/askcos-deploy
cd askcos-deploy

# (5) Deploy (automatically pulls image from Docker Hub)
bash deploy.sh deploy     # start containers (detached) and run other initialization tasks
docker-compose logs -f    # start tailing logs (can CTRL+C to exit)

# (6) Navigate to your instance's external IP
#     - note that it may take ~5 minutes for the retro transformer workers to start up
#     - you can check the status of their startup by looking at "server status"
#     - the first request to a website process may take ~10 seconds
#     - the first request to a retro transform worker may take ~5-10 seconds
#     - the first request to the forward predictor may take ~60 seconds
```

# First Time Deployment

## Software Prerequisites

To deploy ASKCOS, you must have the following installed on your machine:
* git
* Docker ([installation instructions](https://docs.docker.com/install/))
* Docker Compose ([installation instructions](https://docs.docker.com/compose/install/#install-compose))

## (Optional) Building Docker Images

Building the askcos-site Docker image is now more involved following the repository restructuring. We highly recommend
using the pre-built image from Docker Hub if you do not need to modify the code.

To only build askcos-site using a pre-built askcos-core image:
```bash
$ git clone https://github.com/ASKCOS/askcos-site
$ cd askcos-site
$ make [TAG=my_tag]
```

A Makefile is provided to make it easier to build the image with a default image name.
You can also use the `docker build` command directly:
```bash
$ docker build -t <image name>:<tag> .
```

__NOTE:__ The image name should correspond with what exists in the `docker-compose.yml` file. By default, the image name is environment variable `ASKCOS_IMAGE_REGISTRY` + `askcos-site`. If you choose to use a custom image name, make sure to modify the `ASKCOS_IMAGE_REGISTRY` variable or the `docker-compose.yml` file accordingly.

Similarly, if you also want to build askcos-core:
```bash
$ git clone https://github.com/ASKCOS/askcos-core
$ cd askcos-core
$ make [TAG=my_tag]
```

Note that you will need to specify the appropriate askcos-core version when building askcos-site afterwards:
```bash
$ cd askcos-core
$ make TAG=my_tag
$ cd ../askcos-site
$ make CORE_VERSION=my_tag TAG=my_tag
```

## Deploying the Web Application

Deployment is initiated by a bash script that runs a few docker-compose commands in a specific order.
Several database services need to be started first, and more importantly seeded with data, before other services 
(which rely on the availability of data in the database) can start.
The `deploy.sh` script is provided in the askcos-deploy repository and should be run as follows:

```bash
$ cd askcos-deploy
$ bash deploy.sh command [optional arguments]
```

There are a number of available commands, including the following for common deploy tasks:
* `deploy`: runs standard first-time deployment tasks, including `seed-db`
* `update`: pulls new docker image from GitLab repository and restarts all services
* `seed-db`: seed the database with default or custom data files
* `start`: start a deployment without performing first-time tasks
* `stop`: stop a running deployment
* `clean`: stop a running deployment and remove all docker containers

For a running deployment, new data can be seeded into the database using the `seed-db` command along with arguments
indicating the types of data to be seeded. Note that this will replace the existing data in the database.
The available arguments are as follows:
* `-b, --buyables`: specify buyables data to seed, either `default` or path to data file
* `-c, --chemicals`: specify chemicals data to seed, either `default` or path to data file
* `-x, --reactions`: specify reactions data to seed, either `default` or path to data file
* `-r, --retro-templates`: specify retrosynthetic templates to seed, either `default` or path to data file
* `-f, --forward-templates`: specify forward templates to seed, either `default` or path to data file

For example, to seed default buyables data and custom retrosynthetic pathways, run the following from the deploy folder:

```bash
$ bash deploy.sh seed-db --buyables default --retro-templates /path/to/my.retro.templates.json.gz
```

To update a deployment, run the following from the deploy folder:

```bash
$ bash deploy.sh update --version x.y.z
```

To stop a currently running application, run the following from the deploy folder:

```bash
$ bash deploy.sh stop
```

If you would like to clean up and remove everything from a previous deployment (__NOTE: you will lose user data__), run the following from the deploy folder:

```bash
$ bash deploy.sh clean
```

# Important Notes

## Recommended hardware

We recommend running this code on a machine with at least 8 compute cores (16 preferred) and 64 GB RAM (128 GB preferred).

## First startup

The celery worker will take a few minutes to start up (possibly up to 5 minutes; it reads a lot of data into memory from disk). The web app itself will be ready before this, however upon the first get request (only the first for each process) a few files will be read from disk, so expect a 10-15 second delay.

## Scaling Workers

Only 1 worker per queue is deployed by default with limited concurrency. This is not ideal for many-user demand. 
The scaling of each worker is defined at the top of the `deploy.sh` script. 
To scale a desired worker, change the appropriate value in `deploy.sh`, for example:
```
n_tb_c_worker=N          # Tree builder chiral worker
```

where N is the number of workers you want. Then run `bash deploy.sh start [-v <version>]`.

## Managing Django

If you'd like to manage the Django app (i.e. - run python manage.py ...), for example, to create an admin superuser, you can run commands in the _running_ app service (do this _after_ `docker-compose up`) as follows:

```bash
$ docker-compose exec app bash -c "python /usr/local/askcos-site/manage.py createsuperuser"
```

In this case you'll be presented an interactive prompt to create a superuser with your desired credentials.
