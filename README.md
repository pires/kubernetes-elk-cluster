# kubernetes-elk-cluster
**ELK** (**Elasticsearch** + **Logstash** + **Kibana**) cluster on top of **Kubernetes** made easy.

Here you will find:
* Kubernetes pod descriptor that joins Elasticsearch load-balancer container with Logstash container (for ```localhost``` communication)
* Kubernetes pod descriptor that joins Elasticsearch load-balancer container with Kibana container (for ```localhost``` communication)
* Kubernetes service descriptor that publishes Logstash listening for Lumberjack protocol
* Kubernetes service descriptor that publishes Kibana webpage

**Attention:** 
* If you're looking for details on how ```pires/elasticsearch``` images are built, take a look at [my Elasticsearch repository](https://github.com/pires/kubernetes-elasticsearch-cluster).
* If you're looking for details on how ```pires/logstash``` image is built, take a look at [my Logstash repository](https://github.com/pires/logstash).
* If you're looking for details on how ```pires/kibana``` image is built, take a look at [my Kibana repository](https://github.com/pires/kibana).
* If you're looking for details on how ```pires/logstash-forwarder``` image is built, take a look at [my logstash-forwarder repository](https://github.com/pires/logstash-forwarder).

## Pre-requisites

* Kubernetes cluster (tested with 3 nodes [Vagrant + CoreOS](https://github.com/pires/kubernetes-vagrant-coreos-cluster))
* ```kubectl``` configured to access your cluster master API Server
* Elasticsearch cluster deployed - you can skip deploying ```load-balancers```provisioning, since those will be paired with Logstash and Kibana containers, and automatically join the cluster you've assembled with [my Elasticsearch cluster instructions](https://github.com/pires/kubernetes-elasticsearch-cluster)).

## Deploy

```
kubectl create -f logstash-service.json
kubectl create -f logstash-controller.json
kubectl create -f logstash-forwarder-controller.json
kubectl create -f kibana-service.json
kubectl create -f kibana-controller.json
```

## Validate

I leave to you the steps to validate the provisioned pods, but first step is to wait for containers to be in ```RUNNING``` state and check the logs of the master (as in Elasticsearch):

```
kubectl get pods
```

You should see something like this:

```
POD                                    IP                  CONTAINER(S)           IMAGE(S)                     HOST                LABELS                                       STATUS
61f5768d-a94d-11e4-9459-0800272d7481   10.244.95.2         elasticsearch-master   pires/elasticsearch:master   172.17.8.102/       component=elasticsearch,role=master          Running
c74f218e-a94e-11e4-9459-0800272d7481   10.244.92.2         elasticsearch-lb       pires/elasticsearch:lb       172.17.8.103/       component=elasticsearch,role=load-balancer   Running
c7c6e8ce-a94e-11e4-9459-0800272d7481   10.244.95.3         elasticsearch-data     pires/elasticsearch:data     172.17.8.102/       component=elasticsearch,role=data            Running
a96d1b26-a95d-11e4-9459-0800272d7481   10.244.92.6         elasticsearch          pires/elasticsearch:lb       172.17.8.103/       component=elasticsearch,role=logstash        Running
                                                           logstash               pires/logstash
fa58e4c5-a961-11e4-9459-0800272d7481   10.244.26.2         elasticsearch          pires/elasticsearch:lb       172.17.8.104/       component=elasticsearch,role=kibana          Running
                                                           kibana                 pires/kibana
3e7aa0fb-aee1-11e4-a06e-0800272d7481   10.244.102.5        logstash-forwarder     pires/logstash-forwarder     172.17.8.102/       component=logstash-forwarder                 Running
```

As you can assert, the cluster is up and running. Easy, wasn't it?

### Scale

Scaling each type of node to handle your cluster is as easy as:

```
kubectl resize --replicas=3 replicationcontrollers logstash
kubectl resize --replicas=2 replicationcontrollers kibana
```

## Access the service

*Don't forget* that services in Kubernetes are only acessible from containers in the cluster. For different behavior you should configure ```publicIps``` or ```createExternalLoadBalancer```, in your service. That's out of scope of this document, for now.

```
kubectl get service kibana
```

You should see something like this:

```
NAME                LABELS              SELECTOR                              IP                  PORT
kibana              <none>              component=elasticsearch,role=kibana   10.244.63.95        80
```

From inside one of the containers running in your cluster:

```
curl http://pires:kibanarulz@10.244.63.95
```

This should be what you see:

```html
<!DOCTYPE html><!--[if lt IE 7]>
<html class="no-js lt-ie9 lt-ie8 lt-ie7">
  <![endif]--><!--[if IE 7]>
  <html class="no-js lt-ie9 lt-ie8">
    <![endif]--><!--[if IE 8]>
    <html class="no-js lt-ie9">
      <![endif]--><!--[if gt IE 8]><!-->
      <html class="no-js">
        <!--<![endif]-->
        <head>
          <meta charset="utf-8">
          <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
          <meta name="viewport" content="width=device-width">
          <title>Kibana 3{{dashboard.current.title ? " - "+dashboard.current.title : ""}}</title>
          <link rel="stylesheet" href="css/bootstrap.light.min.css" title="Light">
          <link rel="stylesheet" href="css/timepicker.css">
          <link rel="stylesheet" href="css/animate.min.css">
          <link rel="stylesheet" href="css/normalize.min.css">
          <script src="vendor/require/require.js"></script><script src="app/components/require.config.js"></script><script>require(['app'], function () {})</script>
          <style></style>
        </head>
        <body>
          <noscript>
            <div class="container">
              <center>
                <h3>You must enable javascript to use Kibana</h3>
              </center>
            </div>
          </noscript>
          <link rel="stylesheet" ng-href="css/bootstrap.{{dashboard.current.style||'dark'}}.min.css">
          <link rel="stylesheet" href="css/bootstrap-responsive.min.css">
          <link rel="stylesheet" href="css/font-awesome.min.css">
          <div ng-cloak="" ng-repeat="alert in dashAlerts.list" class="alert-{{alert.severity}} dashboard-notice" ng-show="$last">
            <button type="button" class="close" ng-click="dashAlerts.clear(alert)" style="padding-right:50px">&times;</button> <strong>{{alert.title}}</strong> <span ng-bind-html="alert.text"></span>
            <div style="padding-right:10px" class="pull-right small">{{$index + 1}} alert(s)</div>
          </div>
          <div ng-cloak="" class="navbar navbar-static-top">
            <div class="navbar-inner">
              <div class="container-fluid">
                <span class="brand"><img src="img/small.png" bs-tooltip="'Kibana '+(kbnVersion=='@REV@'?'master':kbnVersion)" data-placement="bottom"> {{dashboard.current.title}}</span>
                <ul class="nav pull-right" ng-controller="dashLoader" ng-init="init()" ng-include="'app/partials/dashLoader.html'"></ul>
              </div>
            </div>
          </div>
          <div ng-cloak="" ng-view=""></div>
        </body>
      </html>
```