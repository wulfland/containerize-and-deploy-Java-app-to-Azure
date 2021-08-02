# Containerize and deploy Java apps to cloud

**This guide will walk through the steps needed to containerize a Java application and deploy to the cloud**

> 📝 Please Note you will need the following to complete this guide
> - Azure Subscription
> - GIT CLI and/or GitHub
> - Azure CLI
> - JAVA JDK 8+ 
> - Maven (Optional if you want to build the application outside of Docker)
> - Docker Hub Login (To pull Maven & Tomcat base image if you do not already have them cached)

Containerizing Java applications is simple and the modern way of building & deploying applications at scale. In this guide we will look at the steps needed to containerize an existing Java application and deploy it to the cloud. You're welcome to use your own Java application, or following along with one we have chosen. We've selected one from the wild, [TurkishAirlines](https://github.com/harismuneer/Flight-Booking-System-JavaServlets_App) A fully responsive web-based Flight Booking System for Turkish Airlines based on the Model View Controller (MVC) Architecture made using Java Servlets, Java Server Pages (JSPs). Since this application was originally built with Ant, we've decided to fork it and introduce Maven as an optional build implementation.

**Step 1**
Get your local environment ready

 - Fork or clone https://github.com/chtrembl/Flight-Booking-System-JavaServlets_App
 
 - From your CLI run `az login`
	 *- (This will authenticate you into Azure ARM)*
	 
 - From your CLI run `az account list --output table`
	 *- (This will list your Azure subscriptions, make note of the subscription you want to use for this guide)*
	 
 - From your CLI run `az account set --subscription  "Your subscription name"`
	 *- (This will set the Azure subscription that will hold the resources needed for your Java application deployment)*

**Step 2**
Provision Azure Resources

 - From your CLI run `AZ_RESOURCE_GROUP=javacontainerizationdemorg`
	 *- (This will set the Azure resource group variable that will be used to specify the Azure Resource Group. You can use whatever resource group you prefer, this guide will assume you have chosen javacontainerizationdemorg)*

 - From your CLI run `AZ_CONTAINER_REGISTRY=javacontainerizationdemoacr`
	 *- (This will set the Azure Container Registry variable that will be used to specify the Azure Container Registry. You can use whatever Azure Container Registry value you prefer, this guide will assume you have chosen javacontainerizationdemoacr)*

 - From your CLI run `az group create --name="$AZ_RESOURCE_GROUP" --location=eastus`
	 *- (This will provision the resource group. You can use whatever location you prefer)*
	 
 - From your CLI run `az acr create --resource-group "$AZ_RESOURCE_GROUP" --name "$AZ_CONTAINER_REGISTRY" --sku Basic`
	 *- (This will provision the Azure Container Registry. This resource will be used to retain the Docker images containing the Turkish Airlines Java application we build below. )*

 - From your CLI run `az configure --defaults acr="$AZ_CONTAINER_REGISTRY"`
	 *- (This will set the newly created Azure Container Registry as the default for this session. )*
	 
 - From your CLI run `az acr login -n "$AZ_CONTAINER_REGISTRY"`
	 *- (This will authenticate to the newly created Azure Container Registry. )*
	 
Things you can now do now with this guide

☑️ How to containerize and containerize a Java application and deploy to the cloud

