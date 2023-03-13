FROM wurstmeister/kafka:latest

ADD kafka-jmx-exporter-config.yml /opt/kafka/libs/jmx.yml

# RUN wget https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.17.2/jmx_prometheus_javaagent-0.17.2.jar
ADD jmx_prometheus_javaagent_0.17.2.jar /opt/kafka/libs/jmx_prometheus_javaagent.jar

RUN chmod +r /opt/kafka/libs/jmx_prometheus_javaagent.jar

EXPOSE 7071