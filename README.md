 
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
