---
layout: single
title: Azure - Appservice
excerpt: "We are going to learn how the app service works and the possibilities it offers us to manage the hosting since the app service is still a hosting but the difference is that we can manage it."
date: 2021-10-24
classes: wide
header:
  teaser: /assets/images/img-appservice/appservice.png 
  teaser_home_page: true
  icon: /assets/images/img-appservice/azure.png
categories:
  - Cloud
  - web
tags:
  - Azure
  - App service
  - git
  - github
  - Azure devops
  - Docker
  - webhook
---

<p align = "center">
<img src = "/assets/images/img-appservice/appservice.png">
</p>

We are going to learn how the app service works and the possibilities it offers us to manage the hosting since the app service is still a hosting but the difference is that we can manage it.

In this article we are going to learn:

+ What is app service and azure concepts
+ How to create a resource group
+ Create an App service plan
+ Creat a appservice in azure
+ Deploy our web app in the multiple ways:
	- Deploying with ftp
	- Deploying with our github repository
	- Deploying with bitbuket
	- Deploying with git local
	- Deploying with azure repo
	- Deploying with vs code 2019
		* Push our repo in github in vs code 2019
	- Deploying with vs code
+ Create appservice with azure start devops
+ Some azure CLI commands 
+ Create app service and deploy our web app in docker container
	- Create webhook 

## What is app service?

App service is a fully managed web hosting service for building web applications, services and RESTful APIs. The service offers a variety of plans to meet the needs of any application, from small web sites to worldwide web applications.

<p align = "center">
<img src = "/assets/images/img-appservice/azure-app-service-diagram.png">
</p>

In azure there are three models of cloud computing offered to us as a service, and each one covers some degree of management and this includes other cloud providers, which is the iaaS, PaaS and SaaS.

***iaaS***: this service manage virtual machines and storage in the cloud.

***PaaS***: It provides operating systems, databases, etc. that you can raise, modify and shut down with a few clicks.

***SaaS***: Allows users to connect to applications that are in the cloud over the Internet and use them (hosting, outlook, google docs). 

<p align = "center">
<img src = "/assets/images/img-appservice/azuremodels.png">
</p>

There is one important thing we have to keep in mind when using azure is that it charges us for reading data and this includes with other providers.

## How to create resource group

A ***resource group*** allows us to manage resources in a flexible and simple way (vm, web app, databases, networks, etc.) The resource group can include all the resources of the solution or only those that you want to manage as group. 

<p align = "center">
<img src = "/assets/images/img-appservice/group.png">
</p>

To create a group of resources, we go to the azure portal and click on the three stripes that appear in the upper left. And then we select a resource group and click on "create" .

<p align = "center">
<img src = "/assets/images/img-appservice/captura1.png">
</p>

In case we have other types of azure subscriptions we will select it, and we gon a put name our resource group. It is important to check which region we choose, because depending on the region, there are different costs. And click on "review + finish".

<p align = "center">
<img src = "/assets/images/img-appservice/captura2.png">
</p>

## Create an app service plan

We are going to create a plan for our app service, and when we are going to create a new app service we can select this plan. 

So to create a app service plan, we are going to search "app service plans" in the azure portal navegation bar. Here we are gon a click in "create".

<p align = "center">
<img src = "/assets/images/img-appservice/captura3.png">
</p>

Here we are going to select our resource group that we have just created, we put the name of our plan, we will choose an operating system this is important since each operating system offers us different functionalities in the app service, we choose a region (it is important to select the region where we are located) and finally we will choose the price plan, there are two price plans that are for testing, which is f1 (which is free) and b2 (which is paid and with more capacity) there are other price plans but we'll see later. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura4.png">
</p>

After selecting the sku, we click on "create and review" and a summary of the configuration that we have put in our app service plan will appear. and we click "create" to finish. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura5.png">
</p>

## Create an app service

We will type "app service" in the search engine of the azure portal and click on app service. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura6.png">
</p>

Here in the app service panel we are going to click on "create". 

<p align = "center">
<img src = "/assets/images/img-appservice/captura7.png">
</p>

First we are going to select the group of resources that we have created previously, and then we will put a domain name for our web app. In "publish" we are going to select code, since the second option is if we want our app service to be published within a docker container, but we will see that later.

Where it says "runtime stack" defines the technology stack used to develop the application, in my case I will choose php 7.3. We choose an OS as I said before, it is important that we select a suitable one for our needs since each OS offers different features in app service. 

