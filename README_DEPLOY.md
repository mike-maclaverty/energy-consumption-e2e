# The Full Stack 7-Steps MLOps Framework

### LIVE DEMO [FORECASTING](http://35.207.134.188:8501/) | LIVE DEMO [MONITORING](http://35.207.134.188:8502/)

--------

# Deploy to GCP

This step must only be finished if you want to deploy the code on GCP VMs and build the CI/CD with GitHub Actions.

Note that this step might result in a few costs on GCP. It won't be much. While I was developing this course, I spent only ~20$, and it will probably be less for you.

Also, you can get some free credits if you have a new GCP account. Just be sure to delete the resources after you finish the course.


## General Set Up

Before setting up the code, we need to go to our GCP `energy_consumption` project and create a few resources. After we can SSH to our machines and deploy our code.


#### GCP Resources

- create static external IP address - [docs](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address#console)


### Admin VM Service Account with IAP Access

We need a new GCP service account that has admin rights when working with GCP VMs & IAP access. You have to assign the following roles:
* Compute Instance Admin (v1)
* IAP-secured Tunnel User
* Service Account Token Creator
* Service Account User

IAP stands for Identity-Aware Proxy. It is a way to create tunnels that route TCP traffic. For your knowledge, you can read more about this topic using the following docs:
* [Using IAP for TCP forwarding](https://cloud.google.com/iap/docs/using-tcp-forwarding)
* [Overview of TCP forwarding](https://cloud.google.com/iap/docs/tcp-forwarding-overview)

### Expose Ports Firewall Rule

Create a firewall rule that exposes the following TCP ports: 8501, 8502, 8001.

Also, add a `target tag` called `energy-forecasting-expose-ports`.

Here is how my firewall rule looks like:

<p align="center">
  <img src="images/gcp_expose_ports_firewall_rule_screenshot.png">
</p>

Here are 2 docs that helped me create and configure the ports for the firewall rule:
* [Doc 1](https://stackoverflow.com/questions/21065922/how-to-open-a-specific-port-such-as-9090-in-google-compute-engine)
* [Doc 2](https://www.howtogeek.com/devops/how-to-open-firewall-ports-on-a-gcp-compute-engine-instance/)


### IAP for TCP Tunneling Firewall Rule

Now we will create a firewall rule that will allow IAP for TCP Tunneling on all the VMs that are connected to the `default` network.

[Docs on how to create the firewall rule.](https://cloud.google.com/iap/docs/using-tcp-forwarding#preparing_your_project_for_tcp_forwarding)

Here is how my firewall rule looks like:

<p align="center">
  <img src="images/gcp_iap_for_tcp_firewall_rule.png">
</p>


### VM for the Pipeline

Go to your GCP `energy_consumption` project -> VM Instances -> Create Instance

Choose `e2-standard-2: 2 vCPU cores - 8 GB RAM`

Call it: `ml-pipeline`

Change the disk to `20 GB Storage`

Pick region `europe-frankfurt` and zone `europe-west3-c`

Network: `default`

Also, check the `HTTP` and `HTTPS` boxes and add the `energy-forecasting-expose-ports` custom firewall rule we did a few steps back.

Here are 2 docs that helped me create and configure the ports for the firewall rule:
* [Doc 1](https://stackoverflow.com/questions/21065922/how-to-open-a-specific-port-such-as-9090-in-google-compute-engine)
* [Doc 2](https://www.howtogeek.com/devops/how-to-open-firewall-ports-on-a-gcp-compute-engine-instance/)


### VM for the Web App

Go to your GCP `energy_consumption` project -> VM Instances -> Create Instance

**Here are the VM configurations:**

This one can be as small as: `e2-micro: 0.25 2 vCPU - 1 GB memory` 

Call it: `app`

Change the disk to: `15 GB standard persisted disk`

Pick region `europe-frankfurt` and zone `europe-west3-c`

Network: `default`

Also, check the `HTTP` and `HTTPS` boxes and add the `energy-forecasting-expose-ports` custom firewall rule we did a few steps back.

Here are 2 docs that helped me create and configure the ports for the firewall rule:
* [Doc 1](https://stackoverflow.com/questions/21065922/how-to-open-a-specific-port-such-as-9090-in-google-compute-engine)
* [Doc 2](https://www.howtogeek.com/devops/how-to-open-firewall-ports-on-a-gcp-compute-engine-instance/)


### External Static IP

If we want the external IP for our web app to be static (aka not to change) we have to attach a static address to our web app VM.

More exactly, to the `app` VM we created a few steps ahead. If you want to repeat this process for the `ml-pipeline` VM that is perfetly fine. 

[Docs on reserving a static external IP address.](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address)


----

#### Finally, all the boring setup is done. Let's start deploying our code 👇 👇 👇 


## Deploy - General Steps

### Configure Your Service Account

We will use your service account configured with admin rights for VMs and IAP access to SSH from your local machine to the GCP VMs.

First, we have to tell the `gcloud` GCP CLI to use that service account.

To do so, you have to create a key for your service account and download it as a JSON file (same as you did for the buckets service accounts - [here are some docs to refresh your mind](https://cloud.google.com/iam/docs/keys-create-delete)).

After you downloaded the file you just have to run the following `gcloud` command:
```shell
gcloud auth activate-service-account SERVICE_ACCOUNT@DOMAIN.COM --key-file=/path/key.json --project=PROJECT_ID
```

[Check out this docs for more details about the gcloud auth command](https://cloud.google.com/sdk/gcloud/reference/auth/activate-service-account).

Now whenever you run commands with `gcloud` it will use this service account to authenticate.


## Deploy - The Pipeline

Let's connect through SSH to the `ml-pipeline` GCP VM you created a few steps ahead:
```shell
gcloud compute ssh ml-pipeline --zone europe-west3-c --quiet --tunnel-through-iap --project <your-project-id>
```
**NOTE 1:** Change the `zone` if you haven't created a VM with the same zone as us.<br/>
**NOTE 2:** Your `project-id` is NOT your `project name`. Go to your GCP projects list and you will find the project id.

Starting this point, if you configured the firewalls and service account OK, as everything is Dockerized, all the steps will be 99% similar as the ones from the [Set Up Additional Tools](https://github.com/iusztinpaul/energy-forecasting#-set-up-additional-tools-) and [Usage](https://github.com/iusztinpaul/energy-forecasting#usage) sections.

You can go back and follow the exact same steps, while your terminal has an SSH connection with the GCP machine.

Note that the GCP machine is a linux machine. Thus, this time, the commands I used in the README.md will work just fine.

<p align="center">
  <img src="images/gcp_ssh_screenshot.png">
</p>

Now you have to repeat all the steps you've done setting `The Pipeline` locally, but using this SSH connection.

### BUT YOU HAVE TO KEEP IN MIND THE FOLLOWING:

**Clone the code in the home directory of the VM:**

Just SHH to the VM and run:
```shell
git clone https://github.com/iusztinpaul/energy-forecasting.git
cd energy-forecasting
```

**Install Docker using the following commands:** <br/><br/>
Install Docker:
```shell
sudo apt update
sudo apt install --yes apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt update
sudo apt install --yes docker-ce
```
Add sudo access to Docker:
```shell
sudo usermod -aG docker $USER
logout 
```
Login again to your machine:
```shell
gcloud compute ssh ml-pipeline --zone europe-west3-c --quiet --tunnel-through-iap --project <your-project-id>
```

**Replace all `cp` commands with `gcloud compute scp`:** <br/><br/>

This command will help you to copy files from your local machine to the VM.

For example, instead of running:
```shell
cp -r /path/to/admin/gcs/credentials/admin-buckets.json credentials/gcp/energy_consumption
```

Run in a different terminal (not the one connected with SSH to your VM):
```shell
gcloud compute scp --recurse --zone europe-west3-c --quiet --tunnel-through-iap --project <your-project-id> /local/path/to/admin-buckets.json ml-pipeline:~/energy-forecasting/airflow/dags/credentials/gcp/energy_consumption/
```
This command will your local `admin-buckets.json` file directly to the `ml-pipeline` VM.


**!!!** And this is all. All the other steps are the same as running locally. Basically, only Docker has a slight different installation and you need a different way to copy files from your local machine to the VM. 


Now go to your VM view from GCP and go to the `Network tags` section. There you will find the `External IP address` column as shown in the image bellow. Copy that ip and attach port `8080` and vualua. You connected to your self hosted Airflow application.

Note that if it doesn't, give it a few seconds to load up properly.

For example, based on the `External IP address` from the image below I accessed Airflow using this address: `35.207.134.188:8080`. 

<p align="center">
  <img src="images/gcp_vm_external_ip_screenshot.png">
</p>


## Deploy - The Web App
Let's connect through SSH to the `app` GCP VM you created a few steps ahead:
```shell
gcloud compute ssh app --zone europe-west3-c --quiet --tunnel-through-iap --project <your-project-id>
```
**NOTE 1:** Change the `zone` if you haven't created a VM with the same zone as us.<br/>
**NOTE 2:** Your `project-id` is NOT your `project name`. Go to your GCP projects list and you will find the project id.

This time you are in luck. You can repeat the exact same steps as in the  























## Poetry

Install Python system dependencies:
```shell
sudo apt-get install -y python3-distutils
```

```shell
curl -sSL https://install.python-poetry.org | python3 -
nano ~/.bashrc
```

Add `export PATH=~/.local/bin:$PATH` to `~/.bashrc`

Check if Poetry is intalled:
```shell
source ~/.bashrc
poetry --version
```


## Private PyPi Server Credentials
Install pip:
```shell
sudo apt-get -y install python3-pip
```

Create credentials:
```shell
sudo apt install -y apache2-utils
pip install passlib

mkdir ~/.htpasswd
htpasswd -sc ~/.htpasswd/htpasswd.txt energy-forecasting
```
Set credentials:
```shell
poetry config repositories.my-pypi http://localhost
poetry config http-basic.my-pypi energy-forecasting <password>
```
Check credentials:
```shell
cat ~/.config/pypoetry/auth.toml
```



## Install Docker

Install Docker on GCP instructions [here](https://tomroth.com.au/gcp-docker/).

TLDR
```shell
sudo apt update
sudo apt install --yes apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt update
sudo apt install --yes docker-ce
```
docker sudo access:
```shell
sudo usermod -aG docker $USER
logout 
```

### Setup

#### Run

clone repo
```shell
git clone https://github.com/iusztinpaul/energy-forecasting.git
cd energy-forecasting
```

```shell
# Move to the airflow directory.
cd airflow

# Make expected directories and set an expected environment variable
mkdir -p ./logs ./plugins
sudo chmod 777 ./logs ./plugins
echo -e "AIRFLOW_UID=$(id -u)" > .env
echo "ML_PIPELINE_ROOT_DIR=/opt/airflow/dags" >> .env



cd ./dags
# Copy bucket writing access GCP service JSON file
mkdir -p credentials/gcp/energy_consumption
gcloud compute scp --recurse --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512 ~/Documents/credentials/gcp/energy_consumption/admin-buckets.json ml-pipeline:~/energy-forecasting/airflow/dags/credentials/gcp/energy_consumption/

touch .env
# Complete env vars from the .env file
# Check .env.default for all possible variables.
gcloud compute scp --recurse --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512 ~/Documents/projects/energy-forecasting/airflow/dags/.env ml-pipeline:~/energy-forecasting/airflow/dags/

# Initialize the database
cd <airflow_dir>
docker compose up airflow-init

# Start up all services
# Note: You should setup the PyPi server credentials before running the docker containers.
docker compose --env-file .env up --build -d
```


#### Set Variables
ml_pipeline_days_export = 30
ml_pipeline_feature_group_version = 5
ml_pipeline_should_run_hyperparameter_tuning = False

## Build & Publish Python Modules
Set experimental installer to false:
```shell
poetry config experimental.new-installer false
```
Run the following to build and publish all the modules:
```shell
cd <root_dir>
sh deploy/ml-pipeline.sh
```
**NOTE:** Be sure that are modules are deployed before starting the DAG. Otherwise, it won't know how to load them inside the DAG. 


### GCP

install gcp SDK:
```shell
```

IAM principals - service accounts:
* read-buckets
* admin-buckets
* admin-vm

Firewall rules:
* IAP for TCP tunneling [docs](https://cloud.google.com/iap/docs/using-tcp-forwarding)
* Expose port 8080

[Open 8080 Port](https://stackoverflow.com/questions/21065922/how-to-open-a-specific-port-such-as-9090-in-google-compute-engine)
[Open 8080 Port](https://www.howtogeek.com/devops/how-to-open-firewall-ports-on-a-gcp-compute-engine-instance/)


VM machine:
* 2 vCPU cores - 8 GB RAM / e2-standard-2 with 20 GB Storage

create VM machine `ml-pipeline`:
```shell
```

connect to VM machine through shh:
```shell
gcloud compute ssh ml-pipeline --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512
```
###### Set credentials for GitHub Actions

print json credentials in one line:
```shell
jq -c . admin-vm.json 
```

set private key:
```
```



# Run APP
Copy the GCP credentials with which you can read from the GCP buckets:
```shell
mkdir -p credentials/gcp/energy_consumption
cp your/location/file.json credentials/gcp/energy_consumption/
```
Create `.env` file:
```shell
cp app-api/.env.default app-api/.env
# Change values in .env if necessary
```
Build & run:
```shell
docker compose -f deploy/app-docker-compose.yml --project-directory . up --build
```
Run local dev from root dir:
```shell
docker compose -f deploy/app-docker-compose.yml -f deploy/app-docker-compose.local.yml --project-directory . up --build
```


### Deploy APP on GCP

#### GCP Resources

- VM: e2-micro - 0.25 2 vCPU - 1 GB memory - 15 GB standard persisted disk
- firewall: expose ports 8501, 8502, 8001
- firewall: IAP for TCP tunneling
- create static external IP address - [docs](https://cloud.google.com/compute/docs/ip-addresses/reserve-static-external-ip-address#console)
- service roles for: reading buckets & SSH access

#### Commands

Connect to VM:
```shell
gcloud compute ssh app --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512
```
Install requirements:
```shell
sudo apt-get update && sudo apt-get upgrade
sudo apt-get install -y git
```
Install docker:
```shell
sudo apt update
sudo apt install --yes apt-transport-https ca-certificates curl gnupg2 software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
sudo apt update
sudo apt install --yes docker-ce

# docker sudo access:
sudo usermod -aG docker $USER
logout
```
SSH again:
```shell
gcloud compute ssh app --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512
```
Clone repo:
```shell
git clone https://github.com/iusztinpaul/energy-forecasting.git
cd energy-forecasting
```
Create credentials folder:
```shell
mkdir -p credentials/gcp/energy_consumption
```
Copy GCP credentials JSON file:
```shell
gcloud compute scp --recurse --zone europe-west3-c --quiet --tunnel-through-iap --project silver-device-379512 ~/Documents/credentials/gcp/energy_consumption/read-buckets.json app:~/energy-forecasting/credentials/gcp/energy_consumption/
```
Create `.env` file:
```shell
cp app-api/.env.default app-api/.env
```
Install numpy to speed up IAP TCP upload bandwidth:
```shell
$(gcloud info --format="value(basic.python_location)") -m pip install numpy
```
Build & run:
```shell
docker compose -f deploy/app-docker-compose.yml --project-directory . up --build
```