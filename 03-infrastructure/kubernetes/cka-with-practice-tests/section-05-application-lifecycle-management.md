# 89. Application Lifecycle Management - Section Introduction
- We start at Rolling updates and Rollbacks, the different ways to configure applications, scale applications, before finally looking at the primitives of self healing application.
# 90. Download Slide Deck
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.
이 강의 자료
<file src=""></file>
# 91. Rolling Updates and Rollbacks
![]()
- Before we look at how we upgrade our application, let;s try to understand Rollout and versioning in a Deployment.
![]()
- When you first create a Deployment, It triggers a Rollout. A new Rollout creates a new Deployment revision. Let’s call it Revision 1.
![]()
- In the future when the Application is upgraded, meaning when the container version is updated to a new one, a new Rollout is triggered and a new Deployment Revision is cretaed named Revision 2.
- This helps us keep track of the changes made to our Deployment and enables us to Rollback to a previous version of Deployment if necessary.
![]()
- You can see the status of your Rollout by running the command `kubectl rollout status` followed by the name of the Deployment to see the Revisions and history of Rollout.
![]()
- Run the `kubectl rollout history` command followed by the Deployment name and this will show you the Revision and history of our Deployment.
### 🔰 Two Types of Deployment Strategies
- There is 2 types of Deployment strategies. For example, you have 5 Replicas of your Web Application instance deployed. One way to upgrade these to a newer version is to destroy all of these and then create newer versions of Application instances. 
![]()
- Meaning first, destroy the 5 running instances and then deploy 5 new instances of the new application version. 
![]()
- The problem with this, as you can imagine, is that during the period after the older versions are down and before any newer version is up, the application is down and inaccessible to users.
![]()
- This strategy is known as the recreate strategy, and thankfully this is not the default deployment strategy. 
![]()
- The second strategy is where we do not destroy all of them at once. Instead, we take down the older version and bring up a newer version one by one. This way the application never goes down and the upgrade is seamless.
![]()
- Remember, If you do not specify a strategy while creating the Deployment, It will assume it to be Rolling Update. In other words, Rolling Update is the default Deployment strategy.
### 🔰 About Upgrade
![]()
- How exactly do you update your Deployment? When I say update, It could be different things such as updating your Application version by updating the version of Docker containers used, updating their labels or updating the number of Replicas, etc.
![]()
- Since we already have a deployment definition file, it is easy for us to modify this file. 
![]()
- Once we make the necessary changes, we run the `kubectl apply` command to apply the changes. 
![]()
- A new Rollout is triggered and a new Revision of the Deployment is created. But there is another way to do the same thing. You could use the `kubectl set image` command to update the image of your Application, but remember doing it this way will result in the deployment-definition file having a different configuration. So you must be careful when using the same definition file to make changes in the future.
### 🔰 Difference between the recreate and Rolling Update strategies
![]()
- The difference between the recreate and rolling update strategies can also be seen when you view the deployments in detail. Run the `kubectl describe deployment` command to see the detailed information regarding the Deployments.
![]()
- You will notice when the Recreate strategy was used. The events indicate that the old Replica Set was scaled down to 0/1, and then the new Replica Set scaled up to 5.
![]()
- However, when the Rolling  Update strategy was used, the old Replica Set was scaled down one at a time, simultaneously scaling up the new Replica Set one at a time. 
### 🔰 How a Deployment performs an upgrade under the hood
- Let’s look at how a Deployment performs an upgrade under the hoods. 
![]()
- When a new Deployment is created, to deploy 5 Replicas. It first creates a Replica Set automatically, which in turn creates the number of Pods required to meet the number of Replicas.
- When you upgrade your Application, as we saw in the previous slide, the Kubernetes Deployment object creates a new Replica Set under the hoods and starts deploying the Containers there. 
![]()
- At the same time, taking down the Pods in the old Replica Set, following a Rolling update strategy.
![]()
- This can be seen when you try yo list the Replica Set using the `kubectl get replicasets` command. Here we see the old Replica Set with 0 Pod and the new Replica Set with 5 Pods.
- For instance, once you upgrade your Application, you realize something isn’t very right. Something’s wrong with the new version of build you use to upgrade. So you would like to Rollback your Update.
![]()
- Kubernetes Deployments allow you to Rollback to a previous revision to undo a change, run the `kubectl rollout undo` command followed by the name of the Deployment. 
![]()
- The Deployment will then destroy the Pods in the new Replica Set and bring the older ones up in the old Replica Set and your Application is back to its older format.
![]()
- When you compare the output of the `kubectl get replicasets` command, before and after the Rollback, you will be able to notice this difference. 
![]()
- Before the Rollback, the first Replica Set had 0 Pods and new Replica Set had 5 Pods and this is reversed after the Rollback is finished.
### 🔰 Summarize Commands
![]()
- To summarize the commands real quick, use the `kubectl create` command to create the Deployment, `get deployments` command to list the Deployments, `apply` and `set image` commands to update the Deployments and `rollout status` command to see the status of Rollouts and `rollout undo` command to roll back a Deployment operation.
# 92. Practice Test - Rolling Updates and Rollbacks
Practice Test: [https://uklabs.kodekloud.com/topic/practice-test-rolling-updates-and-rollbacks-2/](https://uklabs.kodekloud.com/topic/practice-test-rolling-updates-and-rollbacks-2/)
# 93. Solution: Rolling update : (Optional)
1. We have deployed a simple web application. Inspect the PODs and the Services Wait for the application to fully deploy and view the application using the link called `Webapp Portal` above your terminal.
	![]()
2. What is the current color of the web application? Access the Webapp Portal. ⇒ **`blue`**
3. Run the script named `curl-test.sh` to send multiple requests to test the web application. Take a note of the output. Execute the script at `/root/curl-test.sh`.
	```shell
controlplane ~ ➜  /root/curl-test.sh
Hello, Application Version: v1 ; Color: blue OK

Hello, Application Version: v1 ; Color: blue OK

Hello, Application Version: v1 ; Color: blue OK
	```
4. Inspect the deployment and identify the number of PODs deployed by it. ⇒ **`4`**
	```shell
controlplane ~ ✖ kubectl get deploy
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
frontend   4/4     4            4           3m33s
	```
5. What container image is used to deploy the applications? ⇒ **`kodekloud/webapp-color:v1`**
	```shell
controlplane ~ ➜  kubectl describe deploy
Name:                   frontend
Namespace:              default
CreationTimestamp:      Tue, 30 Aug 2022 05:39:59 +0000
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               name=webapp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        20
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  name=webapp
  Containers:
   simple-webapp:
    Image:        kodekloud/webapp-color:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   frontend-5c74c57d95 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  4m7s  deployment-controller  Scaled up replica set frontend-5c74c57d95 to 4
	```
6. Inspect the deployment and identify the current strategy. ⇒ **`RollingUpdate`**
7. If you were to upgrade the application now what would happen? ⇒ **`PODs are upgraded few at a time`**
8. Let us try that. Upgrade the application by setting the image on the deployment to `kodekloud/webapp-color:v2`. Do not delete and re-create the deployment. Only set the new image name for the existing deployment.
	- Deployment Name: frontend
	- Deployment Image: kodekloud/webapp-color:v2
	⇒ Run the command `kubectl edit deployment frontend` and modify the image to `kodekloud/webapp-color:v2`. Next, save and exit. The pods should be recreated with the new image.
9. Run the script `curl-test.sh` again. Notice the requests now hit both the old and newer versions. However none of them fail. Execute the script at `/root/curl-test.sh`.
10. Up to how many PODs can be down for upgrade at a time. Consider the current strategy settings and number of PODs - 4. ⇒ **`1`**
11. Change the deployment strategy to `Recreate`. Delete and re-create the deployment if necessary. Only update the strategy type for the existing deployment.
	- Deployment Name: frontend
	- Deployment Image: kodekloud/webapp-color:v2
	- Strategy: Recreate
	```shell
controlplane ~ ➜  kubectl edit deployment frontend

apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: default
spec:
  replicas: 4
  selector:
    matchLabels:
      name: webapp
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        name: webapp
    spec:
      containers:
      - image: kodekloud/webapp-color:v2
        name: simple-webapp
        ports:
        - containerPort: 8080
          protocol: TCP
	```
12. Upgrade the application by setting the image on the deployment to `kodekloud/webapp-color:v3`. Do not delete and re-create the deployment. Only set the new image name for the existing deployment.
	- Deployment Name: frontend
	- Deployment Image: kodekloud/webapp-color:v3
	⇒ Run the command: `kubectl edit deployment frontend` and modify the image to `kodekloud/webapp-color:v3`. Next, save and exit. The pods should be recreated with the new image.
13. Run the script `curl-test.sh` again. Notice the failures. Wait for the new application to be ready. Notice that the requests now do not hit both the versions. Execute the script at `/root/curl-test.sh`.
# 94. Configure Applications
Configuring applications comprises of understanding the following concepts:
- Configuring Command and Arguments on applications
- Configuring Environment Variables
- Configuring Secrets
We will see these next
# 95. Commands
- In this section, we will talk about Command and Arguments in a pod-definition file. This is not listed as a required topic in the certification curriculum, but I think it’s important to explain as it is a topic that is usually overlooked. Let us first refresh our memory on Commands in Containers and Docker. We will then translate this into Pods in the next lecture.
![]()
- In this lecture, we will look at Commands arguments and entry points in Docker. For example, Say you were to run a Docker Container from an Ubuntu image. When you run the `docker run ubuntu` command, it runs an instance of Ubuntu image and exits immediately. If you were to list the running containers, you wouldn’t see the container running.
![]()
- If you list all Containers including those that are stopped you will see that the new Container you ran is in an exited state.
![]()
- Now, why is that? Unlike Virtual Machines, Containers are not meant to host an operating system. Containers are meant to run a specific task or process such as to host an instance of a Web Server or Application Server or a Database or simply to carry out some kind of computation or analysis. Once the task is complete, the Container exits, a Container only lives as long as the process inside it is alive.
![]()
- If the Web Service inside the Container is stopped or crashes the Container exits. So who defines what process is run within the Container.
![]()
- If you look at the Docker file for poplar Docker images like Nginx you will see an instruction called CMD which stands for command that defines the program that will be run within the container when it starts. 
![]()
- For the Nginx image, it is the Nginx command, for the MySQL image, it is the MySQL command.
![]()
- What we tried to do earlier was to run a Container with a plain Ubuntu Operating System. Let us look at the Docker file for this image you will see that it uses bash as the default command. Now bash is now really a process like a Web Server or Database Server. It is a shell that listens for inputs from a terminal if it cannot find a terminal it exits.
![]()
- When we ran the Ubuntu Container earlier, Docker created a Container from the Ubuntu image and launched the bash program. By default Docker does not attach a terminal to a Container when it is run.
![]()
- And so the bash program does not find the terminal and so it exits since the process that was started when the Container was created, finished, the Container exits as well.
- So how do you specify a different command to start the Container? 
	![]()
- Option 1: To append a command to the `docker run` command and that way it overrides the default command specified within the image. In this case, I run the `docker run ubuntu` command with the ‘sleep 5’ command as the added option.
	![]()
- This way when the Container starts it runs the sleep program, waits for 5 seconds and then exits. But how do you make that change permanent?
![]()
- Say you want the image to always run the sleep command when it starts. You would then create your own image from the base Ubuntu image and specify a new command.
![]()
- There are different ways of specifying the command either the command simply as is in a shell form or in a JSON array format like this. But remember, when you specify in a JSON array format, the first element in the array should be the executable. 
![]()
- In this case, the sleep program do not specify the command and parameters together like this. The command and its parameters should be separate elements in the list.
![]()
- So I now build by new image using the `docker build` command, and name it as ubuntu-sleeper.
![]()
- I could now simply run the Docker Ubuntu sleeper command and get the same results. It always sleeps for 5 seconds and exits.
- But what if I wish to change the number of seconds it sleeps. Currently, It is hard coded to 5 seconds. As we learned before, one option is to run the `docker run` command with the new command appended to it.
![]()
- In this case, sleep 10 and so the command that will be run at startup will be sleep 10. But it doesn’t look very good. The name of the image Ubuntu sleeper in itself implies that the Container will sleep. So we shouldn’t have to specify the sleep command again.
![]()
- Instead we would like it to be something like this `docker run ubuntu-sleeper 10`. We only want to pass in the number of seconds the Containers should sleep and sleep command should be invoked automatically.
![]()
- And that is where the entry point instruction comes into play. 
![]()
- The entry point instruction is like the command instruction as in you can specify the program that will be run when the Container starts and whatever you specify on the Command line, in this case, 10 will be appended to the entry point so the command that will be run when the Container starts is sleep 10. So that’s the difference between the two. In case of the CMD instruction, the command line parameters passed will get replaced entirely, whereas in case of entry point, the command line parameters will get appended.
![]()
- Now, in the second case, what if I run the ubuntu-sleeper without appending the number of seconds. 
![]()
- Then the command at startup will be just sleep and you get the error that the operant is missing. So how do you configure a default value for the command?
![]()
- If one was not specified in the command line. That’s where you would use both entry point as well as the command instruction.
![]()
- In this case, the command instruction will be appended to the entry point instruction. So at startup the command would be sleep 5, if you didn’t specify any parameters in the command line. 
![]()
- If you did then that will override the command instruction. And remember for this, to happen you should always specify the entry point and command instructions in a JSON format. 
- Finally, what If you really want to modify the entry point during runtime. Say from sleep to an imaginary sleep 2.0 command.
![]()
- In that case, you can override it by using the entry point option in the `docker run` command. The final command at startup would then be sleep 2.0 10.
# 96. Commands and Arguments
- We will look at Command and Arguments in a Kubernetes Pod. In previous lecture, we create a simple Docker image that sleeps for a given number of seconds.
![]()
- We name it ubuntu-sleeper and we ran it using the docker command `docker run ubuntu-sleeper`. By default, it sleeps for 5 seconds, but you can override it by passing a command line argument.
![]()
- We will now create a Pod using this image.
![]()
- We start with a blank pod-definition template, input the name of the Pod and specify the image name. When the Pod is created, it creates a container from the specified image, and the Container sleeps for 5 seconds before exiting.
![]()
- Now if you need the Container to sleep for 10 seconds as in the second command, how do you specify the additional argument in the pod-definition file? Anything that is appended to the `docker run` command will go into the “args” property of the pod-definition file in the form of an array like this.
![]()
- Let us try to relate that to the Docker file we created earlier. The Dockerfile has an entrypoint as well as a CMD instruction specified. The entrypoint is thecommand that is run at startup, and the CMD is the default parameter passed to the command. With the args option in the pod-definition file, we override the CMD instruction in the Dockerfile.
![]()
- But what if you need to override the entrypoint? Say from sleep to a hypothetical sleep 2.0 command? 
![]()
- In the Docker world, we would run the `docker run` command with the entrypoint option set to the new command the corresponding entry in the pod-definition file would be using a command field. The command field corresponds to entry point instruction in the Docker file.
![]()
- So to summarize, there are two fields that correspond to two instructions in the Docker file. The command field overrides the entry point instruction and the args field overrides the command instruction in the Docker file. Remember, it is not the command field that overrides the CMD instruction in the Docker file.
# 97. Practice Test - Commands and Arguments
Practice Test: [https://uklabs.kodekloud.com/topic/practice-test-commands-and-arguments-2/](https://uklabs.kodekloud.com/topic/practice-test-commands-and-arguments-2/)
# 98. Solution - Commands and Arguments (Optional)
1. How many PODs exist on the system? in the current(default) namespace. ⇒ **`1`**
	```shell
controlplane ~ ➜  kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
ubuntu-sleeper   1/1     Running   0          40s
	```
2. What is the command used to run the pod `ubuntu-sleeper`? ⇒ **`sleep 4800`**
	```shell
controlplane ~ ➜  kubectl describe pod ubuntu-sleeper 
Name:         ubuntu-sleeper
Namespace:    default
Priority:     0
Node:         controlplane/172.25.1.69
Start Time:   Tue, 30 Aug 2022 07:53:56 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  ubuntu:
    Container ID:  containerd://958d55ca6223d6f7bd3aabed5461e10417eb3080b52b0df8b0fe1c141ce76701
    Image:         ubuntu
    Image ID:      docker.io/library/ubuntu@sha256:34fea4f31bf187bc915536831fd0afc9d214755bf700b5cdb1336c82516d154e
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4800
    State:          Running
      Started:      Tue, 30 Aug 2022 07:54:03 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-l64sn (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-l64sn:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  102s  default-scheduler  Successfully assigned default/ubuntu-sleeper to controlplane
  Normal  Pulling    102s  kubelet            Pulling image "ubuntu"
  Normal  Pulled     96s   kubelet            Successfully pulled image "ubuntu" in 5.328394516s
  Normal  Created    96s   kubelet            Created container ubuntu
  Normal  Started    96s   kubelet            Started container ubuntu
	```
3. Create a pod with the ubuntu image to run a container to sleep for 5000 seconds. Modify the file `ubuntu-sleeper-2.yaml`. Note: Only make the necessary changes. Do not modify the name.
	- Pod Name: ubuntu-sleeper-2
	- Command: sleep 5000
	```shell
controlplane ~ ➜  vim ubuntu-sleeper-2.yaml
controlplane ~ ✖ cat ubuntu-sleeper-2.yaml
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-2 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "5000"

controlplane ~ ➜  kubectl create -f ubuntu-sleeper-2.yaml 
pod/ubuntu-sleeper-2 created
	```
4. Create a pod using the file named `ubuntu-sleeper-3.yaml`. There is something wrong with it. Try to fix it! Note: Only make the necessary changes. Do not modify the name.
	- Pod Name: ubuntu-sleeper-3
	- Command: sleep 1200
	```shell
controlplane ~ ➜  kubectl create -f ubuntu-sleeper-3.yaml 
pod/ubuntu-sleeper-3 created
	```
5. Update pod `ubuntu-sleeper-3` to sleep for 2000 seconds. Note: Only make the necessary changes. Do not modify the name of the pod. Delete and recreate the pod if necessary.
	- Pod Name: ubuntu-sleeper-3
	- Command: sleep 2000
	```shell
controlplane ~ ➜  vim ubuntu-sleeper-3.yaml 

controlplane ~ ➜  kubectl create -f ubuntu-sleeper-3.yaml 
pod/ubuntu-sleeper-3 created

controlplane ~ ➜  cat ubuntu-sleeper-3.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: ubuntu-sleeper-3 
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command:
      - "sleep"
      - "2000"
	```
6. Inspect the file `Dockerfile` given at `/root/webapp-color` directory. What command is run at container startup? ⇒ **`python app.py`**
	```shell
controlplane ~ ➜  cd /root/webapp-color

controlplane ~/webapp-color ➜  ls
Dockerfile   Dockerfile2

controlplane ~/webapp-color ➜  cat Dockerfile
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]
	```
7. Inspect the file `Dockerfile2` given at `/root/webapp-color` directory. What command is run at container startup? ⇒ **`python app.py --color red`**
	```shell
controlplane ~ ➜  cd /root/webapp-color

controlplane ~/webapp-color ➜  ls
Dockerfile   Dockerfile2

controlplane ~/webapp-color ➜  cat Dockerfile2
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
	```
8. Inspect the two files under directory `webapp-color-2`. What command is run at container startup? Assume the image was created from the Dockerfile in this folder. ⇒ **`--color green`**
	```shell
controlplane ~/webapp-color ✖ cd ..

controlplane ~ ➜  ls
sample.yaml            webapp-color
ubuntu-sleeper-2.yaml  webapp-color-2
ubuntu-sleeper-3       webapp-color-3
ubuntu-sleeper-3.yaml

controlplane ~ ➜  cd webapp-color-2

controlplane ~/webapp-color-2 ➜  ls
Dockerfile2            webapp-color-pod.yaml

controlplane ~/webapp-color-2 ➜  cat webapp-color-pod.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    command: ["--color","green"]
	```
9. Inspect the two files under directory `webapp-color-3`. What command is run at container startup? Assume the image was created from the Dockerfile in this folder. ⇒ **`python app.py --color pink`**
	![]()
	```shell
controlplane / ➜  cd /root/webapp-color-3

controlplane ~/webapp-color-3 ➜  ls
Dockerfile2              webapp-color-pod-2.yaml

controlplane ~/webapp-color-3 ➜  cat Dockerfile2 
FROM python:3.6-alpine

RUN pip install flask

COPY . /opt/

EXPOSE 8080

WORKDIR /opt

ENTRYPOINT ["python", "app.py"]

CMD ["--color", "red"]
	```
10. Create a pod with the given specifications. By default it displays a `blue` background. Set the given command line arguments to change it to `green`.
	- Pod Name: webapp-green
	- Image: kodekloud/webapp-color
	- Command line arguments: --color=green
	```shell
controlplane ~ ➜  vim pod.yaml

controlplane ~ ➜  cat pod.yaml 
apiVersion: v1 
kind: Pod 
metadata:
  name: webapp-green
  labels:
      name: webapp-green 
spec:
  containers:
  - name: simple-webapp
    image: kodekloud/webapp-color
    args: ["--color", "green"]

controlplane ~ ➜  kubectl create -f pod.yaml 
pod/webapp-green created
	```
# 99. Configure Environment Variables in Applications
![]()
- In this lecture, we will see how to set an environment variable in Kubernetes. Given a pod-definition file which uses the same image as the Docker command we ran in the last lecture. To set an environment variable, use the `env` property. `env` is an array. So every item under the env property starts with a dash, indicating an item in the array.
![]()
- Each item has a name and a value property. 
![]()
- The name is the name of the environment variable made available with the container and the value is its value.
![]()
- What we just saw was a direct way of specifying the environment variables using a plain key value pair format.
- However, there are other ways of setting the environment variables such as suing config maps and secrets.
![]()
- The difference in this case is that instead of specifying value, we say `valueForm`. 
![]()
- And then a specification of configMap or secret.
- We will discuss about configMaps and secretKeys in the upcoming lectures.
# 100. Configuring ConfigMaps in Applications
![]()
- In this lecture, we discuss how to work with configuration data in Kubernetes. In the previous lecture, we saw how to define environment variables in it pod-definition file. When you have a lot of pod-definition files, it will become difficult to manage the environment data stored within the query files.
![]()
- We can take this information out of the pod-definition file and manage it centrally using Configuration Maps. ConfigMaps are used to pass configuration data in the form of key value pairs in Kubernetes. 
![]()
- When it Pod is created inject the ConfigMap into the Pod, so the key value pairs that are available as environment variables for the application hosted inside the container in the Pod.
![]()
- There are 2 phases involved in configuring ConfigMaps. First create the ConfigMap and second inject them into the Pod. 
![]()
- Just like any other Kubernetes object, there are 2 ways of creating a ConfigMap. The `imperative` way - without using a ConfigMap definition file and the `declarative` way - by using a ConfigMap definition file.
![]()
- If you do not with to create a ConfigMap definition file, you could simply use the `kubectl create  configmap` command and specify the required arguments.
![]()
- Let’s take a look at that first. With this method, you can directly specify the key value pairs in the command line. To create a ConfigMap of the given values, run the `kubectl create configmap` command. The command is followed by the config name and the option `--from-literal`. The `--from-literal` option is used to specify the key value pairs in the command itself. In this example, we are creating a ConfigMap by the name `app-config`, with a key value pair `APP_COLOR=blue`.
![]()
- If you wish to add additional key value pairs, simply specify the `--from-literal` options multiple times. However, this will get complicated when you have too many configuration items.
![]()
- Another way to input configuration data is through a file. Use the `--from-file` option to specify a path to the file that containers the required data. The data from this file is read and stored under the name of the file.
![]()
- Let us now look at the declarative approach. For this, we create a definition file just like how we did for the Pod. The file has `apiVersion`, `kind`, `metadata` and instead of spec, here we have `data`.
![]()
- The apiVersion is ‘v1’, kind is ‘ConfigMap’, under metadata specif the name of the ConfigMap, we will call it ‘app-config’. Under data add the configuration data in a key value format. Run the `kubectl create` command and specify the configuration file name. So that creates the ‘app-config’ ConfigMap with the values we specified.
- You can create as many ConfigMaps as you need in the same way for various different purposes.
![]()
- Here I have one for my application, other for MySQL and another one for Redis.
![]()
- So It is important to name the ConfigMaps appropriately as you will be using these names later while associating it with Pods. To view the ConfigMaps, run the `kubectl get configmaps` command. 
![]()
- This lists the newly created ConfigMap named ‘app-config’. The describe ConfigMaps command list the configuration data as well under the data section.
![]()
- Now that we have the ConfigMap created, let us proceed with step 2, configuring it with a Pod. Here I have a simple pod-definition file that runs my simple web application. To inject an environment variable add a new property to the container called `envForm`.
- The `envForm` property is a list, so we can pass as many environment variables as required. Each item in the list corresponds to a ConfigMap item. 
![]()
- Specify the name of the ConfigMap we created earlier, This is how we inject a specific ConfigMap from the ones we created before. 
![]()
- Creating the pod-definition file creates a web application with a blue background.
![]()
- What we just saw was using ConfigMaps to inject environment variables, there are other way to inject configuration data into Pods.
![]()
- You can inject it as a single environment variable or you can inject the whole the data as files in a volume. We will look at some of these options in the coding exercises that accompany this lecture.
# 101. Practice Test: Environment Variables
Practice Test: [https://uklabs.kodekloud.com/topic/practice-test-env-variables-2/](https://uklabs.kodekloud.com/topic/practice-test-env-variables-2/)
# 102. Solution - Environment Variables (Optional)
1. How many PODs exist on the system? in the current(default) namespace. ⇒ **`1`**
	```shell
controlplane ~ ➜  kubectl get pods
NAME           READY   STATUS    RESTARTS   AGE
webapp-color   1/1     Running   0          68s
	```
2. What is the environment variable name set on the container in the pod? ⇒ **`APP_COLOR`**
	```shell
controlplane ~ ➜  kubectl describe pod webapp-color 
Name:         webapp-color
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.87
Start Time:   Wed, 31 Aug 2022 07:10:04 +0000
Labels:       name=webapp-color
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  webapp-color:
    Container ID:   containerd://140ce66f48f7aa283e745372dfe84cc8db2d8b23d78bd2d38140e71f31a43ed9
    Image:          kodekloud/webapp-color
    Image ID:       docker.io/kodekloud/webapp-color@sha256:99c3821ea49b89c7a22d3eebab5c2e1ec651452e7675af243485034a72eb1423
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 31 Aug 2022 07:10:11 +0000
    Ready:          True
    Restart Count:  0
    Environment:
      APP_COLOR:  pink
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-5fs9t (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-5fs9t:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  104s  default-scheduler  Successfully assigned default/webapp-color to controlplane
  Normal  Pulling    104s  kubelet            Pulling image "kodekloud/webapp-color"
  Normal  Pulled     98s   kubelet            Successfully pulled image "kodekloud/webapp-color" in 5.673276514s
  Normal  Created    98s   kubelet            Created container webapp-color
  Normal  Started    98s   kubelet            Started container webapp-color
	```
3. What is the value set on the environment variable `APP_COLOR` on the container in the pod? ⇒ **`pink`**
4. View the web application UI by clicking on the `Webapp Color` Tab above your terminal. This is located on the right side.
	![]()
5. Update the environment variable on the POD to display a `green` background Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.
	- Pod Name: webapp-color
	- Label Name: webapp-color
	- Env: APP_COLOR=green
	```shell
controlplane ~ ✖ kubectl delete pod webapp-color 
pod "webapp-color" deleted

controlplane ~ ✖ vim webapp-color.yml

controlplane ~ ➜  cat webapp-color.yml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - env:
    - name: APP_COLOR
      value: green
    image: kodekloud/webapp-color
    name: webapp-color

controlplane ~ ➜  kubectl create -f webapp-color.yml 
pod/webapp-color created
	```
6. View the changes to the web application UI by clicking on the `Webapp Color` Tab above your terminal. If you already have it open, simply refresh the browser.
	![]()
7. How many `ConfigMaps` exists in the `default` namespace? ⇒ **`2`**
	```shell
controlplane ~ ➜  kubectl get configmaps
NAME               DATA   AGE
kube-root-ca.crt   1      10m
db-config          3      29s
	```
8. Identify the database host from the config map `db-config`. ⇒ **`SQL01.example.com`**
	```shell
controlplane ~ ➜  kubectl describe configmaps
Name:         kube-root-ca.crt
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/description:
                Contains a CA bundle that can be used to verify the kube-apiserver when using internal endpoints such as the internal service IP or kubern...

Data
====
ca.crt:
----
-----BEGIN CERTIFICATE-----
MIIBdjCCAR2gAwIBAgIBADAKBggqhkjOPQQDAjAjMSEwHwYDVQQDDBhrM3Mtc2Vy
dmVyLWNhQDE2NjE5Mjk1NzYwHhcNMjIwODMxMDcwNjE2WhcNMzIwODI4MDcwNjE2
WjAjMSEwHwYDVQQDDBhrM3Mtc2VydmVyLWNhQDE2NjE5Mjk1NzYwWTATBgcqhkjO
PQIBBggqhkjOPQMBBwNCAASFM+OW6j/HUebBGyqH2SV/z+4ewmaNhEfRbRjBiuju
Oo7DzDK8gKeSSVg9Vjfg2PhFybsmUaPBMo26RSujF14po0IwQDAOBgNVHQ8BAf8E
BAMCAqQwDwYDVR0TAQH/BAUwAwEB/zAdBgNVHQ4EFgQUx/NGLCfoFYCTwyZ5A1Th
nc8kChkwCgYIKoZIzj0EAwIDRwAwRAIgYVcsPsyvIMS6fv3nDMmS7HC51IVzzkxL
1zbmmX9gxLwCIFJc7d2sanjySeCjrPqDI3U2zLLgi32T5bqCk6nRGiei
-----END CERTIFICATE-----


BinaryData
====

Events:  <none>


Name:         db-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
DB_PORT:
----
3306
DB_HOST:
----
SQL01.example.com
DB_NAME:
----
SQL01

BinaryData
====

Events:  <none>
	```
9. Create a new ConfigMap for the `webapp-color` POD. Use the spec given below.
	(Hint) Run the command `kubectl create configmap webapp-config-map --from-literal=APP_COLOR=darkblue`
	- ConfigName Name: webapp-config-map
	- Data: APP_COLOR=darkblue
	```shell
controlplane ~ ✖ kubectl create -f webapp-config-map.yml 
configmap/webapp-config-map created

controlplane ~ ➜  cat webapp-config-map.yml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: webapp-config-map
data:
  APP_COLOR: darkblue
	```
10. Update the environment variable on the POD to use the newly created ConfigMap. Note: Delete and recreate the POD. Only make the necessary changes. Do not modify the name of the Pod.
	- Pod Name: webapp-color
	- EnvFrom: webapp-config-map
	```shell
controlplane ~ ➜  cat webapp-color.yml 
apiVersion: v1
kind: Pod
metadata:
  labels:
    name: webapp-color
  name: webapp-color
  namespace: default
spec:
  containers:
  - envFrom:
    - configMapRef:
         name: webapp-config-map
    image: kodekloud/webapp-color
    name: webapp-color

controlplane ~ ➜  kubectl create -f webapp-color.yml 
pod/webapp-color created
	```
11. View the changes to the web application UI by clicking on the `Webapp Color` Tab above your terminal. If you already have it open, simply refresh the browser.
	![]()
# 103. Configure Secrets in Applications
![]()
- Here we have a simple Python Web Application that connects to a MySQL Database. 
![]()
- On success, the Application displays a successful message.
![]()
- If you look closely into the code, you will see the hostname, username, and password hardcoded. This is of course not a good idea.
![]()
- As we learned in the previous lecture, one option would be to move these values into a ConfigMap. 
![]()
- The ConfigMap Stores configuration data in plain text format. So while it would be okay to move the hostname and username into a ConfigMap it is definitely not the right place to store a password. This is where secrets coming.
![]()
- Secrets are used to store sensitive information like passwords or keys. They’re similar to ConfigMap except that they’re stored in an encoded or hashed format.
![]()
- As with ConfigMaps, there are 2 steps involved in working with secrets. First, create the secret, and second, injected the Pod.
![]()
- There are 2 ways of creating a secret. The `imperative` way - without using a secret definition file and the `declarative` way - by using a secret definition file.
![]()
- With the imperative method, you can directly specify the key value pairs in the command line itself. To create a secret of the given values, run the `kubectl create secret generic` command. The command is followed by the secret name and the option `--form-literal`. The `--from-literal` option is used to specify the key value pairs in the command itself. In this example, we are creating a secret by the name ‘app-secret’, with a key value pair ‘DB_HOST=mysql’. 
![]()
- If you wish to add additional key value pairs, simply specify the `—from-literal` options multiple times. 
![]()
- However, this could get complicated when you have too many secret to pass in. Another way to input the secret data is through a file. Use the `--from-file` option to specify a path to the file that contains the required data. The data from this file is read and stored under the name of the file.
![]()
- Let us now look at the `declarative`  approach. For this, we create a definition file, just like how we did for the ConfigMap. The file has `apiVersion`, `kind`, `metadata` and `data`. The `apiVersion` is ‘v1’, kind is ‘Secret’. Under metadata specify the name of the Secret, we will call it ‘app-secret’.
![]()
- Under the data, add the secret data in a key-value format. 
![]()
- However, on thing we discussed about secrets was that there used to store sensitive data and are stored in an encoded format. Here we have specified the data in plain text which is not very safe. 
![]()
- So while creating a secret with a declarative approach, you must specify the secret values in a hashed format. 
![]()
- So you must specify the data in an encoded form like this.
![]()
- But how do you convert the data from plain text to an encoded format.
![]()
- On a Linux host from the command, `echo -n` followed by the text you are trying to convert, which is MySQL in this case and pipe that to the base64 utility. 
![]()
- To view secrets run the `kubectl get secrets` command. 
![]()
- This lists the newly created secret along with another secret previously created by Kubernetes for its internal purposes.
- To view more information on the newly created Secret, run the `kubectl describe secret` command. This shows the attributes in the Secret. But hides the value themselves. 
![]()
- To view the values as well, run the `kubectl get secret` command with the output displayed in a YAML format using the `-o` option. You can now see the hashed values as well.
![]()
- Now, how do you decode these hashed values? Use the same base64 command used earlier to encode it, but this time add a decode option to it. Now that we have Secret created, let us proceed with step 2.
![]()
- Configuring it with a Pod, here I have a simple pod-definition file that runs my application.
![]()
- To inject an environment variable, add a new property to the container called `envForm`. The `envForm` property is a list. So we can pass as many environment variables as required.
![]()
- Each item in the list corresponds to a secret item. Specify the name of the Secret we created earlier, creating the pod-definition file now makes the data in the Secret available as environment variables for the application.
![]()
- What we just saw was injecting Secrets as environment variables into the Pods. There are other ways to inject secret into Pods.
![]()
- You can inject as single environment variables or inject the whole secret as files in a volume.
![]()
- If you were to mount the Secret as a volume in the Pod, each attribute in the Secret is created as a file with the value of the Secret as its content. In this case, since we have 3 attributes in our Secret 3 files are created and If we look at the contents of the ‘DB_Password’ file we see the password in it.
# 104. A note about Secrets!
Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.
The concept of safety of the Secrets is a bit confusing in Kubernetes. The [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it.
Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:
- Not checking-in secret object definition files to source code repositories.
- [Enabling Encryption at Rest ](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/)for Secrets so they are stored encrypted in ETCD.
Also the way kubernetes handles secrets. Such as:
- A secret is only sent to a node if a pod on that node requires it.
- Kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.
Read about the [protections ](https://kubernetes.io/docs/concepts/configuration/secret/#protections)and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)
Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, [HashiCorp Vault](https://www.vaultproject.io/). I hope to make a lecture on these in the future.
# 105. Practice Test - Secrets
Practice Test: [https://uklabs.kodekloud.com/topic/practice-test-secrets-2/](https://uklabs.kodekloud.com/topic/practice-test-secrets-2/)
# 106. Solution - Secrets (Optional)
1. How many `Secrets` exist on the system? in the current(default) namespace. ⇒ **`1`**
	```shell
controlplane ~ ➜  kubectl get secrets
NAME                  TYPE                                  DATA   AGE
default-token-xn5z9   kubernetes.io/service-account-token   3      4m7s
	```
2. How many secrets are defined in the `default-token` secret? ⇒ **`3`**
	```shell
controlplane ~ ➜  kubectl describe secret default-token-xn5z9 
Name:         default-token-xn5z9
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: default
              kubernetes.io/service-account.uid: 985769eb-b5f5-4a07-aeef-6be814b67fa9

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     566 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjJPOWVoMkhhZzU0UEkxMWhzWkNYUFQyZWYyUW8yeEw3WVNyQTNwXzY3dkkifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4teG41ejkiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6Ijk4NTc2OWViLWI1ZjUtNGEwNy1hZWVmLTZiZTgxNGI2N2ZhOSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.KqXUKOsASWY-IWk9zZc4eUitP8uFgwRh0BPl_HGm7Jo4J_V3qci8dtvbWqofbf5-k3t3Yq8Dxy5U9k3kadNAp6cT7-dpmd0twGh8CCLWZQ3aLd6em1ahbi96B1k8CbzcZ_8IrwuguAT3g3KP2Od8xRARE56Th4DdgKWhR-LoWNZupdrcvQUCADKkleL8Tm9Pk9SY4UQhTIlRDDbGjYJl96bvWNzSlMTxFl3FwPhfFkPbO-p31IsGUo_vCdXpCxmoPiNF7Qcd4kmxwHAo9MbUjlvEuadS7UeieHkQzplBRs42mh6kqGzIcn0TXAPV7f8v0LyiGCEC3HtOTR5vNKZEAg
	```
3. What is the type of the `default-token` secret? ⇒ **`kubernetes.io/service-account-token`**
4. Which of the following is not a secret data defined in `default-token` secret? ⇒ **`type`**
5. We are going to deploy an application with the below architecture We have already deployed the required pods and services. Check out the pods and services created. Check out the web application using the `Webapp MySQL` link above your terminal, next to the Quiz Portal Link.
	![]()
6. The reason the application is failed is because we have not created the secrets yet. Create a new secret named `db-secret` with the data given below. You may follow any one of the methods discussed in lecture to create the secret.
	(Hint) Run the command: `kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123`
	- Secret Name: db-secret
	- Secret 1: DB_Host=sql01
	- Secret 2: DB_User=root
	- Secret 3: DB_Password=password123
	![]()
	```shell
controlplane ~ ➜  kubectl create secret generic db-secret --from-literal=DB_Host=sql01 --from-literal=DB_User=root --from-literal=DB_Password=password123
secret/db-secret created

controlplane ~ ➜  kubectl edit secret db-secret

apiVersion: v1
data:
  DB_Host: c3FsMDE=
  DB_Password: cGFzc3dvcmQxMjM=
  DB_User: cm9vdA==
kind: Secret
metadata:
  creationTimestamp: "2022-08-31T08:40:56Z"
  name: db-secret
  namespace: default
  resourceVersion: "943"
  uid: 47aba266-56d4-4950-b827-eaef5ff3803f
type: Opaque
...
	```
	```shell
controlplane ~ ✖ cat db-secret.yml 
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  DB_Host: sql01
  DB_User: root
  DB_Password: password123

controlplane ~ ➜  echo -n 'spq01' | base64
c3BxMDE=

controlplane ~ ➜  echo -n 'root' | base64
cm9vdA==

controlplane ~ ➜  echo -n 'password123' | base64
cGFzc3dvcmQxMjM=

controlplane ~ ➜  vim db-secret.yml

controlplane ~ ➜  cat db-secret.yml 
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
data:
  DB_Host: c3BxMDE=
  DB_User: cm9vdA==
  DB_Password: cGFzc3dvcmQxMjM=C
	```
7. Configure `webapp-pod` to load environment variables from the newly created secret. Delete and recreate the pod if required.
	- Pod name: webapp-pod
	- Image name: kodekloud/simple-webapp-mysql
	- Env From: Secret=db-secret
	```shell
controlplane ~ ✖ kubectl delete pod webapp-pod 
pod "webapp-pod" deleted

controlplane ~ ✖ vim webapp-pod.yml

controlplane ~ ➜  cat webapp-pod.yml 
apiVersion: v1 
kind: Pod 
metadata:
  labels:
    name: webapp-pod
  name: webapp-pod
  namespace: default 
spec:
  containers:
  - image: kodekloud/simple-webapp-mysql
    imagePullPolicy: Always
    name: webapp
    envFrom:
    - secretRef:
        name: db-secret

controlplane ~ ➜  kubectl create -f webapp-pod.yml 
pod/webapp-pod created
	```
8. View the web application to verify it can successfully connect to the database.
	![]()
# 107. Scale Applications
We have already discussed about scaling applications in the Deployments and Rolling updates and Rollback sections.
# 108. Multi Container PODs
![]()
- The idea of decoupling a large Monolithic Application into sub-components known as Microservices enables us to develop and deploy a set of independent small and reusable code.
![]()
- This architecture can then help us scale up, down as well as modify each Service as required as opposed to modifying the entire Application.
![]()
- However at times you may need 2 Services to work together such as a Web Server and a logging Service.
![]()
- You need on agent instance per Web Server instance paired together. You don’t want to march and bloat the code of the 2 Services as each of them target different functionalities,
![]()
- And you’d still like them to be developed and deployed separately.
- You only need the 2 functionality yo work together.
![]()
- You need 1 agent per Web Server instance paired together and can scale up and down together.
![]()
- And that is why you have multi-container Pods that share the same life cycle which means they are created together and destroy together.
![]()
- They share the same network space which means they can refer to each other as local host and they have access to the same storage volumes. This way you do not have to establish volume sharing or Services between the Pods to enable communication between them.
![]()
- To create a Multi-Container Pod, add the new container information to the pod-definition file. Remember, the container section under the `spec` section in a pod-definition file is an array. And the reason it is an array is to allow multiple containers in a single Pod.
![]()
- In this case, we add a new container named log agent to our existing Pod.
# 109. Practice Test - Multi Container PODs
Link to Practice Test: [https://uklabs.kodekloud.com/topic/practice-test-multi-container-pods-2/](https://uklabs.kodekloud.com/topic/practice-test-multi-container-pods-2/)
# 110. Solution - Multi-Container Pods (Optional)
1. Identify the number of containers created in the `red` pod. ⇒ **`3`**
	```javascript
root@controlplane ~ ➜  kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
app         0/1     ContainerCreating   0          42s
fluent-ui   0/1     ContainerCreating   0          42s
red         0/3     ContainerCreating   0          26s
	```
2. Identify the name of the containers running in the `blue` pod. ⇒ **`teal & navy`**
	```javascript
root@controlplane ~ ➜  kubectl describe pod blue
Name:         blue
Namespace:    default
Priority:     0
Node:         controlplane/10.137.165.3
Start Time:   Fri, 02 Sep 2022 01:10:24 +0000
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:           
IPs:          <none>
Containers:
  teal:
    Container ID:  
    Image:         busybox
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r242z (ro)
  navy:
    Container ID:  
    Image:         busybox
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sleep
      4500
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-r242z (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-r242z:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  44s   default-scheduler  Successfully assigned default/blue to controlplane
  Normal  Pulling    40s   kubelet            Pulling image "busybox"
	```
3. Create a multi-container pod with 2 containers. Use the spec given below. If the pod goes into the `crashloopbackoff` then add the command `sleep 1000` in the `lemon` container.
	- Name: yellow
	- Container 1 Name: lemon
	- Container 1 Image: busybox
	- Container 2 Name: gold
	- Container 2 Image: redis
	```javascript
apiVersion: v1
kind: Pod
metadata:
  name: yellow
spec:
  containers:
  - name: lemon
    image: busybox
    command:
      - sleep
      - "1000"

  - name: gold
    image: redis
	```
4. We have deployed an application logging stack in the `elastic-stack` namespace. Inspect it. Before proceeding with the next set of questions, please wait for all the pods in the `elastic-stack` namespace to be ready. This can take a few minutes.
	![]()
5. We will configure a sidecar container for the application to send logs to Elastic Search.NOTE: It can take a couple of minutes for the `Kibana` UI to be ready after the `Kibana` pod is ready. You can inspect the `Kibana` logs by running:`kubectl -n elastic-stack logs kibana`.
	<details>
	<summary>logs</summary>
		```javascript
root@controlplane ~ ✖ kubectl -n elastic-stack logs kibana
{"type":"log","@timestamp":"2022-09-02T01:12:40Z","tags":["status","plugin:kibana@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:40Z","tags":["status","plugin:elasticsearch@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:40Z","tags":["status","plugin:xpack_main@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:40Z","tags":["status","plugin:searchprofiler@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:40Z","tags":["status","plugin:ml@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:41Z","tags":["status","plugin:tilemap@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:41Z","tags":["status","plugin:watcher@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:41Z","tags":["status","plugin:license_management@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:41Z","tags":["status","plugin:index_management@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["status","plugin:timelion@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["status","plugin:graph@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["status","plugin:monitoring@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["security","warning"],"pid":1,"message":"Generating a random key for xpack.security.encryptionKey. To prevent sessions from being invalidated on restart, please set xpack.security.encryptionKey in kibana.yml"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["security","warning"],"pid":1,"message":"Session cookies will be transmitted over insecure connections. This is not recommended."}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["status","plugin:security@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:42Z","tags":["status","plugin:grokdebugger@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:dashboard_mode@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:logstash@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:apm@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:console@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:console_extensions@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:notifications@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:43Z","tags":["status","plugin:metrics@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from uninitialized to green - Ready","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["reporting","warning"],"pid":1,"message":"Generating a random key for xpack.reporting.encryptionKey. To prevent pending reports from failing on restart, please set xpack.reporting.encryptionKey in kibana.yml"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:reporting@6.4.2","info"],"pid":1,"state":"yellow","message":"Status changed from uninitialized to yellow - Waiting for Elasticsearch","prevState":"uninitialized","prevMsg":"uninitialized"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["error","elasticsearch","admin"],"pid":1,"message":"Request error, retrying\nHEAD http://elasticsearch:9200/ => connect ECONNREFUSED 10.104.211.75:9200"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:xpack_main@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:searchprofiler@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:ml@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:tilemap@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:watcher@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:index_management@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:graph@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:security@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:grokdebugger@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:logstash@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:reporting@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:elasticsearch@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from yellow to red - Request Timeout after 3000ms","prevState":"yellow","prevMsg":"Waiting for Elasticsearch"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["error","elasticsearch","data"],"pid":1,"message":"Request error, retrying\nGET http://elasticsearch:9200/_xpack => connect ECONNREFUSED 10.104.211.75:9200"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["warning","elasticsearch","data"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["warning","elasticsearch","data"],"pid":1,"message":"No living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["license","warning","xpack"],"pid":1,"message":"License information from the X-Pack plugin could not be obtained from Elasticsearch for the [data] cluster. Error: No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:xpack_main@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:searchprofiler@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:ml@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:tilemap@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:watcher@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:index_management@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:graph@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:security@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:grokdebugger@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:logstash@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:49Z","tags":["status","plugin:reporting@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"No living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:xpack_main@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:searchprofiler@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:ml@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:tilemap@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:watcher@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:index_management@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:graph@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:security@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:grokdebugger@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:logstash@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:reporting@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:elasticsearch@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - Unable to connect to Elasticsearch at http://elasticsearch:9200/.","prevState":"red","prevMsg":"Request Timeout after 3000ms"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["warning","elasticsearch","data"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["warning","elasticsearch","data"],"pid":1,"message":"No living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["license","warning","xpack"],"pid":1,"message":"License information from the X-Pack plugin could not be obtained from Elasticsearch for the [data] cluster. Error: No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:xpack_main@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:searchprofiler@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:ml@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:tilemap@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:watcher@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:index_management@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:graph@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:security@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:grokdebugger@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:logstash@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:52Z","tags":["status","plugin:reporting@6.4.2","error"],"pid":1,"state":"red","message":"Status changed from red to red - No Living connections","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:12:56Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:12:56Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"No living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:01Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"Unable to revive connection: http://elasticsearch:9200/"}
{"type":"log","@timestamp":"2022-09-02T01:13:01Z","tags":["warning","elasticsearch","admin"],"pid":1,"message":"No living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:11Z","tags":["status","plugin:elasticsearch@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"Unable to connect to Elasticsearch at http://elasticsearch:9200/."}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["license","info","xpack"],"pid":1,"message":"Imported license information from Elasticsearch for the [data] cluster: mode: basic | status: active"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:xpack_main@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:searchprofiler@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:ml@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:tilemap@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:watcher@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:index_management@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:graph@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:grokdebugger@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:logstash@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:reporting@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["info","monitoring-ui","kibana-monitoring"],"pid":1,"message":"Starting monitoring stats collection"}
{"type":"log","@timestamp":"2022-09-02T01:13:13Z","tags":["status","plugin:security@6.4.2","info"],"pid":1,"state":"green","message":"Status changed from red to green - Ready","prevState":"red","prevMsg":"No Living connections"}
{"type":"log","@timestamp":"2022-09-02T01:13:14Z","tags":["license","info","xpack"],"pid":1,"message":"Imported license information from Elasticsearch for the [monitoring] cluster: mode: basic | status: active"}
{"type":"log","@timestamp":"2022-09-02T01:13:47Z","tags":["info","http","server","listening"],"pid":1,"message":"Server running at http://0:5601"}
{"type":"response","@timestamp":"2022-09-02T01:13:48Z","tags":[],"pid":1,"method":"head","statusCode":200,"req":{"url":"/","method":"head","headers":{"host":"0.0.0.0:30601","user-agent":"curl/7.58.0","accept":"*/*"},"remoteAddress":"10.244.0.1","userAgent":"10.244.0.1"},"res":{"statusCode":200,"responseTime":107,"contentLength":9},"message":"HEAD / 200 107ms - 9.0B"}
{"type":"response","@timestamp":"2022-09-02T01:13:58Z","tags":[],"pid":1,"method":"post","statusCode":200,"req":{"url":"/api/saved_objects/index-pattern/filebeat-*?overwrite=false","method":"post","headers":{"host":"0.0.0.0:30601","user-agent":"curl/7.58.0","accept":"*/*","content-type":"application/json","kbn-xsrf":"athing","content-length":"66"},"remoteAddress":"10.244.0.1","userAgent":"10.244.0.1"},"res":{"statusCode":200,"responseTime":2862,"contentLength":9},"message":"POST /api/saved_objects/index-pattern/filebeat-*?overwrite=false 200 2862ms - 9.0B"}
{"type":"response","@timestamp":"2022-09-02T01:14:01Z","tags":[],"pid":1,"method":"post","statusCode":200,"req":{"url":"/api/kibana/settings/defaultIndex","method":"post","headers":{"host":"0.0.0.0:30601","user-agent":"curl/7.58.0","accept":"*/*","content-type":"application/json","kbn-xsrf":"anytng","content-length":"22"},"remoteAddress":"10.244.0.1","userAgent":"10.244.0.1"},"res":{"statusCode":200,"responseTime":2197,"contentLength":9},"message":"POST /api/kibana/settings/defaultIndex 200 2197ms - 9.0B"}
		```
	</details>
6. Inspect the `app` pod and identify the number of containers in it. It is deployed in the `elastic-stack` namespace. ⇒ **`1`**
	```javascript
root@controlplane ~ ➜  kubectl describe pod app -n elastic-stack
Name:         app
Namespace:    elastic-stack
Priority:     0
Node:         controlplane/10.137.165.3
Start Time:   Fri, 02 Sep 2022 01:09:35 +0000
Labels:       name=app
Annotations:  <none>
Status:       Running
IP:           10.244.0.4
IPs:
  IP:  10.244.0.4
Containers:
  app:
    Container ID:   docker://17a82674c4333e957b5c11a78ffeb3938b5d592303140b3c32530a53f4c50566
    Image:          kodekloud/event-simulator
    Image ID:       docker-pullable://kodekloud/event-simulator@sha256:1e3e9c72136bbc76c96dd98f29c04f298c3ae241c7d44e2bf70bcc209b030bf9
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Fri, 02 Sep 2022 01:09:56 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /log from log-volume (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-hxnnh (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  log-volume:
    Type:          HostPath (bare host directory volume)
    Path:          /var/log/webapp
    HostPathType:  DirectoryOrCreate
  kube-api-access-hxnnh:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  8m42s  default-scheduler  Successfully assigned elastic-stack/app to controlplane
  Normal  Pulling    8m37s  kubelet            Pulling image "kodekloud/event-simulator"
  Normal  Pulled     8m29s  kubelet            Successfully pulled image "kodekloud/event-simulator" in 7.720993844s
  Normal  Created    8m25s  kubelet            Created container app
  Normal  Started    8m21s  kubelet            Started container app
	```
7. The application outputs logs to the file `/log/app.log`. View the logs and try to identify the user having issues with Login. Inspect the log file inside the pod. ⇒ **`USER5`**
	```javascript
$ kubectl -n elastic-stack exec -it app -- cat /log/app.log

[2022-09-02 01:19:42,316] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-09-02 01:19:42,316] INFO in event-simulator: USER4 is viewing page2
[2022-09-02 01:19:42,466] INFO in event-simulator: USER1 is viewing page2
[2022-09-02 01:19:43,318] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-09-02 01:19:43,318] INFO in event-simulator: USER1 is viewing page1
[2022-09-02 01:19:43,467] INFO in event-simulator: USER4 logged out
[2022-09-02 01:19:44,366] INFO in event-simulator: USER4 logged out
[2022-09-02 01:19:44,468] INFO in event-simulator: USER2 is viewing page1
	```
8. Edit the pod to add a sidecar container to send logs to Elastic Search. Mount the log volume to the sidecar container. Only add a new container. Do not modify anything else. Use the spec provided below.
	- Name: app
	- Container Name: sidecar
	- Container Image: kodekloud/filebeat-configured
	- Volume Mount: log-volume
	- Mount Path: /var/log/event-simulator/
	- Existing Container Name: app
	- Existing Container Image: kodekloud/event-simulator
	```javascript
apiVersion: v1
kind: Pod
metadata:
  name: app
  namespace: elastic-stack
  labels:
    name: app
spec:
  containers:
  - name: app
    image: kodekloud/event-simulator
    volumeMounts:
    - mountPath: /log
      name: log-volume

  - name: sidecar
    image: kodekloud/filebeat-configured
    volumeMounts:
    - mountPath: /var/log/event-simulator/
      name: log-volume

  volumes:
  - name: log-volume
    hostPath:
      # directory location on host
      path: /var/log/webapp
      # this field is optional
      type: DirectoryOrCreate
	```
9. Inspect the Kibana UI. You should now see logs appearing in the `Discover` section. You might have to wait for a couple of minutes for the logs to populate. You might have to create an index pattern to list the logs. If not sure check this video: `https://bit.ly/2EXYdHf`
	⇒ Go to Kibaba UI.
	![]()
	![]()
	![]()
	- Is that you have to create an index pattern and combine a user’s index patterns to retrieve data from Elasticsearch. In this case, Basically we’re just going to get everything.
	![]()
	![]()
	![]()
	- So, these are the logs that are coming through.
# 111. Multi-container PODs Design Patterns
There are 3 common patterns, when it comes to designing multi-container PODs. The first and what we just saw with the logging service example is known as a side car pattern. The others are the adapter and the ambassador pattern.
But these fall under the CKAD curriculum and are not required for the CKA exam. So we will be discuss these in more detail in the CKAD course.
![](https://img-c.udemycdn.com/redactor/raw/2019-06-07_09-07-13-d077fffd07f3232ae259a9298c0ffa66.PNG)
# 112. InitContainers
In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.
But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only  one time when the pod is first created. Or a process that waits  for an external service or database to be up before the actual application starts. That's where **initContainers **comes in.
An **initContainer **is configured in a pod like all other containers, except that it is specified inside a `initContainers` section,  like this:`<br>1. apiVersion: v1<br>2. kind: Pod<br>3. metadata:<br>4.   name: myapp-pod<br>5.   labels:<br>6.     app: myapp<br>7. spec:<br>8.   containers:<br>9.   - name: myapp-container<br>10.     image: busybox:1.28<br>11.     command: ['sh', '-c', 'echo The app is running! && sleep 3600']<br>12.   initContainers:<br>13.   - name: init-myservice<br>14.     image: busybox<br>15.     command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ; done;']`
When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts.
You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run **one at a time in sequential order**.
If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.`<br>1. apiVersion: v1<br>2. kind: Pod<br>3. metadata:<br>4.   name: myapp-pod<br>5.   labels:<br>6.     app: myapp<br>7. spec:<br>8.   containers:<br>9.   - name: myapp-container<br>10.     image: busybox:1.28<br>11.     command: ['sh', '-c', 'echo The app is running! && sleep 3600']<br>12.   initContainers:<br>13.   - name: init-myservice<br>14.     image: busybox:1.28<br>15.     command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']<br>16.   - name: init-mydb<br>17.     image: busybox:1.28<br>18.     command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']`
Read more about initContainers here. And try out the upcoming practice test.
[https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
# 113. Practice Test - Init Containers
Practice Test Link: [https://uklabs.kodekloud.com/topic/practice-test-init-containers-2/](https://uklabs.kodekloud.com/topic/practice-test-init-containers-2/)
# 114. Solution - Init Containers (Optional)
1. Identify the pod that has an `initContainer` configured. ⇒ **`blue`**
	```javascript
controlplane ~ ➜  kubectl describe pods
Name:         red
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.24
Start Time:   Fri, 02 Sep 2022 07:21:44 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.9
IPs:
  IP:  10.42.0.9
Containers:
  red-container:
    Container ID:  containerd://21774882e9815a7047d58b308b52ec02d37fa31b5305db1beec4af26789632e7
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Running
      Started:      Fri, 02 Sep 2022 07:21:47 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dp6cr (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-dp6cr:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  31s   default-scheduler  Successfully assigned default/red to controlplane
  Normal  Pulling    30s   kubelet            Pulling image "busybox:1.28"
  Normal  Pulled     30s   kubelet            Successfully pulled image "busybox:1.28" in 719.049093ms
  Normal  Created    30s   kubelet            Created container red-container
  Normal  Started    29s   kubelet            Started container red-container


Name:         green
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.24
Start Time:   Fri, 02 Sep 2022 07:21:44 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.10
IPs:
  IP:  10.42.0.10
Containers:
  green-container-1:
    Container ID:  containerd://a2bfdf93454bf17c0afee51698e313ab221b60d3cb7f6b73fc8a5b89b2b880fa
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Running
      Started:      Fri, 02 Sep 2022 07:21:48 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2qj46 (ro)
  green-container-2:
    Container ID:  containerd://8681667b9ac0965f32919dfabb144d17b862b67f179e7ef5aaa165c3a2750d91
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Running
      Started:      Fri, 02 Sep 2022 07:21:49 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-2qj46 (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-2qj46:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  31s   default-scheduler  Successfully assigned default/green to controlplane
  Normal  Pulling    30s   kubelet            Pulling image "busybox:1.28"
  Normal  Pulled     30s   kubelet            Successfully pulled image "busybox:1.28" in 684.379195ms
  Normal  Created    29s   kubelet            Created container green-container-1
  Normal  Started    28s   kubelet            Started container green-container-1
  Normal  Pulled     28s   kubelet            Container image "busybox:1.28" already present on machine
  Normal  Created    28s   kubelet            Created container green-container-2
  Normal  Started    27s   kubelet            Started container green-container-2


Name:         blue
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.24
Start Time:   Fri, 02 Sep 2022 07:21:44 +0000
Labels:       <none>
Annotations:  <none>
Status:       Running
IP:           10.42.0.11
IPs:
  IP:  10.42.0.11
Init Containers:
  init-myservice:
    Container ID:  containerd://dc446b5bcd0061b087b9ccd47c5227646e833bbad3d6e253bf5c613b96e0aa2f
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:20142e89dab967c01765b0aea3be4cec3a5957cc330f061e5503ef6168ae6613
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 5
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Fri, 02 Sep 2022 07:21:49 +0000
      Finished:     Fri, 02 Sep 2022 07:21:54 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jjz7r (ro)
Containers:
  green-container-1:
    Container ID:  containerd://4718f6aec5eef4161300adf063bd0c4a06a1f75345ff8e486edcdc096d5d148d
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Running
      Started:      Fri, 02 Sep 2022 07:21:55 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-jjz7r (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-jjz7r:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  31s   default-scheduler  Successfully assigned default/blue to controlplane
  Normal  Pulling    30s   kubelet            Pulling image "busybox"
  Normal  Pulled     28s   kubelet            Successfully pulled image "busybox" in 2.688520996s
  Normal  Created    27s   kubelet            Created container init-myservice
  Normal  Started    27s   kubelet            Started container init-myservice
  Normal  Pulled     22s   kubelet            Container image "busybox:1.28" already present on machine
  Normal  Created    22s   kubelet            Created container green-container-1
  Normal  Started    21s   kubelet            Started container green-container-1
	```
2. What is the image used by the `initContainer` on the `blue` pod? ⇒ **`busybox`**
3. What is the state of the `initContainer` on pod `blue`? ⇒ **`Terminated`**
4. Why is the `initContainer` terminated? What is the reason? ⇒ **`The process completed successfully`**
5. We just created a new app named `purple`. How many `initContainers` does it have? ⇒ **`2`**
	```javascript
controlplane ~ ✖ kubectl describe pod purple 
Name:         purple
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.24
Start Time:   Fri, 02 Sep 2022 07:26:53 +0000
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:           10.42.0.12
IPs:
  IP:  10.42.0.12
Init Containers:
  warm-up-1:
    Container ID:  containerd://33f045e4542d8f876409ac938ee0ed2a39c5a9f5c59c28b6eeded27de378e661
    Image:         busybox:1.28
    Image ID:      docker.io/library/busybox@sha256:141c253bc4c3fd0a201d32dc1f493bcf3fff003b6df416dea4f41046e0f37d47
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 600
    State:          Running
      Started:      Fri, 02 Sep 2022 07:26:55 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-655f2 (ro)
  warm-up-2:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleep 1200
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-655f2 (ro)
Containers:
  purple-container:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-655f2 (ro)
Conditions:
  Type              Status
  Initialized       False 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-655f2:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  25s   default-scheduler  Successfully assigned default/purple to controlplane
  Normal  Pulled     24s   kubelet            Container image "busybox:1.28" already present on machine
  Normal  Created    24s   kubelet            Created container warm-up-1
  Normal  Started    23s   kubelet            Started container warm-up-1
	```
6. What is the state of the POD? ⇒ **`Pending`**
7. How long after the creation of the POD will the application come up and be available to users? ⇒ **`30 Minutes`**
8. Update the pod `red` to use an `initContainer` that uses the `busybox` image and `sleeps for 20` seconds. Delete and re-create the pod if necessary. But make sure no other configurations change.
	- Pod: red
	- initContainer Configured Correctly
	```javascript
apiVersion: v1
kind: Pod
metadata:
  name: red
  namespace: default
spec:
  containers:
  - command:
    - sh
    - -c
    - echo The app is running! && sleep 3600
    image: busybox:1.28
    name: red-container
  initContainers:
  - image: busybox
    name: red-initcontainer
    command: 
      - "sleep"
      - "20"
	```
9. A new application `orange` is deployed. There is something wrong with it. Identify and fix the issue. Once fixed, wait for the application to run before checking solution.
	(Hint) There is a typo in the command used by the initContainer. To fix this, first get the pod definition file by running `kubectl get pod orange -o yaml > /root/orange.yaml`. Next, edit the command and fix the typo. Then, delete the old pod by running `kubectl delete pod orange` Finally, create the pod again by running `kubectl create -f /root/orange.yaml`.
	```javascript
controlplane ~ ➜  kubectl get pods
NAME     READY   STATUS                  RESTARTS      AGE
green    2/2     Running                 0             11m
blue     1/1     Running                 0             11m
purple   0/1     Init:0/2                0             6m8s
orange   0/1     Init:CrashLoopBackOff   1 (11s ago)   14s
red      1/1     Running                 0             25s

controlplane ~ ➜  kubectl describe pod orange 
Name:         orange
Namespace:    default
Priority:     0
Node:         controlplane/172.25.0.24
Start Time:   Fri, 02 Sep 2022 07:32:47 +0000
Labels:       <none>
Annotations:  <none>
Status:       Pending
IP:           10.42.0.14
IPs:
  IP:  10.42.0.14
Init Containers:
  init-myservice:
    Container ID:  containerd://c02ee3f5e96e226d9f93cb607227b27bf26c53db80d776c5190970f5c5c43da3
    Image:         busybox
    Image ID:      docker.io/library/busybox@sha256:20142e89dab967c01765b0aea3be4cec3a5957cc330f061e5503ef6168ae6613
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      sleeeep 2;
    State:          Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 02 Sep 2022 07:33:03 +0000
      Finished:     Fri, 02 Sep 2022 07:33:03 +0000
    Last State:     Terminated
      Reason:       Error
      Exit Code:    127
      Started:      Fri, 02 Sep 2022 07:32:50 +0000
      Finished:     Fri, 02 Sep 2022 07:32:50 +0000
    Ready:          False
    Restart Count:  2
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xlttj (ro)
Containers:
  orange-container:
    Container ID:  
    Image:         busybox:1.28
    Image ID:      
    Port:          <none>
    Host Port:     <none>
    Command:
      sh
      -c
      echo The app is running! && sleep 3600
    State:          Waiting
      Reason:       PodInitializing
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-xlttj (ro)
Conditions:
  Type              Status
  Initialized       False 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  kube-api-access-xlttj:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  28s                default-scheduler  Successfully assigned default/orange to controlplane
  Normal   Pulled     27s                kubelet            Successfully pulled image "busybox" in 500.253686ms
  Normal   Pulled     25s                kubelet            Successfully pulled image "busybox" in 497.301972ms
  Normal   Pulling    13s (x3 over 27s)  kubelet            Pulling image "busybox"
  Normal   Pulled     13s                kubelet            Successfully pulled image "busybox" in 506.277072ms
  Normal   Created    13s (x3 over 27s)  kubelet            Created container init-myservice
  Normal   Started    12s (x3 over 26s)  kubelet            Started container init-myservice
  Warning  BackOff    12s (x3 over 25s)  kubelet            Back-off restarting failed container
	```
# 115. Self Healing Applications
Kubernetes supports self-healing applications through ReplicaSets and Replication Controllers. The replication controller helps in ensuring that a POD is re-created automatically when the application within the POD crashes. It helps in ensuring enough replicas of the application are running at all times.
Kubernetes provides additional support to check the health of applications running within PODs and take necessary actions through Liveness and Readiness Probes. However these are not required for the CKA exam and as such they are not covered here. These are topics for the Certified Kubernetes Application Developers (CKAD) exam and are covered in the CKAD course.
# 116. If you like it, Share it!
Hope you are enjoying this course. If you like it share it in your community. Here's a twitter template:
[https://ctt.ac/nc_f6](https://ctt.ac/nc_f6)
