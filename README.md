# Tutorial T7 Cheese App:  Data Versioning Demo

In this tutorial go over data versioning techniques using the cheese app data. We will use Docker to run everything inside containers.

## Prerequisites
* Have Docker installed
* Cloned this repository to your local machine with a terminal up and running


### Clone the github repository
- Clone or download from [here](https://github.com/dlops-io/data-versioning)


## Make sure we do not have any running containers and clear up an unused images
* Run `docker container ls`
* Stop any container that is running
* Run `docker system prune`
* Run `docker image ls`

## Cheese App: Data Versioning
In this tutorial we will setup a data versioning step for the cheese app pipeline. We will use Docker to run everything inside containers.

### Fork the github repository
- Fork or download from [here](https://github.com/dlops-io/data-versioning)

Your folder structure should look like this:
```
   |-data-labeling
   |---docker-volumes
   |-----label-studio
   |-data-versioning
   |-secrets
```

- To view all the code open `data-versioning` folder in VSCode or any IDE of choice

### Create a Data Store folder in GCS Bucket
- Go to `https://console.cloud.google.com/storage/browser`
- Go to the bucket `cheese-app-data-demo` (REPLACE WITH YOUR BUCKET NAME)
- Create a folder `dvc_store` inside the bucket

## Run DVC Container
We will be using [DVC](https://dvc.org/) as our data versioning tool. DVC (Data Version Control) is an Open-source, Git-based data science tool. It applies version control to machine learning development, make your repo the backbone of your project.

### Setup DVC Container Parameters
In order for the DVC container to connect to our GCS Bucket open the file `docker-shell.sh` and edit some of the values to match your setup
```
export GCS_BUCKET_NAME="cheese-app-data-demo" [REPLACE WITH YOUR BUCKET NAME]
export GCP_PROJECT="ac215-project" [REPLACE WITH YOUR GCP PROJECT]
export GCP_ZONE="us-central1-a"

```
For windows open the file `docker-shell.bat`
```

```


### Run `docker-shell.sh` or `docker-shell.bat`
Based on your OS, run the startup script to make building & running the container easy

- Make sure you are inside the `data-versioning` folder and open a terminal at this location
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows

This will run a container that has DVC already installed. You can verify the containers running by `docker container ls` on another terminal prompt. You should see something like this:
```
CONTAINER ID   IMAGE                             COMMAND                  CREATED              STATUS              PORTS                                                      NAMES
00d808ab0386   data-label-cli                    "pipenv shell"           About a minute ago   Up About a minute                                                              data-labeling-data-label-cli-run
4ab1ec940b4a   heartexlabs/label-studio:latest   "./deploy/docker-ent…"   2 days ago           Up 2 days           0.0.0.0:8080->8080/tcp                                     data-label-studio
e87e8c6f180f   data-version-cli                  "pipenv shell"           5 seconds ago        Up 5 seconds                                                                   data-version-cli
```

### Download Labeled Data

In this step we will download all the labeled data from the GCS bucket and create `dataset_v1` version of our dataset.

- Go to the shell where ran the docker container for `data-versioning`
- Run `python cli.py -d`

If you check inside the `data-versioning` folder you should see the a `cheese_dataset` folder with labeled images in them.
```
   .
   |-cheese_dataset
   |---brie
   |---gouda
   |---gruyere
   |---parmigiano
   |-cheese_dataset_prep

```

The dataset from the data labeling step will be downloaded to a local folder called `cheese_dataset`

### Ensure we do not push data files to git
Make sure to have your gitignore to ignore the dataset folders. We do not want the dataset files going into our git repo.
```
/cheese_dataset_prep
/cheese_dataset
```

### Version Data using DVC
In this step we will start tracking the dataset using DVC

#### Initialize Data Registry
In this step we create a data registry using DVC
`dvc init`

#### Add Remote Registry to GCS Bucket (For Data)
`dvc remote add -d cheese_dataset gs://cheese-app-data-demo/dvc_store`

#### Add the dataset to registry
`dvc add cheese_dataset`

#### Push Data to Remote Registry
`dvc push`

You can go to your GCS Bucket folder `dvs_store` to view the tracking files


#### Update Git to track DVC
- First run git status `git status`
- Add changes `git add .`
- Commit changes `git commit -m 'dataset updates...'`
- Add a dataset tag `git tag -a 'dataset_v1' -m 'tag dataset'`
- Push changes `git push --atomic origin main dataset_v1`


### Download Data to view version
In this Step we will use Colab to view various version of the dataset
- Open [Colab Notebook](https://colab.research.google.com/drive/1UXfp9IDnzczGYTQ_tMLGsrrKH5F397_S?usp=sharing)
- Follow instruction in the Colab Notebook

## Make changes to data

### Use Label Studio to annotate some more data

- Go to Label Studio App at `http://localhost:8080/`
- Click on an item in the grid to annotate using the UI
- Repeat for a few of the images

### Download newly Labeled Data

In this step we will download the labeled data from the GCS bucket and create `dataset_v2` version of our dataset.

- Go to the shell where ran the docker container for `data-versioning`
- Run `python cli.py -d`

#### Add the dataset to registry
`dvc add cheese_dataset`

#### Push Data to Remote Registry
`dvc push`

#### Update Git to track DVC changes
- First run git status `git status`
- Add changes `git add .`
- Commit changes `git commit -m 'dataset updates...'`
- Add a dataset tag `git tag -a 'dataset_v2' -m 'tag dataset'`
- Push changes `git push --atomic origin main dataset_v2`


### Download Data to view version
In this Step we will use Colab to view the new version of the dataset
- Open [Colab Notebook](https://colab.research.google.com/drive/1UXfp9IDnzczGYTQ_tMLGsrrKH5F397_S?usp=sharing)
- Follow instruction in the Colab Notebook to view `dataset_v2`


By the end of this tutorial your folder structure should look like this:
```
   |-data-labeling
   |---docker-volumes
   |-----label-studio
   |-data-versioning
   |-cheese_dataset
   |---amanita
   |---crimini
   |---oyster
   |-cheese_dataset_prep
   |-secrets
```

### 🎉 Congratulations we just setup and tested data versioning using DVC

## Docker Cleanup
To make sure we do not have any running containers and clear up an unused images
* Run `docker container ls`
* Stop any container that is running
* Run `docker system prune`
* Run `docker image ls`
