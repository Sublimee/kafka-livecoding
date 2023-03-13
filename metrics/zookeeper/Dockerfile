FROM wurstmeister/zookeeper:latest

ADD zookeeper-jmx-exporter-config.yml /opt/zookeeper-3.4.13/lib/jmx.yml

# RUN wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.17.2/jmx_prometheus_javaagent-0.17.2.jar
ADD jmx_prometheus_javaagent_0.17.2.jar /opt/zookeeper-3.4.13/lib/jmx_prometheus_javaagent.jar

RUN chmod +r /opt/zookeeper-3.4.13/lib/jmx_prometheus_javaagent.jar

EXPOSE 7072