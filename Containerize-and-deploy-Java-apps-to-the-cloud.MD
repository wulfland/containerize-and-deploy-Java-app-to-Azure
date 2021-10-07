# Containerize and deploy Java apps to Azure

**This guide will walk through the steps needed to containerize a Java application and deploy to Azure cloud**

> 📝 Please Note you will need the following to complete this guide
> - Azure Subscription
> - GIT CLI and/or GitHub
> - Azure CLI
> - JAVA JDK 8+ 
> - Maven 
> - Docker Hub Login (To pull Maven & Tomcat base image if you do not already have them cached)

Containerizing Java applications has many benefits for  building & deploying applications at scale. In this guide we will look at the steps needed to containerize an existing Java application and deploy it to Azure Kubernetes Service in Azure cloud. You're welcome to use your own Java application, or follow along with one we have chosen. We've selected one from the wild, [TurkishAirlines](https://github.com/harismuneer/Flight-Booking-System-JavaServlets_App) A fully responsive web-based Flight Booking System for Turkish Airlines based on the Model View Controller (MVC) Architecture made using Java Servlets, Java Server Pages (JSPs). Since this application was originally built with Ant, we've decided to fork it and introduce Maven as an optional build implementation.

**Step 1**
Build your application

 - The Turkish Airlines Application was originally built with Ant but has since been forked and updated to optionally build with Maven.

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
At this point you can take this TurkishAirlines-0.0.1-SNAPSHOT.war and deploy it to your Application Server of choice and TurkishAirlines customers can start using all of the features. However, this type of deployment is cumbersome for many reasons such as orchestration elements like scale, hence the reason we are going to containerize this application and deploy to Azure cloud!

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
 >🛑 Be sure to substitute your own value for <youralias> because this must be globally unique. This should be lowercase letters and should not have special characters, if you do not enter a prefix for <youralias> you will get an error because javacontainerizationdemoaks is already taken.

 - From your CLI run `az aks get-credentials --resource-group "$AZ_RESOURCE_GROUP" --name <youralias>"$AZ_KUBERNETES_CLUSTER"`
   *- (This will download the cluster configuration information so you can manage your cluster from the Kubernetes web interface and  from `kubectl`)*

	```cli
	az aks get-credentials --resource-group "$AZ_RESOURCE_GROUP" --name javacontainerizationdemoaks
	Merged "javacontainerizationdemoaks" as current context in /home/chtrembl/.kube/config
	```
		 
**Step 4**
Containerize your application with a Dockerfile.

With containers, we will be sharing the host machines kernel and operating system. We will be using the Docker runtime to to build a Docker image that will deploy to the host operating system. In this guide, we will be building and deploying this Docker image to a worker node in the newly created Azure Kubernetes Cluster.

![enter image description here](https://github.com/chtrembl/Flight-Booking-System-JavaServlets_App/blob/master/Images/docker.png?raw=true)

As seen in the image above, we will be using Docker to:

 - **Build**
	 - Build our Docker image (layers needed for Docker to give us a running container from our image).
	 
 - **Run**
 
	 - Running instance of our Docker image is a container, all of the layers needed for us to access the Turkish Airlines application.
	 
 - **Push**
	 - Azure Container Registry will store our images so they are readily available and network close when we need to scale on Azure.
	 
 - **Pull**
	 - Containers are initialized from images, they are pulled from Azure Container Registry and this is where Azure Kubernetes will pull from.

Before asking Docker to build an image for us, we need a Dockerfile. (we have created one for Turkish Airlines) If you inspect the [Dockerfile](https://raw.githubusercontent.com/chtrembl/Flight-Booking-System-JavaServlets_App/master/Project/TurkishAirlines/Dockerfile) you will see the following:

```docker
#BUILD Docker image
FROM tomcat
COPY tomcat-users.xml /usr/local/tomcat/conf
ADD target/*.war /usr/local/tomcat/webapps/TurkishAirlines.war
EXPOSE 8080

CMD ["catalina.sh", "run"]
```
This is a single stage Dockerfile to build the Turkish Airlines Docker image. The idea here is to leverage the concept of Docker layers (each line of code is an Docker instruction which in turn is a layer) for efficiency such as caching (each time you execute a build, compute resources are being allocated, and both cost and developer productivity savings can be associated. If layers can be reused then the layer caching benefits can be useful.

This stage starts with the base Tomcat image layer and will be the base layer to our Docker image and we will build on top of that. Once the Tomcat layer has been applied, the tomcat-users.xml is moved to the Tomcat conf (so Turkish Airlines customers can authenticate) followed by the TurkishAirlines-0.0.1-SNAPSHOT.war being moved to the Tomcat webapps directory. Port 8080 needs to be exposed for ingress to the container and last but not least Docker will execute catalina.sh via the run command.

> 📝 Please Note the following, without a version tag, the latest will be applied. Generally you will want to leverage a version tag (remember, caching is applied, so if layers are consistently changing, you will incur bandwidth, latency, compute time and/or side effect of untested builds/layers.) For the sake of this tutorial, no tagging will be used, the latest Tomcat image will be used (they are not changing enough to warrant a specific stable version). 

**Step 5**
Build a Docker image

Once you have a Dockerfile, you can build a Docker image from it.

 - From your CLI run `docker build -t turkishairlines .`
*- (Docker will execute each line instruction and build a layer, and the end result will be your Docker image)*

> 📝 Please Note the following, TurkishAirlines-0.0.1-SNAPSHOT.war is required to be built in advance as seen in **Step 1.** 

```cli
docker build -t turkishairlines .
Sending build context to Docker daemon  115.7MB
Step 1/5 : FROM tomcat
 ---> 921ef208ab56
Step 2/5 : COPY tomcat-users.xml /usr/local/tomcat/conf
 ---> Using cache
 ---> 138158e2cf0b
Step 3/5 : ADD target/*.war /usr/local/tomcat/webapps/TurkishAirlines.war
 ---> 98e0f3f487a8
Step 4/5 : EXPOSE 8080
 ---> Running in c24b57b9f62e
Removing intermediate container c24b57b9f62e
 ---> bf7fa6cc15a8
Step 5/5 : CMD ["catalina.sh", "run"]
 ---> Running in 78835beea340
Removing intermediate container 78835beea340
 ---> cda4f5b459f1
Successfully built cda4f5b459f1
Successfully tagged turkishairlines:latest
```

 - From your CLI run `docker image ls`
*- (This will list all of you Docker images)*

```cli
docker image ls
REPOSITORY                                        TAG                 IMAGE ID            CREATED             SIZE
turkishairlines                                   latest              cda4f5b459f1        About an hour ago   692MB
```

**Step 6**
Run your Docker image

Once you have a Docker image, you can run it via Docker, the running instance of this image will be your **Container**

 - From your CLI run `docker run -p 8080:8080 turkishairlines`
*- (Docker will startup a Container)*
```cli
docker run -p 8084:8080 turkishairlines
NOTE: Picked up JDK_JAVA_OPTIONS:  --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.util.concurrent=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
02-Aug-2021 20:50:22.682 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version name:   Apache Tomcat/9.0.50
02-Aug-2021 20:50:22.687 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server built:          Jun 28 2021 08:46:44 UTC
02-Aug-2021 20:50:22.687 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Server version number: 9.0.50.0
02-Aug-2021 20:50:22.688 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Name:               Linux
02-Aug-2021 20:50:22.688 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log OS Version:            5.10.16.3-microsoft-standard-WSL2
02-Aug-2021 20:50:22.689 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Architecture:          amd64
02-Aug-2021 20:50:22.690 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Java Home:             /usr/local/openjdk-11
02-Aug-2021 20:50:22.694 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Version:           11.0.12+7
02-Aug-2021 20:50:22.694 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log JVM Vendor:            Oracle Corporation
02-Aug-2021 20:50:22.695 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_BASE:         /usr/local/tomcat
02-Aug-2021 20:50:22.696 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log CATALINA_HOME:         /usr/local/tomcat
02-Aug-2021 20:50:22.729 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.lang=ALL-UNNAMED
02-Aug-2021 20:50:22.730 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.io=ALL-UNNAMED
02-Aug-2021 20:50:22.730 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util=ALL-UNNAMED
02-Aug-2021 20:50:22.730 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.base/java.util.concurrent=ALL-UNNAMED
02-Aug-2021 20:50:22.731 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED
02-Aug-2021 20:50:22.731 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.config.file=/usr/local/tomcat/conf/logging.properties
02-Aug-2021 20:50:22.733 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
02-Aug-2021 20:50:22.735 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djdk.tls.ephemeralDHKeySize=2048
02-Aug-2021 20:50:22.735 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.protocol.handler.pkgs=org.apache.catalina.webresources
02-Aug-2021 20:50:22.735 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dorg.apache.catalina.security.SecurityListener.UMASK=0027
02-Aug-2021 20:50:22.735 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dignore.endorsed.dirs=
02-Aug-2021 20:50:22.735 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.base=/usr/local/tomcat
02-Aug-2021 20:50:22.736 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Dcatalina.home=/usr/local/tomcat
02-Aug-2021 20:50:22.736 INFO [main] org.apache.catalina.startup.VersionLoggerListener.log Command line argument: -Djava.io.tmpdir=/usr/local/tomcat/temp
02-Aug-2021 20:50:22.762 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent Loaded Apache Tomcat Native library [1.2.30] using APR version [1.6.5].
02-Aug-2021 20:50:22.763 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR capabilities: IPv6 [true], sendfile [true], accept filters [false], random [true], UDS [true].
02-Aug-2021 20:50:22.763 INFO [main] org.apache.catalina.core.AprLifecycleListener.lifecycleEvent APR/OpenSSL configuration: useAprConnector [false], useOpenSSL [true]
02-Aug-2021 20:50:22.776 INFO [main] org.apache.catalina.core.AprLifecycleListener.initializeSSL OpenSSL successfully initialized [OpenSSL 1.1.1d  10 Sep 2019]
02-Aug-2021 20:50:23.686 INFO [main] org.apache.coyote.AbstractProtocol.init Initializing ProtocolHandler ["http-nio-8080"]
02-Aug-2021 20:50:23.777 INFO [main] org.apache.catalina.startup.Catalina.load Server initialization in [1697] milliseconds
02-Aug-2021 20:50:23.977 INFO [main] org.apache.catalina.core.StandardService.startInternal Starting service [Catalina]
02-Aug-2021 20:50:23.978 INFO [main] org.apache.catalina.core.StandardEngine.startInternal Starting Servlet engine: [Apache Tomcat/9.0.50]
02-Aug-2021 20:50:24.039 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deploying web application archive [/usr/local/tomcat/webapps/TurkishAirlines.war]
02-Aug-2021 20:50:27.164 INFO [main] org.apache.jasper.servlet.TldScanner.scanJars At least one JAR was scanned for TLDs yet contained no TLDs. Enable debug logging for this logger for a complete list of JARs that were scanned but no TLDs were found in them. Skipping unneeded JARs during scanning can improve startup time and JSP compilation time.
02-Aug-2021 20:50:30.887 INFO [main] com.sun.xml.ws.server.MonitorBase.createRoot Metro monitoring rootname successfully set to: com.sun.metro:pp=/,type=WSEndpoint,name=/TurkishAirlines-PriceAndSeats-PriceAndSeatsPort
02-Aug-2021 20:50:31.151 INFO [main] com.sun.xml.ws.transport.http.servlet.WSServletDelegate.<init> WSSERVLET14: JAX-WS servlet initializing
02-Aug-2021 20:50:32.662 INFO [main] com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized WSSERVLET12: JAX-WS context listener initializing
02-Aug-2021 20:50:32.663 INFO [main] com.sun.xml.ws.transport.http.servlet.WSServletContextListener.contextInitialized WSSERVLET12: JAX-WS context listener initializing
02-Aug-2021 20:50:32.735 INFO [main] org.apache.catalina.startup.HostConfig.deployWAR Deployment of web application archive [/usr/local/tomcat/webapps/TurkishAirlines.war] has finished in [8,695] ms
02-Aug-2021 20:50:32.746 INFO [main] org.apache.coyote.AbstractProtocol.start Starting ProtocolHandler ["http-nio-8080"]
02-Aug-2021 20:50:32.768 INFO [main] org.apache.catalina.startup.Catalina.start Server startup in [8990] milliseconds
```
Login to Turkish Airlines http://localhost:8080/TurkishAirlines with a customer from [tomcat-users.xml](https://github.com/chtrembl/Flight-Booking-System-JavaServlets_App/blob/master/Project/TurkishAirlines/tomcat-users.xml) and start exploring!

**username: shariq@customer.com
password:  c**

![enter image description here](https://github.com/chtrembl/Flight-Booking-System-JavaServlets_App/blob/master/Images/main.png?raw=true)

 - From your CLI run `ctrl- c`
*- (This will shutdown and destroy the Container)*

**Step 6**
Publish and deploy your Docker image

You can push your Docker image(s) to the Azure Container Registry. By doing so, your image(s) will be network close to all of your Azure resources, such as your Azure Kubernetes Cluster. We will ultimately configure AKS to pull the TurkishAirlines image from Azure Container Registry.

 - From your CLI run `az acr build --registry "$AZ_CONTAINER_REGISTRY" --resource-group "$AZ_RESOURCE_GROUP" --image turkishairlines:latest .`
	 *- (This will once again build a TurkishAirlines Docker image, however this time it will also **PUSH** it into Azure Container Registry*

 - From your CLI run `az acr repository show -n "$AZ_CONTAINER_REGISTRY" --image turkishairlines:latest`
	 *- (This will show the details of the TurkishAirlines Docker image that was just pushed to Azure Container Registry*

```cli
az acr repository show -n "$AZ_CONTAINER_REGISTRY" --image turkishairlines:latest
{
  "changeableAttributes": {
    "deleteEnabled": true,
    "listEnabled": true,
    "readEnabled": true,
    "writeEnabled": true
  },
  "createdTime": "2021-08-02T22:31:47.9603181Z",
  "digest": "sha256:cda94ea9c3cf89032cb2cdbe298b003061734550fc877492c6aff12ee675dc5d",
  "lastUpdateTime": "2021-08-02T22:31:47.9603181Z",
  "name": "latest",
  "signed": false
}
```
Let's deploy this TurkishAirlines Docker image to your AzureKubernetesCluster. 

 - From your CLI run `vi deployment.yml` or open this [deployment.yml](https://raw.githubusercontent.com/chtrembl/Flight-Booking-System-JavaServlets_App/master/Project/TurkishAirlines/deployment.yml) in the editor of your choice.
	 *- (This is the Kubernetes manifest that informs Azure Kubernetes Service of the desired state you wish to run in. It contains things like what Docker image to use when creating containers and how to ingress into your cluser.*

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: turkishairlines
spec:
  replicas: 1
  selector:
    matchLabels:
      app: turkishairlines
  template:
    metadata:
      labels:
  app: turkishairlines
    spec:
      nodeSelector:
  "beta.kubernetes.io/os": linux
      containers:
      - name: turkishairlines
  image: javacontainerizationdemoacr.azurecr.io/turkishairlines:latest
  resources:
    requests:
      cpu: 100m
      memory: 128Mi
    limits:
      cpu: 250m
      memory: 256Mi
  ports:
  - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: turkishairlines
spec:
  type: LoadBalancer
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: turkishairlines
```
> 📝 Please Note the following, if your deploying turkishairlines, all you will need to update in this file is the image: line. Ensure image: is pointing to your Azure Container Registry that you specified earlier, most likely image: **<youralias>javacontainerizationdemoacr.azurecr.io/turkishairlines:latest**

Save the updates to deployment.yml

 - From your CLI run `kubectl apply -f deployment.yml`
 	 *- (This is the Kubernetes manifest that informs Azure Kubernetes Service of the desired state you wish to run in. It contains things like what Docker image to use when creating containers and how to ingress into your cluster.)*

```cli
kubectl apply -f deployment.yml
deployment.apps/turkishairlines created
service/turkishairlines created
```
```cli
kubectl get all
NAME                                   READY   STATUS              RESTARTS   AGE
pod/turkishairlines-75647c4c98-9v7vk   0/1     ContainerCreating   0          13s

NAME                      TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)          AGE
service/kubernetes        ClusterIP      10.0.0.1      <none>        443/TCP          7h28m
service/turkishairlines   LoadBalancer   10.0.34.128   <pending>     8080:30265/TCP   13s

NAME                              READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/turkishairlines   0/1     1            0           13s

NAME                                         DESIRED   CURRENT   READY   AGE
replicaset.apps/turkishairlines-75647c4c98   1         1         0       13s
```

The CLUSTER-IP LoadBalancer IP Address is how you will access Turkish Airlines

Things you can now do now with this guide

☑️ How to containerize a Java application and deploy to Azure cloud

