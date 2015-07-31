# kubernetes-elk-cluster
**ELK** (**Elasticsearch** + **Logstash** + **Kibana**) cluster on top of **Kubernetes** made easy.

Here you will find:
* Kubernetes pod descriptor that joins Elasticsearch load-balancer container with Logstash container (for `localhost` communication)
* Kubernetes pod descriptor that joins Elasticsearch load-balancer container with Kibana container (for `localhost` communication)
* Kubernetes service descriptor that publishes Logstash listening for Lumberjack protocol
* Kubernetes service descriptor that publishes Kibana webpage

**Attention:**
* If you're looking for details on how `pires/elasticsearch` images are built, take a look at [my Elasticsearch repository](https://github.com/pires/kubernetes-elasticsearch-cluster).
* If you're looking for details on how `quay.io/pires/docker-logstash` image is built, take a look at [my Logstash repository](https://github.com/pires/docker-logstash).
* If you're looking for details on how `quay.io/pires/docker-logstash-forwarder` image is built, take a look at [my docker-logstash-forwarder repository](https://github.com/pires/docker-logstash-forwarder).

## Pre-requisites

* Kubernetes cluster (tested with 2 minions [Vagrant + CoreOS](https://github.com/pires/kubernetes-vagrant-coreos-cluster))
* `kubectl` configured to access your cluster master API Server
* Elasticsearch cluster deployed - you can skip deploying `load-balancers`provisioning, since those will be paired with Logstash and Kibana containers, and automatically join the cluster you've assembled with [my Elasticsearch cluster instructions](https://github.com/pires/kubernetes-elasticsearch-cluster)).

### SSL certificates

Be sure to provide valid SSL certificates for `logstash` and `logstash-forwarder`, by changing the `hostDir` path to whatever folder you will be storing the certificates. Otherwise, **this won't work**! I know `hostPath` is not the best solution but I leave up to you how to provide the `certs` directory to the containers.

Changes must be performed in `logstash-controller.yaml` and `logstash-forwarder-controller.yaml`, **at least** for the `certs` volume. Look for
```yaml
volumes:
...
- name: certs
  source:
    hostPath:
      path: /tmp
```

`hostPath.path` is what you need to change.

### Tailored configuration

If you want to change `logstash` and `logstash-forwarder` configuration, be sure to add to each `replication-controller`
```yaml
volumeMounts:
- mountPath: /logstash/config
  name: config

(...)

volumes:
- name: config
  source:
    hostPath:
      path: /path/to/config
```

and change accordingly.

## Deploy

```
kubectl create -f logstash-service.yaml
kubectl create -f logstash-controller.yaml
kubectl create -f kibana-service.yaml
kubectl create -f kibana-controller.yaml
```

Wait for provisioning to happen and then check the status:

```
kubectl get pods

TODO PUT HERE OUTPUT

```

As you can assert, the cluster is up and running. Easy, wasn't it?

## Access the service

*Don't forget* that services in Kubernetes are only acessible from containers within the cluster.

```
kubectl get service kibana

TODO PUT HERE OUTPUT

```