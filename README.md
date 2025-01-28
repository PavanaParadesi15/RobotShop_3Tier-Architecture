# Robot-Shop-3-Tier-Architecture
This is a 3-tier architecture model for a Project called Robot Shop a demo e-commerce project, deployed on to AWS EKS Cluster.

# Three Tier Architecture Deployment on AWS EKS

Stan's Robot Shop is a sample microservice application.

This sample microservice application has been built using these technologies:
- NodeJS ([Express](http://expressjs.com/))
- Java ([Spring Boot](https://spring.io/))
- Python ([Flask](http://flask.pocoo.org))
- Golang
- PHP (Apache)
- MongoDB
- Redis
- MySQL ([Maxmind](http://www.maxmind.com) data)
- RabbitMQ
- Nginx
- AngularJS (1.x)

The various services in the sample application already include all required Instana components installed and configured. The Instana components provide automatic instrumentation for complete end to end [tracing](https://docs.instana.io/core_concepts/tracing/), as well as complete visibility into time series metrics for all the technologies.

## What are 3-tier applications?
* Applications written using 3 different layers - Frontend (Presentation layer UI), Backend (Logic Layer) and Database (Data layer). Backend layer coordinates with Frontend and Database.
* Backend Languages - Java, Python, Golang.

## High Level Design of this Project
### Workflows
There are 2 workflows
**1. User Workflow: **
* On the application UI, user has 2 options, Registration page and Login page. 
* There is Catalogue page for the products the application is selling, User can choose a product from the catalogue
* Rating for the Products
* Add product to Cart
* Next is payment section
* Provide shipping details
* User order is complete, user gets order ID.

* These are the different components of the application and each component can have different logic in different programming languages. 
* If the entire logic of all components is written in a single application, then it is Monolithic application. 
* Other option is micro service architecture. This project follows micro service architecture. 
* Each micro service can be written in different programming languages. In this project each micro service is written in different programming languages.

* In this project UI which is presentation layer is written in Angular
* All the microservice like cart, shipping, user, payments, catalogue comes under logic layer. 
* Databases - All the user details are stored in database and product details are also stored in DB. 
* For products adding to Cart, we need in memory Datastore - Redis. When user logins, they can see the items they added to card. Redis stores them. 
* We can also In memory cache, for storing the items in cart, but the drawback with this is, when the application goes down, everything in the cache will be lost. So using Redis as in memory datastore is the best option. 
* Creates In memory database as stateful set in K8S.
* For storing user details, mongoDB is used. 
* For catalogue and ratings, mysql is used. 

So overall there  are 6 microservices, 2 databases and there is 1 in-memory datastore - Redis. 

Instead of deploying all the microservices as docker containers, I have created helm charts for all of the microsercices. I created k8s deployment and service file for each of the microservice. 

Already created Docker images from Docker hub are used in values.yaml