We choose a region closer to where we are located and in "plan" we will choose the plan that we created previously, in this case I put another app service plan since the plan that I created previously put the Linux OS. or if we want we can create a plan directly from here by clicking on "creat new". And to finish we click on "review and finish".  

<p align = "center">
<img src = "/assets/images/img-appservice/captura8.png">
</p>

And here we can see the summary of the configuration that we put to create the app service (if we see any wrong configuration we can go back) and we click on "create" to finish the process. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura9.png">
</p>

We are going to wait for the implementation process of our app service to finish. And once it is finished we click on the notification bell and we click on "go to resource" or from the same panel where it is being deployed.

<p align = "center">
<img src = "/assets/images/img-appservice/captura10.png">
</p>

We are in the overview section of our app service. And we can see we have the domain that we put for our app service, in case if we click on the url the default azure page will appear. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura11.png">
</p>

If we go to "Scale up" here we can change our contract plan for our app service.

<p align = "center">
<img src = "/assets/images/img-appservice/captura12.png">
</p>

If we want to upload our app service in production we have other types of plans.

<p align = "center">
<img src = "/assets/images/img-appservice/captura13.png">
</p>

If we go to the configuration of our app service, in "general settings" we can change some settings of our app service in the event that when creating the app service we put some configuration that we didn't want.

<p align = "center">
<img src = "/assets/images/img-appservice/captura14.png">
</p>

## Deploy our web app in the multiple ways: 

### Deploying with ftp:

In this case, to deploy our web app we are going to use an ftp client, in this case I am going to use filezilla. but if we want we can use the terminal. 

In my case I am going to upload an html file that I just created. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura15.png">
</p>

What we are going to do is connect the server to our ftp client. If in your case you are using filezilla, what we are going to do is go to "archives" and click on "site manager". and we click on "new site" and in this case we are going to put the name of our app service that we just created. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura16.png">
</p>

In the ftp client it will ask us for the name of our server, username and password. for this we go to our app service and we will go where it says "Deployment center" and click on "ftps credentials". 

<p align = "center">
<img src = "/assets/images/img-appservice/captura17.png">
</p>

In the filezilla will are going to copy and paste "ftps endpoint", copy the username and password and paste in filezilla. and click in "connect".

<p align = "center">
<img src = "/assets/images/img-appservice/captura18.png">
</p>

If everyting is all right we should get a popup of the ftp certificate that we are using, when we click in "connect". And click "accept".

<p align = "center">
<img src = "/assets/images/img-appservice/captura19.png">
</p>
 
To deploy our web app just where it says "remote site" we go to the path "/site/wwwroot" and here we drag the folder where we have developed our web app. in my case I am just going to upload the html file that I created before.

where it says "status" we can see if our files have been uploaded correctly to the server. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura20.png">
</p>

If I go to the domain of my app service we can see that the deploy has been done correctly.

<p align = "center">
<img src = "/assets/images/img-appservice/captura21.png">
</p>

### Deploying with our github repository:


In my case i have created an repository in my github account, and in that repository i add an html file.

<p align = "center">
<img src = "/assets/images/img-appservice/captura22.png">
</p>
 
Now what we are going to do is to connect our github account and the repository. for this I am going to use the app service that I have created in azure CLI. To do this we go to our app service, and in "deployment center" -> "settings" -> "source" -> "continuous deployment" we select github.

<p align = "center">
<img src = "/assets/images/img-appservice/captura23.png">
</p>

Then a popup will appear and we will log in to our github account (in case we are not logged in) and click on "Authorize app service". And we will have already connected our github account to our app service.

<p align = "center">
<img src = "/assets/images/img-appservice/captura24.png">
</p>

In organization we are going to select an organization that we have created in github, in my case I don't have one so I am going to select the name of my account. In "repository" we are going to select the repo where we have uploaded our web app. And select the branch we want, in this case I have selected the "main" branch. And finally we click on "save".

<p align = "center">
<img src = "/assets/images/img-appservice/captura25.png">
</p>

And as we can see, it has been deployed correctly.

<p align = "center">
<img src = "/assets/images/img-appservice/captura26.png">
</p>

If in case we want another repo in our app service or disconnect our repo. We go to "deployment center" and click on "disconnect".

<p align = "center">
<img src = "/assets/images/img-appservice/captura27.png">
</p>

