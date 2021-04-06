# Elastic observability for Kubernetes

These labs guide you through the Elastic Observability functionality. A sample application is installed on a Kubernetes cluster and connected to an Elastic Observability deployment. The set-up is then used to go through the most important parts of the Elastic Observability product.

Labs:

- Lab 0: Elastic Observability trial instance
- Lab 1: Installation sample application
- Lab 2: Log files
- Lab 3: Metrics
- Lab 4: Application Performance monitoring


## Pre-requisites

For these labs, it is assumed that you have a Kubernetes cluster at your disposal. The labs have been tested on a Minikube cluster, running on Ubuntu.

Intallation instructions for Minikube can be found [here](https://minikube.sigs.k8s.io/docs/start/).


## Running Elastic Observability locally

The Elastic Observability trial is a great way to become acquainted with the product. The obvious drawback is that it has a limited trail period of 14 days.

Should you want to, it is also possible to run the Elastic Observability stack locally, i.e. on the Kubernetes cluster that is also used to run the sample application on. 

More detailed instructions on how to do that can be found in:

- Appendix: Running Elastic Observability locally

=========================================================
=========================================================
=========================================================
=========================================================
=========================================================


# Appendix
Release: 7.10.1

# 1. Configuration

Core monitoring platform:
- Elastic
- Kibana
- APM Server

Data agents:
- filebeat
- metricbeat
- APM instrumentation

Not:
- Logstash

## 1.9 clean out log files in /var/og/containers

## 1.10 minikube

- memory: 10240
- cpu: 6

```bash
developer@developer-VirtualBox:~$ minikube config set cpus 6
❗  These changes will take effect upon a minikube delete and then a minikube start
developer@developer-VirtualBox:~$ minikube config set memory 10240
❗  These changes will take effect upon a minikube delete and then a minikube start
developer@developer-VirtualBox:~$
```

Should you run into swap space issues:

Use less swap (only when needed - swappines)

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ sudo sysctl vm.swappiness=10
[sudo] password for developer: 
vm.swappiness = 10
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$
```

Larger swap file:

```bash
developer@developer-VirtualBox:~$ du -m /swapfile 
2049	/swapfile
developer@developer-VirtualBox:~$ sudo swapoff /swapfile
[sudo] password for developer: 
developer@developer-VirtualBox:~$ sudo rm /swapfile 
developer@developer-VirtualBox:~$ sudo dd if=/dev/zero of=/swapfile bs=1M count=20480
20480+0 records in
20480+0 records out
21474836480 bytes (21 GB, 20 GiB) copied, 32,9329 s, 652 MB/s
developer@developer-VirtualBox:~$ sudo chmod 600 /swapfile 
developer@developer-VirtualBox:~$ sudo mkswap /swapfile 
Setting up swapspace version 1, size = 20 GiB (21474832384 bytes)
no label, UUID=4209a5be-ec58-4d9d-9663-4beb9f3efc90
developer@developer-VirtualBox:~$ sudo swapon /swapfile
developer@developer-VirtualBox:~$ swapon -s
Filename				Type		Size	Used	Priority
/swapfile                              	file    	20971516	0	-2
developer@developer-VirtualBox:~$ du -m /swapfile 
20481	/swapfile
developer@developer-VirtualBox:~$
```
And re-boot



Start minikube:

```bash
developer@developer-VirtualBox:~$ minikube start
😄  minikube v1.17.1 on Ubuntu 18.04 (vbox/amd64)
✨  Using the docker driver based on existing profile
👍  Starting control plane node minikube in cluster minikube
🔄  Restarting existing docker container for "minikube" ...
🐳  Preparing Kubernetes v1.20.2 on Docker 20.10.2 ...
🔎  Verifying Kubernetes components...
🌟  Enabled addons: storage-provisioner, pod-security-policy, default-storageclass, metrics-server
🏄  Done! kubectl is now configured to use "minikube" cluster and "default" namespace by default
developer@developer-VirtualBox:~$ 
```










# 2 Intallation

## 2.1 Elasticsearch


```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm repo add elastic https://helm.elastic.co
"elastic" has been added to your repositories
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "elastic" chart repository
Update Complete. ⎈Happy Helming!⎈
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm install  elasticsearch --values values-es.yaml elastic/elasticsearch
NAME: elasticsearch
LAST DEPLOYED: Tue Feb 16 19:34:02 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Watch all cluster members come up.
  $ kubectl get pods --namespace=default -l app=elasticsearch-master -w
2. Test cluster health using Helm test.
  $ helm test elasticsearch
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ 
```

## 2.2 Kibana

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm install kibana --values values-kibana.yaml elastic/kibana
NAME: kibana
LAST DEPLOYED: Tue Feb 16 19:52:20 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$
```

## 2.3 metricbeat for sample dashboards

Download metricbeat and untar:

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ curl -L -O https://artifacts.elastic.co/downloads/beats/metricbeat/metricbeat-7.10.1-linux-x86_64.tar.gz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 39.0M  100 39.0M    0     0  3314k      0  0:00:12  0:00:12 --:--:-- 3750k
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ tar xzvf metricbeat-7.10.1-linux-x86_64.tar.gz 
metricbeat-7.10.1-linux-x86_64/fields.yml
metricbeat-7.10.1-linux-x86_64/modules.d/
metricbeat-7.10.1-linux-x86_64/modules.d/activemq.yml.disabled
metricbeat-7.10.1-linux-x86_64/modules.d/aerospike.yml.disabled
...
```

Two terminal windows with forwards:

Kibana:
```bash
developer@developer-VirtualBox:~$ kubectl port-forward services/kibana-kibana 5601:5601
Forwarding from 127.0.0.1:5601 -> 5601
Forwarding from [::1]:5601 -> 5601
```

Elasticsearch:
```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ kubectl port-forward service/elasticsearch-master  9200:9200
Forwarding from 127.0.0.1:9200 -> 9200
Forwarding from [::1]:9200 -> 9200
```

Load dashboards (this will take a while):

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation/metricbeat-7.10.1-linux-x86_64$ ./metricbeat setup -e
...
```

## 2.4 APM Server

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm install apm-server --values values-apm.yaml elastic/apm-server 
NAME: apm-server
LAST DEPLOYED: Tue Feb 16 21:44:35 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all containers come up.
  $ kubectl get pods --namespace=default -l app=apm-server-apm-server -w
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$
```

## 2.5 Metricbeat

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm install metricbeat --values values-metricbeat.yaml elastic/metricbeat
NAME: metricbeat
LAST DEPLOYED: Tue Feb 16 21:48:41 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all containers come up.
  $ kubectl get pods --namespace=default -l app=metricbeat-metricbeat -w
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$
```

## 2.6 Filebeat

```bash
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$ helm install filebeat --values values-filebeat.yaml elastic/filebeat
NAME: filebeat
LAST DEPLOYED: Tue Feb 16 21:59:16 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
1. Watch all containers come up.
  $ kubectl get pods --namespace=default -l app=filebeat-filebeat -w
developer@developer-VirtualBox:~/projects/elastic-observability-labs/0_installation$
```


## 2.7 Filesystem fixkubec

[todo: put this fix in the right place: is it immediately after elasticsearch helm install?]

```bash
root@developer-VirtualBox:/tmp/hostpath-provisioner# ls -al
total 20
drwxr-xr-x  5 root root 4096 feb 17 19:33 .
drwxrwxrwt 16 root root 4096 feb 17 19:33 ..
drwxr-xr-x  2 root root 4096 feb 17 19:33 pvc-8b0b80c7-7085-11eb-bed4-08002729a4cd
drwxr-xr-x  2 root root 4096 feb 17 19:33 pvc-8b0cfcec-7085-11eb-bed4-08002729a4cd
drwxr-xr-x  2 root root 4096 feb 17 19:33 pvc-8b0ff535-7085-11eb-bed4-08002729a4cd
root@developer-VirtualBox:/tmp/hostpath-provisioner# sudo chown -R 1000:1000 *
root@developer-VirtualBox:/tmp/hostpath-provisioner# ll
total 20
drwxr-xr-x  5 root      root      4096 feb 17 19:33 ./
drwxrwxrwt 16 root      root      4096 feb 17 19:34 ../
drwxr-xr-x  2 developer developer 4096 feb 17 19:33 pvc-8b0b80c7-7085-11eb-bed4-08002729a4cd/
drwxr-xr-x  2 developer developer 4096 feb 17 19:33 pvc-8b0cfcec-7085-11eb-bed4-08002729a4cd/
drwxr-xr-x  2 developer developer 4096 feb 17 19:33 pvc-8b0ff535-7085-11eb-bed4-08002729a4cd/
root@developer-VirtualBox:/tmp/hostpath-provisioner#
```

# Instrumentation


## say-hello-node (NodeJS)

npm install elastic-apm-node --save



## say-hello-micronaut (Micronaut)


## say-hello-springboot (SpringBoot)


## say-hello-elastic (NodeJS)

npm install elastic-apm-node --save

const apm = require('elastic-apm-node').start({
    serviceName: 'say-hello-elastic',
    secretToken: '',
    apiKey: '',
    serverUrl: 'http://apm-server-apm-server:8200',
  })






# Notes

Data in:
./var/lib/docker/volumes/minikube/_data/hostpath-provisioner/default/elasticsearch-master-elasticsearch-master-0/nodes/0/lgo-touch




