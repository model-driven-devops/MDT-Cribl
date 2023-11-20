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

![Screenshot 2023-11-20 at 2 05 20 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/e6a99bb4-c715-4ba0-a906-da48531998ba)

## OpenTelemetry Source

Now that data is streaming from telegraf to cribl, we need to set up a source to accept the data. Open your Cribl UI, navigate to “collect” and then select “add source” on the top.
Scroll all the way to the bottom and select “more sources”. You’ll finally see OpenTelemetry. Go ahead and add it.
