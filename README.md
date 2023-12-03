# Wine_Quality_Preddiction_ECS_Docker

<p>This guide provides a detailed explanation of the process for utilizing AWS services to train a Machine Learning (ML) model on multiple parallel EC2 instances within the Elastic Compute Cloud. The ML program is developed in Python language, incorporating the [Apache Spark MLlib libraries](https://spark.apache.org/mllib/). Both the training and prediction programs are configured to execute within a container.  <br>

[Github link](https://github.com/pavanvenkatreddy/Wine_Quality_Preddiction_ECS_Docker)  <br>
*Docker Image for Training*: [pavanvenkatbhumula/wine:v1](https://hub.docker.com/repository/docker/pavanvenkatbhumula/wine/general)
<br>
*Docker Image for Testing*: [pavanvenkatbhumula/winetest:v1](https://hub.docker.com/repository/docker/pavanvenkatbhumula/winetest/general)

To run the ML Image application for training on multiple parallel EC2 instances, a cluster needs to be set up in ECS. The steps given below are followed to create a cluster with 4 instances.
<br>
<br>In AWS Management Console search for **Elastic Container Service (ECS)**

  In ECS Console, select **Cluster** and click on *"Create Cluster"*
  <br>
  __*Cluster name*__ : *winequality* ~your desired name
  <br>
  __*Provisioning model*__ : *On-Demand Instance*
  <br>
  __*EC2 instance type*__ : *t2.micro*
  <br>
  __*Number of instances*__ : *4*
  <br>
  __*Key Pair*__ : *Choose an appropriate key pair*
  <br>
  __*Security group inbound rules (Port range)*__ : *22-80* 
  <br>
<br>- Click on *"Create"*  to create a cluster 
<br>
This can be verified on EC2 dashboard and checking the number of EC2 instances running.
## Setting up Task Definitions and Tasks
We have successfully created a cluster but there is no task running on our instances. In order to run our ML container application we need to create a "Task Definition" which describes information regarding the containers to use, bind mounts, volumes etc. These are done as follows: 
- In **ECS console** select *"Task Definitions"*
- Click on *"Create New task Definition"*
- Choose **Select launch type compatibility** as "EC2"

This will open up **Configure task and container definitions** screen where we have to configure the parameters for our docker conatiner to run correctly. The ML conatiner application used for training outputs 2 files,  *Modelfile*  and *reults.txt*. The *Modelfile* is used in the prediction application and *results.txt* gives metrics of the training model against ValidationDataset.csv. A bind-mount is required between the Docker container and the host to access these files. Do the following configuration 
- __*Task Definition Name*__ : *winequalitytask*
- __*Task Role*__ : *ecsTaskExecutionRole*

The following is done so that the files generated by the docker application is available to the host.
Under __*Volumes*__ click on *"Add volume"* 
- __*Name*__ : *host-path*
- __*Volume type*__ : *Bind Mount*
- __*Source path*__ : */home/ec2-user*

Now we will configure the ML container for training 
Under __*Container Definitions*__ click on *"Add container"* 
- __*Container Name*__ : *winequality*
- __*Image*__ : *pavanvenkatbhumula/winetrain:v1*
- __*Memory Limits*__ : *Soft Limit, 512

Under __*Mount points*__ ,  
- __*Source volume*__ : *host-path*
- __*Container path*__ : */job*
- Click on *"Add"* and then click on *"Create"*

We have successfully created our *"Task Definition"* , now we have create a Task which will initiate our docker container application on the EC2 instances. Do the following to run the task:
- On ECS console click on **Cluster**
- Select the cluster recently created (wine-train-quality-cluster)
- Select **Tasks** tab and click on *"Run new Task"*
- Select **Launch type** as *"EC2"* and select **Number of Task** as *"4"*
- Click on *"Run Task"*

This will download ML conatiner application for training on EC2 instances if not present and starts executing it.
Once the execution is completed two files named *"Modelfile"* and *"results.txt"* should be present in the home directory (/home/ec2-user) of all the running EC2 instances. This *"Modelfile"* can be downloaded from any one these instances which will be used in the prediction application. We can use [Cyberduck](https://cyberduck.io/) to transfer the files to and from ec2 instance.

## Running the Prediction Application on AWS
The ML container for prediction uses 2 files as input  *"Modelfile"*, *"TestDataset.csv"* .
The container application takes the *"TestDataset.csv"* as input and applies the model from *"Modelfile"* and generates a csv file with prediction output. The detailed steps are explained below.

### Installing Docker and downloading the prediction container
In order to run the prediction conatiner docker package is required. To install docker in Amazon linux machine the following steps are done
- Run the following command
```Console
    $ sudo yum install docker -y && sudo systemctl start docker
```
- The docker is pulled with the following command 
```Console
    $ sudo docker pull pavanvenkatbhumula/winetest:v1
```
- Run the following command to verify if the container has been installed
```Console
    $ sudo docker images
```

### Running the docker container 
After installing the image, two files must be present in order to run the container application properly (*Modelfile* and *Inputdataset.csv*). These two files can be uploaded on to the instances using WinSCP. To run the container use the following command
```Console
  $ sudo docker run -v /home/ec2-user:/job pavanvenkatbhumula/winetest:v1 TestDataset.csv
```
where ,
`/home/ec2-user` is the path to the home directory in the instance.
`/job` is the path mapped inside the conatiner
`pavanvenkatbhumula/winetest:v1` is the name of prediction docker image
`TestDataset.csv` is the name of the input file for prediction testing.

*Note: For the above command to work the Modelfile and TestDataset.csv must be present at /home/ec2-user*

### F1 Score of the Model
<img src="https://github.com/pavanvenkatreddy/Wine_Quality_Preddiction_ECS_Docker/blob/main/Screenshot%202023-12-04%20at%201.41.54%20AM.png" width="500">

## Running the Prediction Application without Docker
To run the prediction application without docker, the following packages are needed
- [Pyspark](https://pypi.org/project/pyspark/)
- [JAVA JDK](https://www.oracle.com/java/technologies/javase-jdk13-downloads.html)
- [numpy](https://pypi.org/project/numpy/)
- [Apache Spark(spark-3.0.1-bin-hadoop2.7.tgz)](https://spark.apache.org/downloads.html)

### Java Installation
- Download Java JDK from here [link](https://www.oracle.com/java/technologies/javase-jdk13-downloads.html)
- Go to downloads folder and run the following command on console
```Console
 $ sudo dpkg -i jdk-13.0.2_linux-x64_bin.deb
```
- Run `java --version` on console to verify if java is installed.

- To setup environment variables for java, include the following in  `/etc/environemnt` file as shown below using any text editor (i.e nano, gedit)
```Console
 $ JAVA_HOME=/usr/lib/jvm/jdk-13.0.2
```
- Finally source the `/etc/environment` file, 

```Console
 $ source /etc/environment
```

### Installing Apache Spark
- Go to [Spark Website](https://spark.apache.org/downloads.html)
- Select Spark Release Version as 3.0.1 and download the .tgz file
 
- Go to the downloads folder and extract the tgz file using the following command
```Console
 $ sudo tar -xvzf spark-3.0.1-hadoop2.7.tgz 
```

- To setup environment variables for pyspark, include the following `~/.bashrc` file
```Console
 export SPARK_HOME=~/Downloads/spark-3.0.1-bin-hadoop2.7
 export PATH=$PATH:$SPARK_HOME/bin
 export PYTHONPATH=$SPARK_HOME/python:$PYTHONPATH
 export PYSPARK_PYTHON=python3
 export PATH=$PATH:$JAVA_HOME/jre/bin
```

- Finally source `./bashrc` file or restart `console` for the variables to get updated

```Console
 $ source ~./bashrc
 ```
- Run `pyspark` in console to verify the installation. 


### Running prediction application
- Make sure Modelfile is present before running the *wine_test_nodocker.py* file. In case it is not present run the *wine_train_nodocker.py* file to generate the same. This can be done by executing the following command
```Console
 $ python3 wine_train_nodocker.py
```
- Run the prediction app using this command
```Console
 $ python3 wine_test_nodocker.py TestDataset.csv
```
After the command is executed successfully, two files will be generated in the directory, `Results.txt` and `Resultdata` folder containing csv file.


