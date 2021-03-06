# JTS Transaction Service

This image contains JTS transaction service which works in conjunction with JacORB name server.


== Building the Dockerfile

`NARAYANA_VERSION`::
  is used only during docker build and defines what version of Narayana
  will be downloaded

== Usage

```bash
# export NARAYANA_VERSION=...
docker build -t jts-transaction-service --build-arg NARAYANA_VERSION=$NARAYANA_VERSION .
```

## Run container

    docker run -p 4711:4711 -it --link jacorb-name-server:jacorb-name-server --name jts-transaction-service jboss/jts-transaction-service

### Prerequisite

Transaction service requires jboss/jacorb-name-server to be started.

## Optional configuration

### Setting proxy IP address

Default configuration will work as long as everything runs on the same linux machine. However, in order to make JTS service work on the remote machine or on Boot2Docker, we need to set PROXY_IP environment variable. This IP has to be accessible by the client application. Otherwise, IOR generated by ORB will contain local container's IP. You can do this as follows:

    docker run -p 4711:4711 -it -e "PROXY_IP={your proxy IP}" --link jacorb-name-server:jacorb-name-server --name jts-transaction-service jboss/jts-transaction-service

### Using an external object store

In order to keep the contents of the object store safe, it is recommended to store it outside of the docker container. There are two ways of doing that: providing a jbossts-properties.xml with any standard Narayana options (such as connecting to a JBDC object store) as explained in the next section, or mounting a directory to the containers host for object store. With the way we have chosen to configure the docker container, the object store is by default located in container's /opt/jboss/narayana/tx-object-store directory, therefore you can use
-v attribute to mount host's directory as follows:

    docker run -p 4711:4711 -it -v /<REQUIRED_FOLDER_ON_HOST>/:/opt/jboss/narayana/tx-object-store --link jacorb-name-server:jacorb-name-server --name jts-transaction-service jboss/jts-transaction-service

### Providing Narayana configuration

Narayana configuration is located in /opt/jboss/narayana/etc/jbossts-properties.xml and it's logging configuration in
/opt/jboss/narayana/etc/log4j.properties. These can be replaced by mounting host's directory which contains
jbossts-properties.xml and log4j.properties files.

NOTE: In the docker configuration we have provided, the OrbPortabilityEnvironmentBean.bindMechanism and ObjectStoreEnvironmentBean.objectStoreDir properties will always be overwritten with "NAME_SERVICE" and "/opt/jboss/narayana/tx-object-store" respectively. You would need to modify the dockerfile to change that.

### Providing Narayana configuration via environment variable

Individual configuration options can be passed via the NARAYANA_OPTS environment variable. Use docker's -e option for that.

-e "NARAYANA_OPTS=\"-DObjectStoreEnvironmentBean.objectSreDir=/opt/jboss/narayana/object-store\""

### Providing external libraries

For the particular use cases, such as JDBC object store usage, external libraries have to be provided for Narayana. This can be achieved by mounting directory containing required jars to /opt/jboss/lib directory. All jars from this directory will be added to the class path. NOTE: directory is not scanned recursively.
