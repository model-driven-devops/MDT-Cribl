# Topology and Tools Setup

In this section, we will go over setting up your environment. I am doing everything self contained in Cisco Modeling Labs, but the tools can be deployed using any
host that supports docker. I've included the topology files that can be imported directly into CML if it is available to you. If you would like more information
on the topology itself, or using ansible too configure the environment, you can head over to the main [MDD repo](https://github.com/model-driven-devops/mdd/tree/main).

## Import Topology

There are two topology files located in the files directory of this repository - mdd_prod and mdd_test. If you want a fully configured network, use mdd_prod. If you want to set up the network yourself, you can use mdd_test. Go ahead and boot up the telemetry node, hq-mgmt-switch, and hq-mgmt-bridge. Console into the Telemetry node and login. If you need the username and password, you can select the "Edit Config" tab an you should see it in the configuration file. Once logged in, go ahead and grab the IP address. From here you can continue to operate from the console or SSH in.

<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/17dbf7e8-95fb-484a-b723-eee63e488c21">

## Set Up Telemetry Node

The first thing we need to do is install docker compose onto the telemetry node. You can find the instructions for ubuntu here - https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository. Once you have that installed, you can copy this repository to the node.
