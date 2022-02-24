# ASKCOS
Software package for the prediction of feasible synthetic routes towards a desired compound and associated tasks related to synthesis planning. Originally developed under the DARPA Make-It program and now being developed under the [MLPDS Consortium](http://mlpds.mit.edu).

# 2021.01 Release Notes

## askcos-site

User notes:  
* Implemented Ketcher molecule editor in IPP
* New feature to filter IPP results by reacting atoms
* Add SCScore termination criteria to tree builder
* Enable selection of buyables source in IPP and tree builder
* Expose number of pathways to return as a user option
* Enable disabling of price lookup in tree builder
* Support template attribute filters in IPP
* Add splash page during IPP loading
* Display solvent scores in condition recommender results
* Add support for new version of MCTS tree builder
* Implement pathway ranking and clustering model and API
* Redesign tree results page using jspanel
* Display precursor template rank in IPP
* Add QM descriptor model and API
* Add QM-based universal regioselectivity model and update general selectivity API
* Add new regioselectivity page in forward synthesis module
* Update navigation menu organization
* Add support for specifying tree builder result format
* Return tree lengths from tree builder API
* Add support for new regioselectivity model in IPP
* New reaction condition recommendation model added to API v2 and Forward Synthesis Planner
* Add search bar to home page as a new single-entry point to all tasks
* Replace aromatic site-selectivity page with new interactive interface in forward synthesis planner
* Replace atom mapping page with home page utility
* Switch draw page JS drawer to Ketcher
* Switch forward predictor JS drawer to Ketcher
* Add new "tree view" in IPP to highlight individual pathways
* Add feature to view all recommended templates in IPP
* Add pagination to tree list view panel in tree builder results page
* Update module overview page and add references
* Add option to invert reaction atoms filter to select conserved reaction atoms

Developer notes:
* Support custom django settings file
* Support celery task priorities
* Update Docker image Python version to 3.7 and RDKit version to 2020.03.06
* Move some processing of tree builder results from client to server
* Switch to api/v2 in forward prediction page
* Improved flexibility of mongodb, rabbitmq, and redis connections
* Switch to C++ version of rdchiral for improved performance
* Refactor IPP frontend code to use new graph data structure
* Add support for reaction SMILES to /api/v2/rdkit/smiles/canonicalize endpoint
* Reduce celery sub-tasks in impurity predictor worker for efficiency

Bug fixes:
* Enable precursor scoring using SCScore in IPP
* Do not load template in tree builder celery workers
* Canonicalize SMILES input in IPP
* Fix rabbitmq connection environment variable names
* Fix incorrect template attribute comparison operators in IPP
* Fix precursor re-clustering when loading tree builder results in IPP
* Fix treatment of duplicate chemicals in IPP
* Fix loading of tree builder results in treedata format

Breaking changes:
* API parameters changed for /api/v2/general-selectivity/ endpoint
* Graph format changed for saved tree builder results

## askcos-core

User notes:
* Add functions for reordering precursors using SCScore
* Add SCScore termination criteria to tree builder
* Enable disabling of price lookup in tree builder
* Filter templates by attribute filter
* Add solvent scores to the condition recommender
* Add new version of MCTS tree builder
* Add pathway ranking and clustering model
* Add QM descriptor model
* Add QM-based universal regioselectivity model
* Add new quantitative reaction condition recommendation model

Developer notes:
* Update Docker image Python version to 3.7 and RDKit version to 2020.03.06
* New pathway enumeration algorithm for tree builder
* Improve flexibility of mongodb connection parameters
* Return template rank with one-step predictions
* Switch to C++ version of rdchiral for improved performance
* Return graph attributes, including tree depth, in treedata json
* Add conda environment file to facilitate local installation of dependencies
* Adjust impurity predictor inspector to use forward predictor score

Bug fixes:
* Fix template drawing for aromatic structures
* Improved tracking of different buyable sources
* Tree builder no longer returns cyclic pathways
* Pass template_set to retro transformer
* Pass template_set to retrieve_template_metadata
* Do not run pathway ranking when there is 1 tree

Breaking changes:
* Return values changed for neural net context recommender get_n_conditions method
* Return values changed for MCTS tree builder tree_status method
* Return values changed for MCTS tree builder get_buyable_paths method

## askcos-data

Release notes:
* Update buyables data with sources
* Add solvent EHS score data
* Add retrosynthetic pathway ranking model
* Add pistachio template relevance model and data
* Add QM descriptor model
* Add QM-based universal regioselectivity model
* Add graph model for new reaction condition recommender

## askcos-deploy

User notes:
* Add support for custom django settings file
* Introduce Helm chart for Kubernetes deployment
* Add new condition recommender to deployment configuration

Developer notes:
* Add new tree builder services to deployment
* Add pathway ranking services to deployment
* Upgrade backend service images and switch to alpine where available
* Add pistachio template relevance TFX service to deployment
* Add QM descriptor and new regioselectivity services to deployment
* Switch mysql deployment from replication to standalone in Helm chart
* Add depends_on settings to docker-compose configuration
* Add init containers to control pod startup order in Helm chart

Bug fixes:
* Use COMPOSE_PROJECT_NAME in seed-db function
* Clear rabbitmq data when updating deployment
* Fix rabbitmq connection parameters in Helm chart
* Fix service status checking in init containers in Helm chart
* Increase nginx client_max_body_size in Helm chart

Deprecation:
* Previous k8 deployment configuration and script deprecated in favor of Helm chart

# Quickstart Using Google Cloud

```
# (1) Create a Google Cloud instance
#     - recommended specs: 16 vCPUs, 64 GB memory
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
