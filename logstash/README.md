
# docker-logstash
Lean (315MB) and highly configurable Logstash Docker image, based on `gliderlabs/alpine`.

[![Docker Repository on Quay.io](https://quay.io/repository/pires/docker-logstash/status "Docker Repository on Quay.io")](https://quay.io/repository/pires/docker-logstash)

I made this so that I could [easily cluster ELK on top of Kubernetes](https://github.com/pires/kubernetes-elk-cluster), and so, by default it will be listening for the [lumberjack](http://logstash.net/docs/1.4.2/inputs/lumberjack) protocol with certificates provisioned in a mounted directory, `/logstash/certs`.

## Current software

* Oracle JRE 8 Update 51
* Logstash 1.5.3
