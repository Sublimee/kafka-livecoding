FROM wurstmeister/kafka:2.13-2.8.1

ADD kafka-jmx-exporter-config.yml /opt/kafka/libs/jmx.yml

# RUN wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.18.0/jmx_prometheus_javaagent-0.18.0.jar
ADD jmx_prometheus_javaagent-0.18.0.jar /opt/kafka/libs/jmx_prometheus_javaagent.jar

RUN chmod +r /opt/kafka/libs/jmx_prometheus_javaagent.jar

EXPOSE 7071