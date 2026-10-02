# 81. Logging and Monitoring Section Introduction
- In this section, we discussed about the various logging and monitoring options available. We first see how to monitor the Kubernetes Cluster components as well as the applications hosted on them. We then see how to view and manage the logs for the Cluster components as well as the applications. We practice what we learn through a set of fun and challenging exercises.
# 82. Download Presentation Deck
Please note that some slides are animated so content may not have exported correctly. Kindly use the slides as a reference for commands.
### 이 강의 자료
<file src=""></file>
# 83. Monitor Cluster Components
![]()
- In this lecture, we talk about Monitoring a Kubernetes Cluster. How do you monitor resource consumption on Kubernetes? Or more importantly what would you like to monitor? I’d like to know Node level metrics such as the number of Nodes in the Cluster, how many of them are healthy as well as performance metrics such as CPU, Memory, Network and Disk Utilization.
![]()
- As well as Pod level metrics such as the number of Pods, and performance metrics of each Pod such as the CPU and Memory consumption on them.
![]()
- So we need a solution that will monitor these metrics store them and provide analytics around this data.
![]()
- As of this recording, Kubernetes does not come with a full featured build-in monitoring solution. 
![]()
- However, there are a number of open-source solutions available today, such as the Metrics-Server, Prometheus, Elastic Stack, and proprietary solutions like Datadog and Dynatrace.
![]()
- Heapster was one of the original projects that enabled monitoring and analysis features for Kubernetes . You will see a lot of reference online when you like for reference architectures on monitoring Kubernetes.
![]()
- However, Heapster is now Deprecated and a slimmed down version was formed known as the Metrics Server.
![]()
- You can have 1 Matrics Server per Kubernetes Cluster. 
![]()
- The Metrics Server retrieves Metrcis from each of the Kubernetes Nodes and Pods, aggregates them and stores them in memory. 
![]()
- Note that the Metric Server is only an in memory monitoring solution and does not store the Metrics on the desk and as a result you cannot see historical performance data. For that you must rely on one of the advanced monitoring solutions we talked about earlier in this lecture.
### 🔰 How are the MetrIcs generated for the Pods on these Nodes?
![]()
- Kubernetes runs an agent on each Node known as the `kubelet`, which is responsible for receiving instructions from the Kubernetes API Master Server and running Pods on the Nodes.
![]()
- The `kubelet` also contains a subcomponent known as a cAdvisor or Container Advisor. 
![]()
- cAdvisor is responsible for retrieving performance Metrics from Pods, and exposing them through the `kubelet` API to make the Metrics available for the Metrics Server. 
![]()
- If you are using Minikube for your local Cluster, run the command `minikube addons enable metrics-server`. 
![]()
- For all other environments deploy the Metrics Server by cloning the Metrics-Server deployment files from the github repository. 
![]()
- And then deploying the required components, using the `kubectl create` command. This command deploys a set of Pods, Services and roles to enable Metrics Server to poll for performance Metrics from the Nodes in the Cluster.
![]()
- Once deployed, give the Metrics Server some time to collect and process data. Once processed, cluster performance can be viewed by running the command `kubectl top node`. This procides the CPU and Memory consumption of each of the Nodes. As you can see 8% of the CPU on my master Node is consumed, which is about 166 milli cores. 
![]()
- Use the `kubectl top pod` command to view performance Metrics of Pods in Kubernetes.
# 84. Practice Test - Monitoring
Note: In this test you will enable cluster monitoring. Once you do remember to wait for atleast 5 minutes to allow the metrics-server enough time to collect and report performance metrics.
Practice Test - [https://uklabs.kodekloud.com/topic/practice-test-monitor-cluster-components-2/](https://uklabs.kodekloud.com/topic/practice-test-monitor-cluster-components-2/)
# 85. Solution: Monitor Cluster Components : (Optional)
1. Let us deploy metrics-server to monitor the PODs and Nodes. Pull the git repository for the deployment files.
	Run: `git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git`
	```shell
root@controlplane ~ ➜  git clone https://github.com/kodekloudhub/kubernetes-metrics-server.git
Cloning into 'kubernetes-metrics-server'...
remote: Enumerating objects: 24, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 24 (delta 4), reused 0 (delta 0), pack-reused 12
Unpacking objects: 100% (24/24), done.
	```
2. Deploy the metrics-server by creating all the components downloaded. Run the `kubectl create -f .` command from within the downloaded repository.
	```shell
root@controlplane kubernetes-metrics-server on  master ✖ kubectl create -f metrics-server-deployment.yaml
serviceaccount/metrics-server created
deployment.apps/metrics-server created
	```
3. It takes a few minutes for the metrics server to start gathering data. Run the `kubectl top node` command and wait for a valid output.
4. Identify the node that consumes the `most` CPU. ⇒ **`node01`**
	```shell
root@controlplane:~# kubectl top node --sort-by='cpu' --no-headers | head -1                   
node01         234m   2%    515Mi   2%   
	```
5. Identify the node that consumes the `most` Memory. ⇒ **`controlplane`**
	```shell
root@controlplane:~# kubectl top node --sort-by='memory' --no-headers | head -1 
controlplane   183m   2%    662Mi   3%
	```
6. Identify the POD that consumes the `most` Memory. ⇒ **`rabbit`**
	```shell
root@controlplane:~# kubectl top pod --sort-by='memory' --no-headers | head -1 
rabbit     134m   201Mi
	```
7. Identify the POD that consumes the `least` CPU. ⇒ **`lion`**
	```shell
root@controlplane:~# kubectl top pod --sort-by='cpu' --no-headers | tail -1 
lion       2m     5Mi
	```
# 86. Managing Application Logs
- We will talk about various Logging mechanisms in Kubernetes. Let us start with logging in Docker. 
![]()
- I run  a Docker container called ‘event-simulator’ and all that it does is generate random events simulating a Web Server. These are events streamed to the standard output by the application. 
![]()
- Now, If I were to run the Docker Container in the background, in a detached mode using the -d option, I wouldn’t see the logs. 
![]()
- If I wanted to view the logs, I could use the `docker logs` command followed by the Container ID. The -f option helps us see the live log trail.
![]()
- Back to Kubernetes. We create a Pod with the same Docker Image using the pod-definition file. Once It’s the Pod is running, we can view the logs using the `kubectl logs` command with the Pod name. Use the ‘-f’ option to stream the logs live just like the Docker Command. 
- Now these logs are specific to the container running inside the Pod. As we learned before, Kubernetes Pods can have multiple Docker Container in them. 
![]()
- In this case, I modify my pod-definition file to include an additional Container called image-processor. If you run the `kubectl logs` command, with the Pod name, which Container’s log would it show? If there are multiple Containers within a Pod, you must specify the name of the Container explicitly in the command. Otherwise It would fail asking you to specify a name.
![]()
- In this case I will specify the name of the first Container event-simulator and that prints the relevant log messages. Now, that is the simple logging functionality implemented within Kubernetes. And that is all that an application developer really needs to know to get started with Kubernetes. And that is all you really need to know as part of the certification program.
- However in the next lecture, we will see more about some advanced logging configuration and third party support for logging in Kubernetes. We also have a nice demo that shows how a popular logging framework is integrated with Kubernetes.
# 87. Practice Test - Monitor Application Logs
Practice Test - [https://uklabs.kodekloud.com/topic/practice-test-managing-application-logs-2/](https://uklabs.kodekloud.com/topic/practice-test-managing-application-logs-2/)
# 88. Solution: Logging : (Optional)
1. We have deployed a POD hosting an application. Inspect it. Wait for it to start.
	```shell
controlplane ~ ➜  kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS      RESTARTS   AGE
kube-system   local-path-provisioner-6c79684f77-4bd5f   1/1     Running     0          5m23s
kube-system   coredns-5789895cd-rhfw7                   1/1     Running     0          5m23s
kube-system   metrics-server-7cd5fcb6b7-78kkk           1/1     Running     0          5m23s
kube-system   helm-install-traefik-crd-pfcr6            0/1     Completed   0          5m23s
kube-system   helm-install-traefik-76xvn                0/1     Completed   1          5m23s
kube-system   svclb-traefik-kgk9z                       2/2     Running     0          3m38s
kube-system   traefik-6bb96f9bd8-jz4s2                  1/1     Running     0          3m38s
default       webapp-1                                  1/1     Running     0          34s
	```
2. A user - `USER5` - has expressed concerns accessing the application. Identify the cause of the issue. Inspect the logs of the POD. ⇒ **`Account Locked due to Many Failed Attempts`**
	```shell
controlplane ~ ✖ kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          2m12s

controlplane ~ ➜  kubectl logs webapp-1
[2022-08-26 08:06:55,352] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:06:56,353] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:06:57,354] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:06:58,356] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:06:59,357] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:07:00,358] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:00,358] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:07:01,359] INFO in event-simulator: USER3 logged out
[2022-08-26 08:07:02,360] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:03,361] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:03,362] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:04,363] INFO in event-simulator: USER3 logged in
[2022-08-26 08:07:05,363] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:05,364] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:06,365] INFO in event-simulator: USER2 logged in
[2022-08-26 08:07:07,366] INFO in event-simulator: USER3 logged in
[2022-08-26 08:07:08,367] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:09,394] INFO in event-simulator: USER4 logged in
[2022-08-26 08:07:10,395] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:10,395] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:07:11,397] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:11,397] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:07:12,398] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:13,399] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:07:14,400] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:07:15,400] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:15,400] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:07:16,401] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:07:17,403] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:18,403] INFO in event-simulator: USER2 logged out
[2022-08-26 08:07:19,404] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:19,404] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:07:20,404] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:20,405] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:07:21,406] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:07:22,407] INFO in event-simulator: USER1 logged in
[2022-08-26 08:07:23,408] INFO in event-simulator: USER2 logged in
[2022-08-26 08:07:24,409] INFO in event-simulator: USER2 logged out
[2022-08-26 08:07:25,410] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:25,411] INFO in event-simulator: USER4 logged in
[2022-08-26 08:07:26,411] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:07:27,412] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:27,412] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:07:28,414] INFO in event-simulator: USER2 logged in
[2022-08-26 08:07:29,416] INFO in event-simulator: USER2 logged out
[2022-08-26 08:07:30,416] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:30,416] INFO in event-simulator: USER3 logged in
[2022-08-26 08:07:31,417] INFO in event-simulator: USER4 logged in
[2022-08-26 08:07:32,419] INFO in event-simulator: USER3 logged out
[2022-08-26 08:07:33,420] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:34,421] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:07:35,423] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:35,423] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:35,423] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:07:36,424] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:07:37,425] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:07:38,427] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:07:39,428] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:07:40,429] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:40,430] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:41,431] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:07:42,432] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:43,433] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:43,434] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:07:44,435] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:07:45,436] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:45,436] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:07:46,437] INFO in event-simulator: USER1 logged in
[2022-08-26 08:07:47,439] INFO in event-simulator: USER1 logged in
[2022-08-26 08:07:48,440] INFO in event-simulator: USER1 logged in
[2022-08-26 08:07:49,441] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:07:50,443] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:50,443] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:07:51,443] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:51,443] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:07:52,445] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:07:53,446] INFO in event-simulator: USER1 logged in
[2022-08-26 08:07:54,446] INFO in event-simulator: USER3 logged in
[2022-08-26 08:07:55,447] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:07:55,447] INFO in event-simulator: USER1 logged out
[2022-08-26 08:07:56,449] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:57,450] INFO in event-simulator: USER4 logged out
[2022-08-26 08:07:58,451] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:07:59,452] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:07:59,452] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:08:00,453] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:00,454] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:01,455] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:08:02,456] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:08:03,457] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:08:04,459] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:08:05,459] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:05,460] INFO in event-simulator: USER2 logged out
[2022-08-26 08:08:06,460] INFO in event-simulator: USER4 logged in
[2022-08-26 08:08:07,461] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:07,461] INFO in event-simulator: USER1 logged in
[2022-08-26 08:08:08,463] INFO in event-simulator: USER3 logged in
[2022-08-26 08:08:09,464] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:08:10,465] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:10,466] INFO in event-simulator: USER4 logged in
[2022-08-26 08:08:11,467] INFO in event-simulator: USER4 logged out
[2022-08-26 08:08:12,468] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:13,469] INFO in event-simulator: USER3 logged out
[2022-08-26 08:08:14,470] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:15,471] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:15,471] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:15,472] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:08:16,473] INFO in event-simulator: USER2 logged out
[2022-08-26 08:08:17,474] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:18,474] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:08:19,475] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:08:20,476] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:20,476] INFO in event-simulator: USER2 logged out
[2022-08-26 08:08:21,477] INFO in event-simulator: USER2 logged out
[2022-08-26 08:08:22,478] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:23,480] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:23,480] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:08:24,481] INFO in event-simulator: USER1 logged in
[2022-08-26 08:08:25,482] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:25,482] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:08:26,483] INFO in event-simulator: USER3 logged in
[2022-08-26 08:08:27,483] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:08:28,484] INFO in event-simulator: USER2 logged out
[2022-08-26 08:08:29,485] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:30,486] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:30,486] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:31,486] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:31,487] INFO in event-simulator: USER4 logged out
[2022-08-26 08:08:32,487] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:08:33,488] INFO in event-simulator: USER3 logged in
[2022-08-26 08:08:34,489] INFO in event-simulator: USER3 logged out
[2022-08-26 08:08:35,490] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:35,491] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:36,491] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:08:37,492] INFO in event-simulator: USER4 logged in
[2022-08-26 08:08:38,493] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:08:39,495] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:39,495] INFO in event-simulator: USER1 logged out
[2022-08-26 08:08:40,496] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:40,496] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:08:41,497] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:08:42,499] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:08:43,500] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:08:44,501] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:08:45,502] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:45,502] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:08:46,503] INFO in event-simulator: USER4 logged in
[2022-08-26 08:08:47,504] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:47,504] INFO in event-simulator: USER4 logged in
[2022-08-26 08:08:48,505] INFO in event-simulator: USER3 logged out
[2022-08-26 08:08:49,506] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:08:50,507] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:50,507] INFO in event-simulator: USER3 logged in
[2022-08-26 08:08:51,508] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:08:52,509] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:08:53,510] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:08:54,512] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:08:55,512] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:08:55,572] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:08:55,572] INFO in event-simulator: USER3 logged in
[2022-08-26 08:08:56,573] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:08:57,575] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:08:58,576] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:08:59,577] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:09:00,579] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:00,579] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:09:01,580] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:09:02,582] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:09:03,583] WARNING in event-simulator: USER7 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:09:03,583] INFO in event-simulator: USER1 logged out
[2022-08-26 08:09:04,584] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:09:05,585] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:05,585] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:09:06,587] INFO in event-simulator: USER1 logged in
[2022-08-26 08:09:07,587] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:09:08,589] INFO in event-simulator: USER2 is viewing page2
	```
3. We have deployed a new POD - `webapp-2`  - hosting an application. Inspect it. Wait for it to start.
	```shell
controlplane ~ ➜  kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
webapp-1   1/1     Running   0          3m21s
webapp-2   2/2     Running   0          42s
	```
4. A user is reporting issues while trying to purchase an item. Identify the user and the cause of the issue. Inspect the logs of the webapp in the POD. ⇒ **`USER30 - Item Out of Stock`**
	![]()
	```shell
controlplane ~ ✖ kubectl logs webapp-2 -c simple-webapp
[2022-08-26 08:09:31,648] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:09:32,649] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:09:33,650] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:09:34,650] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:09:35,652] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:09:36,653] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:36,653] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:09:37,654] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:09:38,654] INFO in event-simulator: USER3 logged out
[2022-08-26 08:09:39,655] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:09:39,655] INFO in event-simulator: USER4 logged out
[2022-08-26 08:09:40,656] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:09:41,657] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:41,657] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:09:42,658] INFO in event-simulator: USER2 logged in
[2022-08-26 08:09:43,659] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:09:44,659] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:09:45,661] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:09:46,661] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:46,662] INFO in event-simulator: USER2 logged out
[2022-08-26 08:09:47,663] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:09:47,663] INFO in event-simulator: USER3 logged out
[2022-08-26 08:09:48,664] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:09:49,665] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:09:50,667] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:09:51,668] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:51,668] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:09:52,669] INFO in event-simulator: USER1 logged in
[2022-08-26 08:09:53,671] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:09:54,671] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:09:55,673] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:09:55,673] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:09:56,674] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:09:56,674] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:09:57,674] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:09:58,676] INFO in event-simulator: USER4 logged out
[2022-08-26 08:09:59,677] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:10:00,679] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:10:01,680] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:01,680] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:10:02,681] INFO in event-simulator: USER4 logged out
[2022-08-26 08:10:03,695] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:03,695] INFO in event-simulator: USER4 logged out
[2022-08-26 08:10:04,696] INFO in event-simulator: USER3 logged in
[2022-08-26 08:10:05,696] INFO in event-simulator: USER3 logged out
[2022-08-26 08:10:06,697] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:06,698] INFO in event-simulator: USER1 logged in
[2022-08-26 08:10:07,699] INFO in event-simulator: USER2 logged out
[2022-08-26 08:10:08,700] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:10:09,701] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:10:10,702] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:10:11,717] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:11,717] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:11,717] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:10:12,717] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:10:13,719] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:10:14,720] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:10:15,721] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:10:16,723] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:16,723] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:10:17,724] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:10:18,725] INFO in event-simulator: USER1 logged in
[2022-08-26 08:10:19,726] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:19,726] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:10:20,727] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:10:21,728] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:21,728] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:10:22,729] INFO in event-simulator: USER1 logged in
[2022-08-26 08:10:23,730] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:10:24,732] INFO in event-simulator: USER2 logged out
[2022-08-26 08:10:25,733] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:10:26,734] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:26,735] INFO in event-simulator: USER3 is viewing page3
[2022-08-26 08:10:27,735] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:27,735] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:10:28,736] INFO in event-simulator: USER2 logged in
[2022-08-26 08:10:29,737] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:10:30,739] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:10:31,740] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:31,740] INFO in event-simulator: USER2 logged in
[2022-08-26 08:10:32,741] INFO in event-simulator: USER3 logged in
[2022-08-26 08:10:33,743] INFO in event-simulator: USER4 logged out
[2022-08-26 08:10:34,744] INFO in event-simulator: USER3 logged out
[2022-08-26 08:10:35,745] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:35,746] INFO in event-simulator: USER3 logged out
[2022-08-26 08:10:36,746] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:36,746] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:10:37,747] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:10:38,749] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:10:39,749] INFO in event-simulator: USER3 logged in
[2022-08-26 08:10:40,751] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:10:41,752] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:41,752] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:10:42,753] INFO in event-simulator: USER4 is viewing page2
[2022-08-26 08:10:43,754] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:43,754] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:10:44,755] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:10:45,757] INFO in event-simulator: USER4 logged in
[2022-08-26 08:10:46,757] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:46,758] INFO in event-simulator: USER2 is viewing page3
[2022-08-26 08:10:47,759] INFO in event-simulator: USER3 logged out
[2022-08-26 08:10:48,760] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:10:49,762] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:10:50,763] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:10:51,764] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:51,764] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:51,764] INFO in event-simulator: USER3 is viewing page1
[2022-08-26 08:10:52,765] INFO in event-simulator: USER3 logged out
[2022-08-26 08:10:53,767] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:10:54,768] INFO in event-simulator: USER1 logged in
[2022-08-26 08:10:55,769] INFO in event-simulator: USER2 is viewing page1
[2022-08-26 08:10:56,771] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:10:56,771] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:10:57,772] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:10:58,773] INFO in event-simulator: USER1 logged out
[2022-08-26 08:10:59,775] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:10:59,775] INFO in event-simulator: USER1 is viewing page2
[2022-08-26 08:11:00,776] INFO in event-simulator: USER1 logged in
[2022-08-26 08:11:01,777] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:11:01,778] INFO in event-simulator: USER2 logged out
[2022-08-26 08:11:02,778] INFO in event-simulator: USER4 logged in
[2022-08-26 08:11:03,778] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:11:04,780] INFO in event-simulator: USER2 logged out
[2022-08-26 08:11:05,781] INFO in event-simulator: USER2 is viewing page2
[2022-08-26 08:11:06,782] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:11:06,782] INFO in event-simulator: USER1 is viewing page3
[2022-08-26 08:11:07,783] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:11:07,783] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:11:08,784] INFO in event-simulator: USER4 is viewing page3
[2022-08-26 08:11:09,785] INFO in event-simulator: USER1 logged in
[2022-08-26 08:11:10,786] INFO in event-simulator: USER1 logged out
[2022-08-26 08:11:11,787] WARNING in event-simulator: USER5 Failed to Login as the account is locked due to MANY FAILED ATTEMPTS.
[2022-08-26 08:11:11,787] INFO in event-simulator: USER1 is viewing page1
[2022-08-26 08:11:12,789] INFO in event-simulator: USER4 is viewing page1
[2022-08-26 08:11:13,789] INFO in event-simulator: USER2 logged out
[2022-08-26 08:11:14,790] INFO in event-simulator: USER3 is viewing page2
[2022-08-26 08:11:15,791] WARNING in event-simulator: USER30 Order failed as the item is OUT OF STOCK.
[2022-08-26 08:11:15,792] INFO in event-simulator: USER4 is viewing page1
	```