If we go to "deployment center" it notifies us and tells us the date and time and the user who made a pull request or any modification in the repository (it's the same feature we have in github).

<p align = "center">
<img src = "/assets/images/img-appservice/registro.png">
</p>


### Deploying with bitbucket:

In bitbucket it is the same process we have done in github to deploy our web app.

The first thing I have done in bitbucket is create a workspace. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura28.png">
</p>

Then i create a project.

<p align = "center">
<img src = "/assets/images/img-appservice/captura29.png">
</p>

And then i create a repository in bitbucket.

<p align = "center">
<img src = "/assets/images/img-appservice/captura30.png">
</p>

Inside the repository I have created an index.html file with the following content. 

<p align = "center">
<img src = "/assets/images/img-appservice/captura31.png">
</p>

Once I finished I made a commit of the changes.

<p align = "center">
<img src = "/assets/images/img-appservice/captura32.png">
</p>

In this case I have created another app service for this time.

<p align = "center">
<img src = "/assets/images/img-appservice/captura33.png">
</p>

To deploy our web app in the app service we go to "deployment center" and in "source" we choose "bitbucket" (we can use the app service that we have created before and disconnect it and instead of github put bitbucket). we put the organization that we have created in bitbucket, in my case I have created one called "testazure9", we select the repository and the branch, and we click on save.

<p align = "center">
<img src = "/assets/images/img-appservice/captura34.png">
</p>

And as we can see that the deploy has been done correctly.

<p align = "center">
<img src = "/assets/images/img-appservice/captura35.png">
</p>

### Deploying with Azure repo:

To deploy our web app in azure repos is the same process we did in bitbucket and github.

The first thing we will do is go to the azure repo page and click on "start" to configure azure repo or just go to the azure portal and search for "azure devops".

<p align = "center">
<img src = "/assets/images/img-appservice/captura36.png">
</p>

Now we are going to name our project, choose whether we want it to be private or public and select the region. And click on "continue".

<p align = "center">
<img src = "/assets/images/img-appservice/captura37.png">
</p>

And now put the project name and clic in "create project"

<p align = "center">
<img src = "/assets/images/img-appservice/captura38.png">
</p>

And as we can see we have already created a workspace and an organization. Now let's create a repository.

<p align = "center">
<img src = "/assets/images/img-appservice/captura39.png">
</p>

To create the repository we click on the arrow indicated in the previous image and click on "new repository", here we select git and put a name for the repository.

<p align = "center">
<img src = "/assets/images/img-appservice/captura40.png">
</p>

Inside the repository I have added an index.html file with the following content:

<p align = "center">
<img src = "/assets/images/img-appservice/captura41.png">
</p>

And click on "commit" to commit the changes we have made.

<p align = "center">
<img src = "/assets/images/img-appservice/captura42.png">
</p>

In my case I will create another app service to deploy (as I said before, we can go to the app service we have created and in "deployment centre" we disconnect the previous repo and in "source" we can select "Azure repo").

<p align = "center">
<img src = "/assets/images/img-appservice/captura43.png">
</p>

To deploy we go to "deployment center" -> "settings" in source we select "Azure repo" and in the "organization" select the organization that you have, and select the project we just created and the repo where we have our web app. And click on "save".

<p align = "center">
<img src = "/assets/images/img-appservice/captura44.png">
</p>

And we can see that the deploy has been done correctly and the changes have been applied correctly.

<p align = "center">
<img src = "/assets/images/img-appservice/captura45.png">
</p>

### Deploying with git local:

I have created another app service for this demonstration (but as I said before we can go to "deployment center" and disconnect and in "source" change the implementation).

<p align = "center">
<img src = "/assets/images/img-appservice/captura46.png">
</p>

Once we have selected "git local" we click on "save". because this time it won't ask for the repository and branch. Copy the link to clone the repository.

<p align = "center">
<img src = "/assets/images/img-appservice/captura47.png">
</p>

When cloning the repository we will be asked for the username and password, the same will happen if we want to push the repo.

<p align = "center">
<img src = "/assets/images/img-appservice/captura49.png">
</p>

To find the username and password, in the same "deployment center" in the section where it says "Local GIT or FTP credentials", copy the username and password and paste it to clone the repo.

<p align = "center">
<img src = "/assets/images/img-appservice/captura48.png">
</p>

Obviously when cloning the repo it tells us that there is no file as we have just created the app service and deployed it.

<p align = "center">
<img src = "/assets/images/img-appservice/captura50.png">
</p>

In my case I have added an html file in the cloned folder with the following content.

<p align = "center">
<img src = "/assets/images/img-appservice/captura51.png">
</p>

we push the repository with the link that appears in the "deployment centre".

<p align = "center">
<img src = "/assets/images/img-appservice/captura52.png">
</p>

as we can see that the deploy has been successful and the changes have been applied, when we have done the push.

<p align = "center">
<img src = "/assets/images/img-appservice/captura53.png">
</p>

### Deploying with vs code 2019:

In this case the IDE I am using is the community version, but this applied to the pro version.

We open the vs elbow 2019 and we will create a project.

<p align = "center">
<img src = "/assets/images/img-appservice/captura54.png">
</p>

Now we are going to choose one of the extensions we want, in this case as we are going to deploy a web app we are going to choose the following extension shown on the screen.

<p align = "center">
<img src = "/assets/images/img-appservice/captura55.png">
</p>

We will name the project and choose a framework for .NET.

<p align = "center">
<img src = "/assets/images/img-appservice/captura56.png">
</p>

We choose what type of template we are going to make for our web application and then we click on "create".

<p align = "center">
<img src = "/assets/images/img-appservice/captura57.png">
</p>

For this demo I am going to modify the default index.html file in vs code 2019.

<p align = "center">
<img src = "/assets/images/img-appservice/captura58.png">
</p>

Now we are going to deploy our project in the app service we have created. To do this we are going to right click on the "explorer" of our project and click on "publish".

<p align = "center">
<img src = "/assets/images/img-appservice/captura59.png">
</p>

We will see a popup of which way we want to publish our project. In this case we will select "azure".

<p align = "center">
<img src = "/assets/images/img-appservice/captura60.png">
</p>

And in this case we will choose the app service option and click "next". But we can see that we have other options to deploy.

<p align = "center">
<img src = "/assets/images/img-appservice/captura61.png">
</p>

Now if we want to create a new app service in vs code 2019 we click on the following icon that is displayed on the screen, and fill in the following options. and click on "create" (Being the first time we may be asked to log in with our azure account). 

<p align = "center">
<img src = "/assets/images/img-appservice/captura62.png">
</p>

Once the app service has been created, we select the app service we have just created and click on "finish".

<p align = "center">
<img src = "/assets/images/img-appservice/captura63.png">
</p>

Once we have clicked on "finish", we will see a summary of the app service we have just created. Finally we click on "publish" to finish the deploy process.

<p align = "center">
<img src = "/assets/images/img-appservice/captura64.png">
</p>

And as we can see our web app has been published correctly with the changes I have added.

<p align = "center">
<img src = "/assets/images/img-appservice/captura65.png">
</p>

### Deploying with vs code:

Now we are going to learn how to create an app service and deploy our web app in vs code.

First we will go to the extensions section for vs code, and search for app service and install it.

<p align = "center">
<img src = "/assets/images/img-appservice/captura66.png">
</p>

For the first time we will be asked to log in with our azure account, to create the app service we simply click on "+". 

<p align = "center">
<img src = "/assets/images/img-appservice/captura67.png">
</p>

It will ask us to name the app service and then we will choose a language for the runtime stack.

<p align = "center">
<img src = "/assets/images/img-appservice/captura68.png">
</p>

It will ask us to choose a plan for our app service. (if we already have a plan created we select it or we can create another plan).

<p align = "center">
<img src = "/assets/images/img-appservice/captura69.png">
</p>

After selecting the plan, you will be notified that the app service has been created successfully.

<p align = "center">
<img src = "/assets/images/img-appservice/captura70.png">
</p>

Now once we have created the app service, we are going to deploy our web app.

For this demo i have created a html file with the he following content shown in the image. To deploy, right click on the app service and select the option "deploy" and select the folder where we have all the code of our app.

<p align = "center">
<img src = "/assets/images/img-appservice/captura71.png">
</p>

We will get a warning if we are sure to deploy. Click on "deploy" to finish the process.

<p align = "center">
<img src = "/assets/images/img-appservice/captura72.png">
</p>

If the deploy has been done correctly, we will see a warning that the deploy is being done. click on "browser website" to open the browser.

<p align = "center">
<img src = "/assets/images/img-appservice/captura73.png">
</p>

And as we can see the deploy has been done correctly and the changes have been applied.

<p align = "center">
<img src = "/assets/images/img-appservice/captura74.png">
</p>

## Create appservice with azure start devops


