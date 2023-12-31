# Topology and Tools Setup

In this section, we will go over setting up your environment. I am doing everything self contained in Cisco Modeling Labs, but the tools can be deployed using any
host that supports docker. I've included the topology files that can be imported directly into CML if it is available to you. If you would like more information
on the topology itself, or using ansible too configure the environment, you can head over to the main [MDD repo](https://github.com/model-driven-devops/mdd/tree/main).

## Import Topology

There are two topology files located in the files directory of this repository - mdd_prod and mdd_test. If you want a fully configured network, use mdd_prod. If you want to set up the network yourself, you can use mdd_test. Go ahead and boot up the telemetry node, hq-mgmt-switch, and hq-mgmt-bridge. Console into the Telemetry node and login. If you need the username and password, you can select the "Edit Config" tab an you should see it in the configuration file. Once logged in, go ahead and grab the IP address. From here you can continue to operate from the console or SSH in.

<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/17dbf7e8-95fb-484a-b723-eee63e488c21">

## Set Up Telemetry Node

The first thing we need to do is install docker compose onto the telemetry node. You can find the instructions for ubuntu here - https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository. Once you have that installed, you can copy this repository to the node.

```
git clone https://github.com/model-driven-devops/MDT-Cribl.git
```

Navigate to the telemetry directory - MDT-Cribl/telemetry/

You can take a look at the docker-compose.yml file if you want to make any changes, but the compose file deployes four containers:
- Elasticsearch: This will be the timeseries database that stores your telemetry.
- Kibana: This will be the visualization tool.
- Cribl: This will be used to transform the data as it comes through telegraf.
- Telegraf: This will be used to decode the MDT and convert it to OpenTelemetry.

## Setup Telegraf

Change into the telegraf directory located in the telemetry folder. Open the telegraf.conf file using your preferred text editor. You'll see the configuration is pretty basic (which is awesome!). The reason we need to insert telegraf into our workflow is to take advantage of its ability to decode the protocol used for MDT (model driven telemetry). MDT using gRPC to send data from the Cisco device to your preferred collector. After some frustrating work, it turns out gRPC and its encoder (protobuf) is not natively supported in some monitoring tools, so sending data directly to ElasticSearch for example, is not going to work. We need something like telegraf that can decode and understand MDT to help.

### Telegraf Config
One you open the telegraf.conf file, you'll notice it's very basic. We are just going to use the MDT telegraf plugin, define the port to listen on, and send the data on its way.

```
# Cisco MDT Telemetry
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"

[[outputs.opentelemetry]]
  service_address = "0.0.0.0:9420"
```

The only thing you may want to edit in the telegraf config is adding additional ports to listen on. This doesn't do much besides help seperate your data streams. In the configuring telemetry section, you'll see what I mean. If you want to add additional ports, the below example depicts how simple it is:

```
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"

[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57001"

[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57002"

[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57003"

[[outputs.opentelemetry]]
  service_address = "0.0.0.0:9420"
```

Once the MDT comes in, we are going to send it on it's way formatted as OpenTelemetry. If you are using seperate hosts, go ahead and add the IP for your cribl server. If not, you can use 0.0.0.0 since all containers will be running on the same host. Save your changes and lets move onto elasticsearch.

## Setup Elasticsearch

Head back to the telemetry directory. If you want to change any settings for elasticsearch, it is done using a environment variable file. You may not see this in the directory, but if you use your text editor by typing sudo nano .env or sudo vi .env you will be able to change various settings.

```
# Password for the 'elastic' user (at least 6 characters)
ELASTIC_PASSWORD=changeme

# Password for the 'kibana_system' user (at least 6 characters)
KIBANA_PASSWORD=changeme

# Version of Elastic products
STACK_VERSION=8.7.1

# Set the cluster name
CLUSTER_NAME=docker-cluster

# Set to 'basic' or 'trial' to automatically start the 30-day trial
LICENSE=basic
#LICENSE=trial

# Port to expose Elasticsearch HTTP API to the host
ES_PORT=9200

# Port to expose Kibana to the host
KIBANA_PORT=5601

# Increase or decrease based on the available host memory (in bytes)
ES_MEM_LIMIT=1073741824
KB_MEM_LIMIT=1073741824
LS_MEM_LIMIT=1073741824


# SAMPLE Predefined Key only to be used in POC environments
ENCRYPTION_KEY=c34d38b3a14956121ff2170e5030b471551370178f43e5626eec58b04a30fae2
```
Its important to note that we are using a static encryption key, which should only be used for demos and pocs. Before we finish up with elasticsearch, we want to change the java heap setting for the host itself. Go ahead and execute this command:

```
sysctl -w vm.max_map_count=262144
```

## Docker Compose

Everything should be ready to go in the docker compose file. If you did happen to add additional ports to telegraf, you will want to make sure you open those ports in the compose file before executing it.

```
telegraf:
    container_name: telegraf
    image: telegraf:latest
    ports:
      - "57000:57000"
      - "57001:57001"
      - "57002:57002"
      - "57003:57003"
    volumes:
      - ./conf/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf:ro
      - /var/run/docker.sock:/var/run/docker.sock
```
Alright, lets go ahead and spin up our containers. Personally, I like to do a sudo docker compose up from the CML console, and then open a seperate SSH session if I need to do anything else to the containers. This way I can watch the container logs from the CML console. If you want to skip all that, you can just execute the compose in detached mode.

```
sudo docker compose up
```

Once the containers finish coming up, try to access them.
- Kibana is port 5601
- Cribl is port 9420

If you can access both, lets start to configure some telemetry!
