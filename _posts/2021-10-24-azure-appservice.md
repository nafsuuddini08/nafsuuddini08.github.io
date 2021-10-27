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
+ How to creat a appservice in azure
+ Deploy our web app in the multiple ways:
	- Deploying with ftp
	- Deploying with our github repository
	- Deploying with bitbuket
	- Deploying with azure repo
	- Deploying with vs code 2019
	+ Push our repo in github in vs code 2019
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
