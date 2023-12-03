# Wine_Quality_Preddiction_ECS_Docker

This guide provides a detailed explanation of the process for utilizing AWS services to train a Machine Learning (ML) model on multiple parallel EC2 instances within the Elastic Compute Cloud. The ML program is developed in Python language, incorporating the [Apache Spark MLlib libraries] (https://spark.apache.org/mllib/). Both the training and prediction programs are configured to execute within a container.

Github link - <  >
*Docker Image for Training*: [pavanvenkatbhumula/wine:v1] (https://hub.docker.com/repository/docker/pavanvenkatbhumula/wine/general)

*Docker Image for Testing*: [pavanvenkatbhumula/winetest:v1] (https://hub.docker.com/repository/docker/pavanvenkatbhumula/winetest/general)

To run the ML Image application for training on multiple parallel EC2 instances, a cluster needs to be set up in ECS. The steps given below are followed to create a cluster with 4 instances.
In AWS Management Console search for **Elastic Container Service (ECS)**
  __In ECS Console, select **Cluster** and click on *"Create Cluster"*
  __*Cluster name*__ : *winequality* ~your desired name
  __*Provisioning model*__ : *On-Demand Instance*
  __*EC2 instance type*__ : *t2.micro*
  __*Number of instances*__ : *4*
  __*Key Pair*__ : *Choose an appropriate key pair*
  __*Security group inbound rules (Port range)*__ : *22-80* 
- Click on *"Create"*  to create a cluster 
