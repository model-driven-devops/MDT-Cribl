# Visualizing Model Driven Telemetry

This repository is meant to be a guide for transforming and visualizing your network telemetry, allowing you to format, enhance, augment, and basically clean up the data before attempting to make helpful visuals. In this repo we will address the following:

1. Simplifying your data by using Model-Driven telemetry. If you aren't familiar with MDT, you can read about it on [DevNet](https://developer.cisco.com/docs/ios-xe/#!streaming-telemetry-quick-start-guide). The basic jist of MDT is that it allows you to get specefic with the data you send to collectors versus sending bulk data (like syslog or netflow) and making doing something with the data the security teams problem. There are other benefits, like being light weight, low overhead, allowing subscriptions, only sending data on change, etc.
2. Transforming your data as it goes from the device to your visualizing tool. You'll notice any time you send "telemetry" from a thing, you have a lot of garbage that comes with it. Instead of having to comb through it in splunk or elastic or some other visualization tool, lets clean it up first!
3. Enhnacing your data by adding some stuff to it as it makes it way into your visualiztion tool. This includes creating a geo-location to view output on a map.

## Contents
* [Topology and Tools Setup](docs/setup.md) - This section covers using the included docker compose file to spin up the necessary containers. The containers deployed include Telegraf, Cribl, Elasticsearch and Kibana.
* [Configuring Telemetry](docs/telemetry_setup.md) - This section covers configuring the network to start streaming telemetry to our monitoring containers.
* [Transforming Data](docs/cribl_setup.md) - This section covers how to use Cribl to transform your data into something useable for visualizing.
* [Visualizing Data](docs/dashboard_setup.md) - This section gives examples of visualizing your telemetry in Kibana.
