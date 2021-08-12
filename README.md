# Ansible for Apollo Kafka Platform

Ansible playbooks to install kafka cluster and zookeeper ensemble for Apollo Kafka Platform.

This project is composed of two different playbooks, the first being used to install the kafka cluster and zookeeper ensemble installation. 
The second playbook descibes the monitor/manager machine used to monitor the kafka platform.

<img src="img/dia.png" width="500">




---
# Kafka and Zookeeper installation
---
## This playbook will perform the following:

1. Install and configure the zookeeper ensemble
2. Install and configure the kafka cluster
3. Install agents in both kafka and zookeeper needed for monitoring and management
4. Install zookeeper service and run Zookeeper
5. Install kafka service and run kafka

---
## Platform requirement
    * Ubuntu 20.04.x

---
## Components that will be installed on the nodes

Kafka nodes will be installed with the following components:
1. Apache Kafka, kafka_2.13-2.8.0 
2. Prometheus Agent for Kafka, JMX Exporter Agent for Kafka, jmx_prometheus_javaagent-0.16.0.jar
3. Jolokia Agent for apache kafka, jolokia-jvm-1.6.2
4. Java Development Kit, openjdk-11
5. Prometheus Node Exporter, node_exporter-1.1.2.linux-amd64


Zookeeper nodes will be installed with the following components:
1. Apache Zookeeper, apache-zookeeper-3.7.0 
2. Java Development Kit, openjdk-11
3. Prometheus Node Exporter, node_exporter-1.1.2.linux-amd64
   
---

### SSH connectivity
Make sure to connect as root via SSH passwordless. The steps on setting this up is beyond the scope of this document.


## Pre-configuration requirements

1. Update *inventory/hosts* file to reflect the target platform:

- [kakfa] group, the ip address/es of all kafka brokers to be installed
- [zookeeper] group, the ip address/es of all zookeepers to be installed
- [manager] group, the ip address of the management machine for the kafka platform

2. Update *vars/settings.yaml* file and update the lines

    ``` 
    kafka_install_package_source_folder: /path/to/kafka/package
    zookeeper_install_package_source_folder: /path/to/zookeeper/package
    ``` 

- kafka_install_package_source_folder, the folder that contains the kafka_2.13-2.8.0.tgz file
- zookeeper_install_package_source_folder, the folder that contains the apache-zookeeper-3.7.0-bin.tar.gz file 

   > Other config parameters for the kafka cluster can also be set in this file.


---
### Zookeeper ansible configuration

### Kafka ansible configuraiton

To adjust the **server.properties** settings, edit the ./vars/settings.yaml file
Follow the pattern: the prefix "kafka_" + original property with the character "." (dot) edited to "_" (undeline).
Example:
    server.properties
        auto.create.topics.enable
    settings.yaml
        kafka_auto_create_topics_enable 
        
There are 2  gropus of properties for **server.properties** in the file.

- KAFKA server.properties [1]: Common and default properties of
  server.properties original.
  
- KAFKA server.properties [2]: Extra settings for server.prperties. The
  relation of all properties can be visualized in the [kafka
  documentation](https://kafka.apache.org/documentation/).  In this 2nd group the properties are in a list called kafka_properties with 2 attributes:   
	- **key**: The name of property
	- **value**: The value of property. If the value is none the property is not included in server.properties.        

---
## Running the ansible playbook

    > $ ansible-playbook playbooks/install-kafka-zk.yaml 

---
## Testing setup
(Optional steps. These are performed in the post installation test.)

### Testing kafka

Optional steps that can be taken to make sure that the services are running.

1. SSH to kafka machine
2. run 
   > $ systemctl status kafka

Check individually whether zookeeper service is running.
1. SSH to zookeeper machine
2. run
    > $ systemctl status zookeeper 
   
### Testing agents on Kafka machines 
*Check monitoring agents are running on kafka machines*
For kafka machines, the following ports should have been opened: (in addition to kafka port, and other defaults)
    8004, 7000, 7001
    
1. Check JMX_PORT used by (to be) CMAK (kafka manager): curl to the machine at port 8004
    > $ curl <machine>:8004 
2. Prometheus agent for kafka (port 7000)
3. Prometheus agent for the node (port 7001)

### Testing Zookeeper
Check individually whether zookeeper service is running. 
1. SSH to zookeeper machine
2. run 
   > $ systemctl status zookeeper

### Testing agents on Zookeeper machines 
*Check monitoring agents are running on zookeeper machines*
For zookeeper machines, the following ports should have been opened: (in addition to zookeeper port, and other defaults)
    7000, 7001

1. Prometheus agent of zookeeper (7000)
2. Prometheus agent for the node (port 7001)



---
# Monitoring Tools installation

CMAK is a tool for managing kafka cluster. 

---
## This playbook will perform the following:
Install and configure CMAK (Cluster Manager for Apache Kafka, previously known as Kafka Manager)

---
## Platform requirement
    * Ubuntu 20.04.x

---
## Components that will be installed on the nodes

Monitor machine will be installed with the following components:
1. Java Development Kit, openjdk-11
2. Prometheus Node Exporter, node_exporter-1.1.2.linux-amd64
   
---

### SSH connectivity
Make sure to connect as root via SSH passwordless. The steps on setting this up is beyond the scope of this document.

## Pre-configuration requirements
Before running the ansible script, edit *templates/application.conf* file and update the line:

    > cmak.zkhosts="<zookeeper_ip1>:2181,<zookeeper_ip2>:2181"


---
## Running ansible playbook
    > $ ansible-playbook playbooks/install-kafka-monitor.yaml 

---
## Testing setup
(Optional steps. These are performed in the post installation test.)

1. Make sure zookeeper and kafla clusters are up and running
2. open browser at http://monitor-ip-address:9000 
3. Create cluster


---
# Performing post installation checks

## Testing the kafka cluster and zookeeper ensemble

Run the playboook check_kafka-zk_services_status to verify that the services are up and running.
     
    > $ ansible-playbook playbooks/check_kafka-zk_services_status.yml 

## Testing the monitoring tools

Run playbook check_service_cmak_status

    > $ ansible-playbook playbooks/check_service_cmak_status.yml 

---
# References

- [Kafka documentation](https://kafka.apache.org/documentation/), official kafka documentation
- [Apache Zookeeper](https://zookeeper.apache.org/), Apache Zookeeper
- [Yahoo/CMAK](https://github.com/yahoo/CMAK), Yahoo CMAK project
- [Ansible Documentations](https://docs.ansible.com/), list of documentations for ansible