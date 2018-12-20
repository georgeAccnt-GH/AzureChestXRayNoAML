 
# Azure Chest Xay project outside AML
  
#### Author: George Iordanescu

## Introduction 
This repository contains the code related to the blog post: [Using Microsoft AI to Build a Lung-Disease Prediction Model using Chest X-Ray Images](https://blogs.technet.microsoft.com/machinelearning/2018/03/07/using-microsoft-ai-to-build-a-lung-disease-prediction-model-using-chest-x-ray-images/), by Xiaoyong Zhu, George Iordanescu, Ilia Karmanov, data scientists from Microsoft, and Mazen Zawaideh, radiologist resident from University of Washington Medical Center. While the blog [repo](https://github.com/Azure/AzureChestXRay) was developped using [Azure Machine Learning Services (AML)](https://azure.microsoft.com/en-us/services/machine-learning-services/) workbench, we provide here the non-AML version of the code, showing how one can leverage the power of [Microsoft AI platform](https://www.microsoft.com/en-us/ai/ai-platform) to build advanced analytics solutions using  powerful open source tools like docker, Jupyter notebooks, deep learning frameworks like [Keras](https://keras.io/) (with [TensorFlow](https://www.tensorflow.org/) backend) on multi-GPU enabled [Azure Deep Learning Virtual Machines (DLVM)](http://aka.ms/dlvm).  
  
Understanding the inner works of training and deploying deep-learning models is important for developping new models and also highlights the critical benefits of using Azure Machine Learning Services for  training, operationalization and model management.  


## Step by step instructions:
 * Deply an [Azure Deep Learning Virtual Machines (DLVM)](http://aka.ms/dlvm)
 * Open up ports for ssh, plus 2 Jupyter Notebook servers (one plain and the other one used for building the dockerized training and scoring scripts)
 * Add disks or expand the current ones as needed (you will need several 100 GB to store data and images)
 * Move/download [NIH](https://www.nih.gov/news-events/news-releases/nih-clinical-center-provides-one-largest-publicly-available-chest-x-ray-datasets-scientific-community) [Chest X-Ray data](https://nihcc.app.box.com/v/ChestXray-NIHCC) (12 images_xxx.tar.gz files) into an [Azure blob storage](https://azure.microsoft.com/en-us/services/storage/blobs/) account. Make sure you know the container address and its key
 * We will use a Jupyter notebook running on the provisioned Azure DLVM to create the training docker image
 * Training docker image will run in a container on the same DLVM. We'll connect to it via a second Jupyter Notebook server, and we will develop the training script and train a deep learning model for image classification.
 * The trained model and its associated scring script will then be deployed via a scoring docker image on an a [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/en-us/services/kubernetes-service/) cluster. 
* login (ssh) into the VM and create the project base directory structure:
```python
sudo mkdir -p /data/datadrive01
sudo chmod -R ugo=rwx  /data/datadrive01/
sudo mkdir -p /data/datadrive01/prj
sudo mkdir -p /data/datadrive01/data
sudo chmod -R ugo=rwx  /data/datadrive01/
```
* Login into dockerhub:
```
docker login
```
* [Get rid of sudo](https://docs.docker.com/install/linux/linux-postinstall/#manage-docker-as-a-non-root-user) in cli if you wish so:
```
sudo groupadd docker
sudo usermod -aG docker $USER
```
* Update/install a few system libs:
```
sudo apt-get update
pip install --upgrade pip
sudo apt-get install tmux
pip install -U python-dotenv
```
* Clone the project:
```
cd /data/datadrive01/prj/
git clone https://github.com/georgeAccnt-GH/AzureChestXRayNoAML.git
sudo chmod -R ugo=rwx  /data/datadrive01/
```
* The project code structure is shown below. Data will be downloaded by notebooks and stored in /data/datadrive01/data/:
```
ls -l /data/datadrive01/prj/AzureChestXRayNoAML/
total 16
drwxrwxrwx 6 loginVM_001 loginVM_001 4096 Sep 26 17:08 code
drwxrwxrwx 2 loginVM_001 loginVM_001 4096 Sep 26 16:56 docker
-rwxrwxrwx 1 loginVM_001 loginVM_001 1161 Sep 26 16:56 LICENSE
-rwxrwxrwx 1 loginVM_001 loginVM_001 2828 Sep 26 16:56 README.md
```
* Start the base jupyter notebook server on the vm (via tmux if you want to still have cli control):
```
cd /data/datadrive01/
tmux attach-session -t jupyter_srvr
jupyter notebook --notebook-dir=$(pwd) --ip='*' --port=10002 --no-browser --allow-root
```
* If you can not save notebooks, run these commands to enable write rigths to your directories:
```
sudo chmod -R ugo=rwx  /data/datadrive01/

# to change directories' ownership
sudo chown -R loginVM_001:loginVM_001 /data/datadrive01/prj/

# to change files'  ownership
sudo find . -type f  | xargs sudo chown loginVM_001:loginVM_001

```
* You can nonnect to the base Jupyter notebook server from your local machine by using the appropriate port (e.g. __10002__ below) and the tocken reported by the server on the VM in the tmux session:
```
http://ghiordtlvisgpvm.southcentralus.cloudapp.azure.com:10002/some_token
```


* Go to AzureChestXRayNoAML/code/00_create_docker_image.ipynb and generate the training doscker file and associated docker image, The command to start the training docker container is printed towards the end of the notebook, as shown below. Port __10003__  shown below is the one used for the second (training, dockerized) Jupyter notebook server, adn it should match the third port opened on the VM.
```
nvidia-docker run -i -t -p 10003:8888 -v $(pwd):/local_dir:rw ...
```
* You can run the above command in a new tmux session:
```
tmux new -s jupyter_srvr_docker
sudo nvidia-docker run -i -t -p 10003:8888 -v $(pwd):/local_dir:rw georgedockeraccount/chestxray-no-aml-gpu:1.0.1 /bin/bash -c "/opt/conda/bin/jupyter notebook --notebook-dir=/local_dir --ip=* --port=8888 --no-browser --allow-root"
```
* You can nonnect to the training dockerized Jupyter notebook server from your local machine by using the other port (e.g. __10003__ below) and the tocken reported by the server on the VM in the tmux session:
```
http://ghiordtlvisgpvm.southcentralus.cloudapp.azure.com:10003/some_token
```
* Follow the project notebooks (use edit_python_files.ipynb to edit .py files as needed):
  - AzureChestXRayNoAML/code/01_DataPrep/001_get_data.ipynb to get the data (from the storage account where you downloaded NIH image data and auxiliary files (BBox_List_2017.csv, blacklist.csv and Data_Entry_2017.csv)
  - AzureChestXRayNoAML/code/02_Model/000_preprocess.ipynb
  - AzureChestXRayNoAML/code/02_Model/010_train.ipynb
  
