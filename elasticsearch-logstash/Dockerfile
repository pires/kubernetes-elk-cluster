# Ubuntu 14.04.x + Oracle JRE 8
FROM pires/elasticsearch-kubernetes:lb-latest

MAINTAINER pjpires@gmail.com

# Export Lumberjack and ES Transport
EXPOSE 5043 9300

RUN \
  cd / && \
  curl -O https://download.elasticsearch.org/logstash/logstash/logstash-1.4.2.tar.gz && \
  tar zxf logstash-1.4.2.tar.gz && \
  mv logstash-1.4.2 /logstash && \
  rm -f logstash-1.4.2.tar.gz

# Logstash config
ADD logstash.conf /logstash/conf/logstash.conf

# Certificates for logstash-forwarders
ADD certs/logstash-forwarder.crt /logstash/certs/logstash-forwarder.crt
ADD certs/logstash-forwarder.key /logstash/certs/logstash-forwarder.key

# runnable script
ADD run /
RUN chmod +x /run

CMD ["/run"]