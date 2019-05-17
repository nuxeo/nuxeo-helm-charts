# Nuxeo Helm Chart
/!\ NOT PRODUCTION READY, TESTING PURPOSE ONLY /!\

>WARNING
The `nuxeo` chart deploys additional external services (`Mongo`, `Postgresql`, `ES`, etc), but is not a production ready chart.  The main goal of this chart is to be used a base chart when deploying preview and staging envs on JenkinsX. The external services referenced by this helm chart have often another version more suitable for production ( for example Redis vs Redis-HA charts, Mongodb vs Mongodb-Replicaset). Also persistence has been disabled by default on all subcharts.
## Dependencies
 
 This chart has the following dependencies:
 - [`Postgresql`](https://github.com/helm/charts/blob/master/stable/postgresql/values.yaml) 
 - [`Elasticsearch`](https://github.com/helm/charts/blob/master/stable/elasticsearch/values.yaml) 
 - [`MongoDB `](https://github.com/helm/charts/blob/master/stable/mongodb/values.yaml) 
 -  [`Redis`](https://github.com/helm/charts/blob/master/stable/redis/values.yaml)
 -  [`Kafka/Zookeeper cluster`](https://github.com/helm/charts/blob/master/incubator/kafka/values.yaml)

In order to list the dependencies:
```console
 $helm dependency list nuxeo
```
 
### How to enable/disable dependencies and run Nuxeo with Mongodb, Postgresql, Kafka, etc

- When the chart is **deployed directly**, these dependencies are controlled by the following tags set in the `values.yaml` file:
```code
tags:
  mongodb: false
  redis: false
  kafka: false
  elasticsearch: false
  postgresql: false
```
In order to enable any of these, just pass the corresponding tag as true:
```console
helm install  nuxeo  --set tags.mongodb=true ## deploys Mongodb
```
- When the chart is **deployed as a dependency to another chart**, these dependencies are controlled by setting the variables `nuxeo.mongodb.deploy`/ `nuxeo.elasticsearch.deploy`/ etc to `true`. 
For example when a chart is using `nuxeo` as a dependency ( this happens when we deploy a `preview` chart on JenkinsX):
```code
Dependencies:
  - name: nuxeo
    version: 0.1.0
    repository: http://jenkins-x-chartmuseum:8080
    alias: nuxeo
```
We can enable Mongodb, Elasticsearch and any of the other supported sub-charts by using the following in its values.yaml:
```code
nuxeo:
  mongodb:
    deploy: true 
  elasticsearch:
    deploy: true
```

## Installing the Chart directly
If you are also deploying any of the dependency charts, make sure you download them before: 

```console
 $helm dependency update  nuxeo
```
Deploy the chart:
```console
helm install nuxeo --name $my_release --namespace $your_namespace
```
Override chart values:
- You can override any of the values from the base `values.yaml`. You can create your own `myValues.yaml` and pass it directly at install as:

```console
helm install -f myvalues.yaml nuxeo --name $my_release --namespace $your_namespace
```
 - You can also override default values by passing them directly to the install or upgrade commands
 ```console
helm install nuxeo --set nuxeo.image.tag=10.3 --name $my_release --namespace $your_namespace
```
Register the instance and deploy a custom mp:
```console
helm install nuxeo --set nuxeo.packages=nuxeo-web-ui,nuxeo.studio_project=$your_project,nuxeo.connect_username=$connect_username,nuxeo.connect_password=$connect_password
```

or:
```console
helm install nuxeo --set nuxeo.packages=nuxeo-web-ui,nuxeo.clid=$your_clid
```

- You can override any values of the sub-charts in the same name, just by using the sub-chart name as a prefix first 
 ```console
helm install nuxeo --set mongodb.persistence.enabled=true --name $my_release --namespace $your_namespace
```

## Upgrading an existing deployment
Use `helm upgrade`: 
- Example: upgrade the existing deployment to enable persistence for the binaries and logs:

```console
 helm upgrade  $my_release --set nuxeo.persistence.enabled=true nuxeo/ 
```

## See the installed templates
```console
 helm get manifest $my-release
```

## Uninstalling the Chart
```console
helm del --purge $my-release
```

## Using minikube

### Install minikube and helm

Follow the [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)
and [helm](https://helm.sh/docs/using_helm/#install-helm) installation guides.

Below is the procedure for Ubuntu 18.04:
```bash
# Check vm support, the following command must output something
egrep --color 'vmx|svm' /proc/cpuinfo

# Update apt
sudo apt update && sudo apt upgrade
sudo apt install -y apt-transport-https

# Install latest virtualbox deb from
https://www.virtualbox.org/wiki/Linux_Downloads

# Install kubectl
sudo snap install kubectl --classic
kubectl version
#  -> 1.14.1

# Install minikube
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
chmod +x minikube
sudo mv minikube /usr/local/bin

# Install helm
sudo snap install helm --classic

```
### Init minikube and helm

```bash
# start with a bigger VM
minikube start --cpus 4 --memory 8192 --disk-size 10g

# enable some addons
minikube addons enable ingress
minikube addons enable storage-provisioner

# init helm
helm init --history-max 200

# enable incubator repo needed for Kafka
helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator

helm repo update

# test the dashboard
minikube dashboard
```

Then try to deploy a nuxeo:

```bash
helm install \
 --name my-nuxeo \
 --debug \
 --set nuxeo.packages=nuxeo-web-ui \
 --set tags.mongodb=true \
 --set tags.elasticsearch=true,elasticsearch.deploy=true,nuxeo.elasticsearch.deploy=true \
 --set nuxeo.ingress.enabled=true \
 --set nuxeo.clid='<insert the content of your instance.clid file in a single line replacing the new line with --' \
  nuxeo
```

Nuxeo will be exposed on `http://$(minikube ip)/`


# About Nuxeo

Nuxeo dramatically improves how content-based applications are built, managed and deployed, making customers more agile, innovative and successful. Nuxeo provides a next generation, enterprise ready platform for building 
traditional and cutting-edge content oriented applications. Combining a powerful application development environment with SaaS-based tools and a modular architecture, the Nuxeo Platform and Products provide clear business 
value to some of the most recognizable brands including Verizon, Electronic Arts, Sharp, FICO, the U.S. Navy, and Boeing. Nuxeo is headquartered in New York and Paris. More information is available at www.nuxeo.com.

