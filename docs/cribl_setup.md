# Table of Contents

* [Cribl Setup]
  * [Initial Settings]
  * [OpenTelemetry Source]
  * [ElasticSearch Destination]
* [Pipeline Creation]
  * [Eval Function]
  * [Event Breaker Function]
  * [Additional Transformation]
* [Sending to ElasticSearch]

# Cribl Setup

This section walks through using cribl to transform your data and make it usable in ElasticSearch.

## Initial Settings

Once the container is deployed and listening on the default port of 4317, we need to do very little with the initial config. The only setting we really need changed is under 
"System", "General Settings", and "Default TLS Settings". We only need to turn off validate server certs to let Cribl trust data being sent by telegraf. Once you toggle this selection,
the container will restart.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/e6a99bb4-c715-4ba0-a906-da48531998ba" width="50%" height="50%">
</p>


## OpenTelemetry Source

Now that data is streaming from telegraf to cribl, we need to set up a source to accept the data. Open your Cribl UI, navigate to “collect” and then select “add source” on the top. Scroll all the way to the bottom and select “more sources”. You’ll finally see OpenTelemetry. Go ahead and add it.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/812986d1-05b1-434d-ba38-15a149b1c2a3" width="40%" height=40%">
</p>

Once you add it, move your mouse over it and you’ll get the option to configure the source. Under General Settings, you just need to set the IP to listen 
on 0.0.0.0 and set the port to whatever port your container has open to ingest data. The default is 4317. The protocol is gRPC and should already be set.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/e0b16ce9-6ef2-470f-8582-d5d5d837cfd0" width="40%" height="40%">
</p>

Next, you can select “Authentication” and set it to “none”. Since we are not using a production environment and have not set up any type of TLS settings,
we need to make sure we also set the server side settings to disable.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4d60cd75-a46a-44e4-969c-f46c4007b25b" width="40%" height="40%">
</p>

Thats it. Thats all it takes to set up the OpenTelemetry data source. If you want to verify events are coming in, you can select the status, charts, or live data
tabs. You should see data coming in. If you check the ”Live Data” tab, you’ll notice it has a default filter expression to display only the OpenTelemetry data.
It’s worth while to copy this expression since we can use it later to filter “__inputID=='open_telemetry:1”

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/67222c09-9144-4c36-98aa-c496372b09b7" width="40%" height="40%">
</p>

## ElasticSearch Destination

While we are setting up our source, we may as well set up our elasticsearch destination. I would highely recommend not sending data to it until you create your
pipeline though. Back in your collect menu, select "add destination". Find the option to add "ElasticSearch".

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/d62a5431-bab0-4292-a041-927bd52546bb" width="40%" height="40%">
</p>

The settings are pretty straight forward (just like OpenTelemetry). The URL to send data to your Elastic API is https://es01:9200/_bulk. We are able to use es01 since 
all our containers are running on the same host. Name the data stream anything you'd like. You can give it a clever name like "Telemetry" or "mdt". The Username and
Password will match whatever you defined in your docker config. 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4c5e1b01-3cbf-41fd-91aa-b14df06fb49b" width="40%" height="40%">
</p>

And of course, because we all love to sacrifice security for productivity, select the advanced settings and turn off "validate server certs". 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/59fef724-e396-4676-9c04-f86ba102b27b" width="40%" height="40%">
</p>

