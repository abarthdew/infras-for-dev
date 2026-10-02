<table_of_contents color="gray"/>
# \[Section 9\]: MicroServices Architecture
# 45. Microservices Application
## Try & Understand Microservices Architecture using a simple Web Application
- We will then try and deploy this Web Application on multiple different Kubernetes as platforms such as Google Cloud Platform.
- I'm going to use a simple application developed by Docker to demonstrate the various features available in running an application stack on Docker.
- So Let's first get familiarized with the application, because we will be working with the same application in different sections to the rest of the course.
![]()
- This is a sample voting application which provides an interface for a user to vote and another interface to show the result.
![]()
![]()
- The application consists of various components, such as the voting app, which is a Web application developed in Python to provide the user with an interface to choose between two options a cat and a dog.
![]()
- When you make a selection, the vote is stored in Redis for those of you who are new to Redis, In this case, Redis serves as a database In-memory.
![]()
- This vote is then processed by the worker, which is an application written in `.Net`.
![]()
- The Worker application takes the new vote and updates the persistent Database, which is at PostgreSQL in our case.
- The PostgreSQL simply has a table with the number of votes for each category, Cats and Dogs.
- In this case, it increments the number of votes for cats as our vote was for cats.
![]()
![]()
- Finally, the result of the vote is displayed in a Web Interface, which is another Web Application developed in NodeJs.
- This resulting application reads the count of votes from the PostgreSQL Database and displays it to the user.
- ⇒ That is the architecture and dataflow of this simple voting application stack.
- As you can see, this sample application is filled with a combination of different services, different development tools and multiple different development platform such as Python, NodeJs, .Net, etc.
- This sample application will be used to showcase how easy it is to set up an entire application stack consisting of diverse components in Docker.
![]()
- Let us see how we can put together this application stack on a single docker engine using `docker run` commands.
- Let us assume that all images of applications are already build and are available on Docker Repository.
## Let us start with the data layer.
![]()
```shell
docker run -d --name=redis redis
```
### 🔰 First, we are on the `docker run` command to start an instance of Redis by running the `docker run redis` command
- We will add the -(dash)d parameter to run this container in the background and we will also named the container Redis. (Naming the containers is important.)
![]()
```shell
docker run -d --name=db postgres:9.4
```
### 🔰 Next, we will deploy the PostgreSQL database by running the `docker run postgresql` command
- This time, too, we will add the `-d` option to run this in the background and name this container db for Database.
![]()
```shell
docker run -d --name=vote -p 5000:80 voting-app
```
### 🔰 Next, we will start with the application services, we will deploy a front, an app for voting interface by running an instance of voting app image
- Run the `docker run` command and name the instance vote since this is a Web Server.
- It has a Web UI instance running on port 80.
- We will publish that port to 5000 on the host system, so we can access it from a browser.
![]()
```shell
docker run -d --name=result -p 5001:80 result-app
```
### 🔰 Next, we will deploy the result Web Application the show the results the user
- For this, we deploy a container using the result-app image and published port 80 to port 5001 on the host.
- This way we can access the Web UI of the resulting app on a browser.
![]()
```shell
docker run -d --name=worker worker
```
### 🔰 Finally, we deploy the worker by running an instance of the worker image
- Now, This is all good and we can wee that all the instances are running on the host.
![]()
- But, there is some problem. It just not seem to work.
- The problem is that we have successfully run all the different containers, but we haven't actually link them together.
- As in, we haven't told the voting Web Application to use this particular Redis instance. There could be multiple Redis instance running.
- We haven't told the worker and the resulting app to use this particular PostgreSQL database that we ran.
### 🔰 How do we do that?
- That is where we use links. Link is command line option, which can be used to link two containers together.
![]()
- For example, the voting app Web service is dependent on the Redis service.
- When the Web server starts, as you can see in the piece of code on the Web server, it looks for a Redis service running on host Redis.
- But the voting app container cannot resolve a host by the named Redis.
![]()
```shell
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
```
- To make the voting app aware of the Redis service, we add a link option while running the voting app container to link it to the Redis container.
- Adding a `--link` option to the `docker run` command, and specifying the name of the Redis container, which in this case it's redis(:\[name\]), is followed by a :(colon) and the name of the host that the voting app is looking for. Which is also Redis in this case.
- Remember that this is why we named the container when we ran it the first time.
- So we could use its name while creating a link.
![]()
- What this is doing is, it creates an entry into the etc host file on the voting-app container, adding an entry with the host named where the internal IP of the Redis container.
![]()
- Similarly, we add a link for the result app to communicate with the database by adding a link option to refer the database by the name db.
- As you can see in this source code of the application, it makes an attempt to connect to a PostgreSQL database on host db.
![]()
- Finally, the worker application requires access to both the Redis as well as the PostgreSQL database.
- So we add two links to the worker application. One link to link Redis and the other link to link PostgreSQL database.
![]()
- Note that using links this way is deprecated and the support may be removed in our future in Docker.
- This is because, as we will see in some time, advanced and newer concept in Docker Swarm and networking supports better way of achieving what we just did here with links.
# 46. Microservices Application on Kubernetes
- So, we just saw how the voting application works on Docker.
![]()
- Let's now see how to deploy it on Kubernetes.
- So It's important to have a clear idea of what we are trying to achieve and plan accordingly before we get started.
- So we already know how the application works. An It's a good idea to write down what we plan to do.
![]()
1. So our goal is to deploy these containers. These applications as container on our Kubernetes cluster.
2. And then enable connectivity between the containers so that the applications can access each other and the databases
3. And enable external access for the external facing applications, which are the voting and the result-app so that the users can access the Web Browser.
### 🔰 How do we go about this?
- Now we know that we cannot deploy containers directly on Kubernetes.
- We learned that the smallest object that we can create on a Kubernetes cluster is a Pod.
![]()
1. So we must first deploy these applications as a Pod on our Kubernetes cluster. (Or we could deploy them as Replica Sets or Deployments, as we have seen throughout this course. But first, for the sake of simplicity, we will stick to Pods in this lecture. And later we will see how to easily convert that to a deployment.)
2. So once the pods are deployed, the next step is to enable connectivity between the surfaces. It's important to know what the connectivity requirements are. So we must be very clear about what application requires access to what services.
![]()
- We know that the Redis database is accessed by the voting-app and the worker-app.
- The voting-app saves the vote to the Redis database and the worker-app reads the vote from the Redis database.
- We know that the PostgreSQL database is accessed by the worker-app to update it with the total count of votes.
![]()
- And it's also accessed by the result-app to read the total count of vote to be displayed in the resulting Web page browser.
![]()
- So we know that the voting app is accessed by the external users, the voters, and the result app is also accessed by the external users to view the results.
- Most of the components are being accessed by another component except for the worker-app.
- Note that the worker-app is not being accessed by anyone. You can see arrow going into all of these components, but there are no arrows going into worker, which means none of the other components or external users are accessing the worker app.
- The worker-app simply reads the count of votes from the Redis database and then updates the total count of votes on the PostgreSQL database. So none of the other components nor the external users ever access the worker-app.
![]()
- Now, while the voting app has a Python Web Server that listens on port 80 and the result app also has a no JS based server that listens on port 80 and they Redis database has a service that listens on port 6379. And the PostgreSQL database has a service that listens on port 5432.
- The worker-app has no service because It's just a worker and It's just a worker and It's not accessed by any other service or external users.
### 🔰 How do you make one component accessible by another?
- For example, how do you make the greatest database accessible by the voting app? Should the voting-app use the IP address of the Redis Pod perhaps? No, because that can change. The IP of the pod can change if the pod restarts.
- And you may also run into issues when you try to scale your applications in the future.
- The right way to do it to use a service. Now we learned that a service can be used to expose an application to other applications or users for external access.
- The right way to do it to use a service. Now we learned that a service can be used to expose an application to other applications or users for external access.
![]()
- So we will create a service for the Redis Pod so that It can be accessed by the voting-app and the worker-app.
- And we will call it a Redis service, and it will be accessible anywhere within the cluster by the name of the service, Redis.
### 🔰 Why is that name important?
![]()
- The source code within the voting-app and the worker-app are hard coded to point to a Redis database running on a host by the name Redis. So it's important to name your service as Redis.
- These applications can connect to the Redis database.
- And this is not a best practice to hardcore stuff like this within the source code of an application. Instead, you should be using environment variables or something. But for the sake of simplicity, we will just follow this application as it is developed.
- Now, these services are not to be accessed outside the cluster. So they should just be of type ClusterIP.
![]()
- So we will follow the same approach of creating a service for the PostgreSQL Pod so that the PostgreSQL database can be accessed by the worker. And the result app.
### 🔰 What should we name the PostgreSQL service?
![]()
- If you look at the source code of the result app and the worker app, you will see that they are looking for a database at the address DB.
- So the service that we create for PostgreSQL should be named DB.
- Also note that while connecting to the database, the worker and the result-app passing a username and password to connect to the database, both of which are set to PostgreSQL.
- So when we deploy the PostgreSQL DB Pod, we must make sure that we set the these credentials for it as the initial set of credentials to while creating the database.
### 🔰 To enable external access
![]()
- For this, we saw that we could use a service with a type set to NodePort. So we create services for voting -app and the result-app and set there type two NodePort.
- Now we could decide on what port we are going to make them available on. And it would be a high port with a port number greater than 30000. So we'll do that when we create the service. So we are done and we have the high level steps ready.
- To summarize, we will be deploying five pods in total, And we have four services.
- One for Redis another for PostgreSQL, both of which are internal services. So they are of type ClusterIP. And we then have external facing services for voting-app and the result-app.
- However, we have no service for the worker pod. And this is because it is not running any service. That must be accessed by another application or external users.
- So it is just a worker process that reads from one database and updates another. It's not going to require a service.
- Now I say that again, as that's a common question that I get when we talk about services. So, why does the worker not require a service?
- A service is only required if the application has some kind of process or database service or Web service that needs to be exposed. That needs to be accessed by others.
- In this case, that's not true for the worker-app.
![]()
- Now, before we get started with the deployment, Note that we will be using the following Docker images for these applications. So these images are built from a four of the original developed at the Docker samples repository. 
	- The image name : kodekloud/example-voting-app
	- An for the database we will use the official Redis and PostgreSQL release that are available.
# 47. Demo - Deploying Microservices Application on Kubernetes
- 생략
# 48. Demo - Deploying Microservices Application on Kubernetes
- 생략
# 49. Bonus Lecture: Checkout Other Offerings
- 생략
