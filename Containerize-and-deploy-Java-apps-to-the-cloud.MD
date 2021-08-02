# Containerize and deploy Java apps to cloud

**This guide will walk through the steps needed to containerize a Java application and deploy to the cloud**

> 📝 Please Note you will need the following to complete this guide
> - Azure Subscription
> - GIT CLI and/or GitHub
> - Azure CLI
> - JAVA JDK 8+ 
> - Maven (Optional if you want to build the application outside of Docker)
> - Docker Hub Login (To pull Maven & Tomcat base image if you do not already have them cached)

Containerizing Java applications is simple and the modern way of building & deploying applications at scale. In this guide we will look at the steps needed to containerize and existing Java application and deploy it to the cloud. You're welcome to use your own Java application, or following along with one we have chosen. We've selected one from the wild, [TurkishAirlines](https://github.com/harismuneer/Flight-Booking-System-JavaServlets_App) A fully responsive web-based Flight Booking System for Turkish Airlines based on the Model View Controller (MVC) Architecture made using Java Servlets, Java Server Pages (JSPs). Since this application was originally built with Ant, we've decided to fork it and introduce Maven as an optional build implementation.

**Step 1**
Build your application

 - The Turkish Airlines Application was originally built with Ant but has since been forked and updated to optionally build with Maven, 

 - From your CLI run `gh repo clone chtrembl/Flight-Booking-System-JavaServlets_App`
	 *- (This will clone https://github.com/chtrembl/Flight-Booking-System-JavaServlets_App)*
 
 - From your CLI cd to `Flight-Booking-System-JavaServlets_App`
 
 - From your CLI run `cd Project/TurkishAirlines`
*- (You should now be in the TurkishAirlines project root, for example /mnt/c/Users/chtrembl/dev/git/Flight-Booking-System-JavaServlets_App/Project/TurkishAirlines)*

 - From your CLI run `mvn clean install`
*- (Maven should build your application and upon success you will see the following artifact TurkishAirlines-0.0.1-SNAPSHOT.war )*

	```cli
	[INFO] Building war: /mnt/c/Users/chtrembl/dev/git/Flight-Booking-System-JavaServlets_App/Project/TurkishAirlines/target/TurkishAirlines-0.0.1-SNAPSHOT.war
	[INFO] ------------------------------------------------------------------------
	[INFO] BUILD SUCCESS
	[INFO] ------------------------------------------------------------------------
	[INFO] Total time:  17.698 s
	[INFO] Finished at: 2021-08-02T15:18:07-04:00
	[INFO] ------------------------------------------------------------------------
	```
At this point you can take this TurkishAirlines-0.0.1-SNAPSHOT.war and deploy it to your Application Server of choice and TurkishAirlines customers can start using all of the features. However, this type of deployment is cumbersome for many reasons such as all of the orchestration elements such as scale, hence the reason we are going to containerize this application and deploy to Azure cloud!

**Step 2**
Get your environment ready

 - From your CLI run `az login`
	 *- (This will authenticate you into Azure ARM)*
	 
 - From your CLI run `az account list --output table`
	 *- (This will list your Azure subscriptions, make note of the subscription you want to use for this guide)*
	 
 - From your CLI run `az account set --subscription  "Your subscription name"`
	 *- (This will set the Azure subscription that will hold the resources needed for your Java application deployment)*

**Step 3**
Provision Azure Resources

 - From your CLI run `AZ_RESOURCE_GROUP=javacontainerizationdemorg`
	 *- (This will set the Azure resource group variable that will be used to specify the Azure Resource Group. You can use whatever resource group you prefer, this guide will assume you have chosen javacontainerizationdemorg)*

 - From your CLI run `AZ_CONTAINER_REGISTRY=javacontainerizationdemoacr`
	 *- (This will set the Azure Container Registry variable that will be used to specify the Azure Container Registry. You can use whatever Azure Container Registry value you prefer, this guide will assume you have chosen javacontainerizationdemoacr)*

 - From your CLI run `AZ_KUBERNETES_CLUSTER=javacontainerizationdemoaks`
	 *- (This will set the Azure Kubernetes Cluster variable that will be used to specify the Azure Kubernetes Cluster Configuration. You can use whatever Azure Kubernetes Cluster value you prefer, this guide will assume you have chosen javacontainerizationdemoaks)*

 - From your CLI run `az group create --name="$AZ_RESOURCE_GROUP" --location=eastus`
	 *- (This will provision the resource group. You can use whatever location you prefer)*
	 
 - From your CLI run `az acr create --resource-group "$AZ_RESOURCE_GROUP" --name "$AZ_CONTAINER_REGISTRY" --sku Basic`
	 *- (This will provision the Azure Container Registry. This resource will be used to retain the Docker images containing the Turkish Airlines Java application we build below. )*

 - From your CLI run `az configure --defaults acr="$AZ_CONTAINER_REGISTRY"`
	 *- (This will set the newly created Azure Container Registry as the default for this session. )*
	 
 - From your CLI run `az acr login -n "$AZ_CONTAINER_REGISTRY"`
	 *- (This will authenticate to the newly created Azure Container Registry. )*
	 
 - From your CLI run `az aks create --resource-group="$AZ_RESOURCE_GROUP" --name="$AZ_KUBERNETES_CLUSTER" --attach-acr "$AZ_CONTAINER_REGISTRY" --dns-name-prefix=<youralias>"$AZ_KUBERNETES_CLUSTER" --generate-ssh-keys`
   *- (This will create an Azure Kubernetes Cluster that we will be deploying the Java application (Docker image) to. )*

> 📝 Please Note the following, this step will take some time (20 mins or so) to complete while Azure provisions resources for you.
> 
 >🛑 Be sure to substitute your own value for <youralias> because this must be globally unique. This should container lowercase letters and should not have special characters.
	 
**Step 4**
Containerize your application

With containers, we will be sharing the host machines kernel and operating system. We will be using the Docker runtime to to build a Docker image that will deploy to the host operating system. In this guide, we will be building and deploying this Docker image to a worker node in the newly created Azure Kubernetes Cluster.

With Docker we will be building Docker

**Step 5**
Build a Docker image

**Step 6**
Publish and deploy your Docker image

Things you can now do now with this guide

☑️ How to containerize and containerize a Java application and deploy to the cloud

