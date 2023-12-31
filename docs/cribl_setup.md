# Table of Contents

* [Cribl Setup](cribl_setup)
  * [Initial Settings](#initial-settings)
  * [OpenTelemetry Source](#opentelemetry-source)
  * [ElasticSearch Destination](#elasticsearch-destination)
* [Pipeline Creation](#pipeline-creation)
  * [Challenges](#challenges)
  * [Preparing Data](#preparing-data)
  * [Eval Function](#eval-function)
  * [Event Breaker Function](#event-breaker-function)
  * [Baseline Eval Function](#baseline-eval-function)
  * [Adding GeoPoint](#adding-geopoint)
  * [Adding Parsers](#adding-parsers)
  * [Clean Up](#clean-up)
* [Sending to ElasticSearch](#sending-to-elasticsearch)

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

And of course, because we all love to sacrifice security for productivity, select the advanced settings and turn off "validate server certs". Once you're done you can go ahead and 
select add or save.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/59fef724-e396-4676-9c04-f86ba102b27b" width="40%" height="40%">
</p>

We are not sending any data yet, but you should be able to select "Logs" and check if cribl was able to establish a connection to ElasticSearch. If not, we can kick the can down the 
road and worry about troubleshooting that later.

## Pipeline Creation

### Challenges

Now this is the fun and frustrating part of telemetry. We need to make our data useful. If you don’t take time to do this, you’ll spend cycles trying to create 
visualizations that will never tell the story you really want to tell. This gets tricky with MDT and OpenTelemetry, because the data is sent as one 
massive JSON schema with multiple embedded arrays, the useable metrics being buried deep inside the schema. Here is the interface statistics output for a single 
interface on a single device. You can expand the section below.

<details>

<summary>Interface Statistics Output</summary>
 [{"instrumentation_library":{"name":"unknown","version":"unknown"},"metrics":[{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_kbps","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_num_flaps","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_octets","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1815370463}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3721050}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_discards","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_pps","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":2}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_unicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3653323}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos_64","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_multicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards","description":"","unit":"","data":{"data_points":[{"attributes":{"path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_multicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_pps","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_crc_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_broadcast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":485148603}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_broadcast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_kbps","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards_64","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors_64","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets_64","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":485148603}]},"__type":"gauge"}],"schema_url":""}]
</details>

To make it easier to see how deeply the data is embedded, here is a partial JSON representation of just two statistic data points:

```
{
  "resource": {
    "dropped_attributes_count": 0,
    "attributes": {}
  },
  "instrumentation_library_metrics": [
    {
      "instrumentation_library": {
        "name": "unknown",
        "version": "unknown"
      },
      "metrics": [
        {
          "name": "Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_kbps",
          "description": "",
          "unit": "",
          "data": {
            "data_points": [
              {
                "attributes": {
                  "source": "site1-rtr1",
                  "subscription": "301",
                  "host": "f054a84bef4c",
                  "name": "GigabitEthernet2",
                  "path": "Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics"
                },
                "time_unix_nano": 1700500278651000000,
                "exemplars": [],
                "value": 3
              }
            ]
          },
          "__type": "gauge"
        },
        {
          "name": "Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_num_flaps",
          "description": "",
          "unit": "",
          "data": {
            "data_points": [
              {
                "attributes": {
                  "host": "f054a84bef4c",
                  "name": "GigabitEthernet2",
                  "path": "Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics",
                  "source": "site1-rtr1",
                  "subscription": "301"
                },
                "time_unix_nano": 1700500278651000000,
                "exemplars": [],
                "value": 0
              }
            ]
          },
          "__type": "gauge"
        },
```
You’ll notice a few things (or maybe you won’t so I’ll point it out).
- There is a lot of useless data and data fields. The resource, __type, and instrumentation library fields are not even giving us any actual data. 
- Everything is buried under the “instrumentation_library_metrics" field, which is a very annoying name to use when you’re creating visualizations because whether
its interface statistics or SLA telemetry, it all gets put under this field.
- The simple data we need is distributed in multiple arrays. If we want to see the <b>statistics_tx_kbps</b> for the <b>site1-rtr1</b> router coming in from <b>GigabitEthernet2</b>,
  eachpiece of data is different layers of the schema.
  - The name of the statistic is under instrumentation_library_metrics[0].metrics[0].name and displayed as the full xpath.
  - The name of the device is under instrumentation_library_metrics[0].metrics[0].data.data_points[0].attributes.source
  - The value associated with statistics_tx_kbps is under instrumentation_library_metrics[0].metrics[0].data.data_points[0].value
 
To make this more challenging, because the arrays are embedded, you need to use the number to identify where in the array your data is. For example the value of statistics_num_flaps
would be instrumentation_library_metrics[0].metrics[1].data.data_points[1].value. The next data point would be instrumentation_library_metrics[0].metrics[2].data.data_points[2].value
and so on. Not only do visualizations become extremely hard to create, but using any standard function to break apart the data without having to write javascript expressions that 
iterate through the array is frustrating. 

Enough with the problems, lets talk about the solution!

## Preparing data

Cribl has extremely powerful built in functions, and I tried just about everything before figuring this out. The JSON event breaker is where we want to start. This basically lets you 
define a field in your schema and when data runs through your pipeline, cribl will break each of those fields into their own events. For example, the massive interface statistic 
schema above is treated as one event. You’ll notice that the path to the data we need is under the <i>instrumentation_library_metrics[0].metrics[0]</i> field. If we set our event 
breaker to the metrics field, we should get separate events we can filter on. 

You may be thinking “No duh. Why wasn’t this the first thing you tried?”. Well it was and it didn’t work. Why didn’t it work? because dealing with JSON schema and streaming data 
makes my eyes bleed so I failed to notice this “Limitations” notice in the documentation:

https://docs.cribl.io/stream/event-breaker-function/

> The Event Breaker Function operates only on data in _raw. For other events, move the array to _raw and stringify it before applying this Function.

It just so happens our data is not in _raw and not stringified, so the first thing we need to do is put it in _raw and stringify it. In your collect window, mouse over the middle of 
the line connecting your source and destination. You will see the “pipeline” box. Go ahead and select it.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/110b4c22-5a14-4a80-85e4-1c804ac37834" width="40%" height="40%">
</p>

You’ll be presented with the option of a bunch of pre-built pipelines created by engineers who probably ran into all the same issues we ran into. You can use your main pipeline, or 
you can create a new one by selecting “Create Pipeline” in the top right corner.

<p align="center">
 <img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/8509baec-cbf6-438c-95c6-6d7265deeb4c" width="50%" height="50%" alt="Pipeline Creation Window">
</p>

Now we want to capture some sample data to test against as we create our pipeline. Select “Capture Data”.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/b1448a00-bced-4e3f-8c03-eb1c390b06e7" width="50%" height="50%">
</p>

Once you select capture data, you can use that expression you copied earlier to only capture the OpenTelemetry data.

```
__inputId=='open_telemetry:1'
```

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/65abf433-6c09-4a38-bc75-ec904cf427cd" width="50%" height="50%">
</p>

You should see the data start to come in. Based on the number you set in the event capture, you should see that amount of events. Save the sample file.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c3de4fe4-d2fa-4abd-a4ad-df5a50535bae" width="50%" height="50%">
</p>

## Eval Function

Under the “Add Function”, we are going to select “Eval”.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/1ae1796d-caba-4773-8369-39c3c6653693" width="50%" height="50%">
</p>

To format all of our data to use the event breaker function, we are going to enter the following values for name and expression:
- Name: _raw
- Value Expression: JSON.stringify(instrumentation_library_metrics)
  
We are basically taking the entire schema that comes into cribl, placing it into a field called "raw" and turning it into one large string. If you’d like, you can go ahead and remove 
some of those empty fields as well. I added instrumentation_library_metrics, resource, unit, schema_url, and description just to clean 
things up a bit

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4681c530-79f1-4662-afd5-6f6d578c859e">
</p>

Once you save your initial function, take a look at the preview of your data. On the top you’ll see a toggle for IN/OUT. If you select OUT, you will see what the output of your 
function is. Now you will notice you have a raw field with all your data and the instrumentation_library_metrics field has been removed. The data is also grouped together as opposed 
to before the pipeline, where you could drill down into the schema. We basically turned it into a giant string.

| Before Function | After Function |
| --- | ---|
| <img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/71b677f1-bf7c-44f5-a5c8-450587bce90b"> | <img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/50c6c628-3255-4132-834a-59353ddfb6f4"> |

## Event Breaker Function

Now that we have the data in the corrected format, we can use the event breaker to turn the single event, large schema into a series of small events. We can select the same “Add 
Function” field. In any of these functions, you can use a filter to get more granular, but since we are just working with all data coming in, we can keep it set to true. “Under 
Existing or New”, select “New” and then under “Event Breaker Type” select JSON Array. We want to target the “Metrics” field as our breaker. You can leave everything else as default.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/04daa8fe-5407-4982-acfb-ed98cd6dd833" width="30%" height="30%">
</p>

Now take a look at your data. That looks better doesn’t it? In this example, we now have the Statistics_tx_kbps as its own event. From this point forward, the world is your oyster 
and you can cut up and manipulate your data however you want. Feel free to skip ahead or keep modifying your data.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/03217b7e-131e-488b-831d-9824e8dda312">
</p>

Go ahead click save and take a look at your data. It's beautiful.


## Baseline Eval Function

Now we have our data broken apart into individual events (woohoo!), lets start extracting the fields from our _raw schema that are consistant across all our different telemetry 
streams. This will make adding things like geopoints and parsers much easier to work with. Start by adding a new Eval function.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/2e80ff99-edc8-4899-ac78-66e265d9598a" width="50%" height="50%">
</p>

You can see in the screenshot, I used the full data path as my value expression to identifify what specefic field I was targeting. For example, the device name is nested under 
data.data_points[0].attributes.source. If you need to figure out your full data path, you can use the "simple preview" and expand each field until you find the fields you want to 
extract.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/1291467d-6ed0-49ab-be6d-145275f18f22" width="60%" height="60%">
</p>

For our baseline eval function, we only want to elevate the fields that are consistant across all of our telemetry events because this function will be applied to all telemetry data coming through this pipeline. We can use the filter later to make adjustments to our speceific data streams. Below is an example of the SLA event. 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/17e34db3-9367-4ae2-9119-87873bc8cea6" width="60%" height="60%">
</p>

You can see the fields consistent across both the interface event and SLA event are:

- data.data_points[0].value
- data.data_points[0].attributes.source
- data.data_points[0].attributes.name
- data.data_points[0].attributes.path

Here is a closer screenshot of the mapping. You can change the names to whatever you want here. For example, I changed the "name" field to "interface".

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/0a7d9260-ab77-4ae1-98f7-14f5830c3679" width="60%" height="60%">
</p>

Once you save your function, take a look at how your data has changed:

| SLA Telemetry | Interface Telemetry |
| --- | ---|
| <img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/94087c84-c0a0-4f41-bcd2-e921ab76f447"> | <img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/2850ee35-69a3-4cb5-9e9d-d8b6ac8375e4"> |

Before we proceed, lets add one more field. Why? Well, when the data goes into elasticsearch, we want to easily identify what the value means. In our current data streams, we see a "name" field with a long path - Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors_64. We don't need all that extra stuff. Select "add field" and as your "Value Expression" add the following:

```
name.substring(name.lastIndexOf('/') + 1)
```

This basically takes whatever is after the last "/" in your name field and gets rid of the other stuff. Next, we are going to take the new name and place it under the attributes field by using the following for "Name":

```
data.data_points[0].attributes.name
```

Why are we putting it there? Well later on, we will be using parsers to extract our remaining data, placing it under a field that identifies the type 
of telemetry it is. For example, all bgp speceifc data will get placed under bgp.datapoint and SLA data will get placed under sla.datapoint. To easily visualize 
our data, we want the name to easily describe the value. By adding this field, we will eventually end up with interface.statistics_tx_kbps or bgp.
connection_total_dropped.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/1bea7a9b-be96-42c0-aae0-89651fe1503f" width="60%" height="60%">
</p>

Now isn't that beautiful. Under our attributes field, 


## Adding GeoPoint

If you are like me, you appreciate good visuals. I love when I can see data on a map. To do so, you need to feed elasticsearch a GeoPoint for each device. This is not super easy to 
do and there isn't really anything built into cribl ourside of the GeoIP function, which for private networks isn't really useful. To add a GeoPoint as part of your pipeline, we need 
to use two deperate functions. First, we need to create a lookup table. In Cribl, navigate to "More" and select "Knowledge". 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/3308c2fc-dfe7-47f5-817f-56ba5cd8088c" width="60%" height="60%">
</p>

Select "Add Lookup File". You'll have the choice of uploading a csv or creating a new text file. In this scenario, select the new text file. I find it easier to set up the headers, 
saving it, and then editing the file instead of editing it as a text file. At a minimum, we need a field to contain our lookup data and a field for latitude and longitude. I went 
ahead and added a city and state field as well.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/8fda93a8-aa03-4027-94d2-c946bd289f24" width="60%" height="60%">
</p>

Now that we have our initial lookup columns set up, you can save it and open it again. When you open it, you'll have the option to change the "Edit Mode". Change 
it to "table" and lets start adding our data. For our small topology, I am using the router name as the lookup field. If you are doing this for a larger topology, you can use any data field that is being ingested.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c2c56b90-9cf1-4c59-9169-06812e8c9f82" width="60%" height="60%">
</p>

Now we have our lookup table done. All we need to do is add the lookup function and point it to our table. Since we've taken the time to elevate some of the data fields, all we have to do is match the "source" field in our data with the "source" field in the lookup table.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/f546849e-40af-4058-bda9-d17564339d8d" width="60%" height="60%">
</p>

Now look at our data! Each event has the city, state, latitude, and longitude added. 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/b800e667-b38b-4647-b1c5-496351bc18f7" width="60%" height="60%">
</p>

### GeoPoint Eval Function

Now you would think sending fields titled "latitude" and "longitude" to elasticsearch would let you create amazing map visualizations, but no. There is more work to be done. There are many ways to do this, but since we are becoming cribl pipeline experts lets keep moving. For elasticsearch to recognize geopoint data, it needs to be sent in the correct format. Use another Eval function to merge the latitude and longitude into a single field called location. Make sure you use the + sign before both, otherwise they will be sent as a string.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/e8e031aa-573e-4ba9-9512-04fc05493ad5" width="60%" height="60%">
</p>

Placeholder. Need to figure out if Elasticsearch needs to be prepped.

## Adding Parsers

Alright, we're almost done. Now it's time to add our parsers to drop the rest of the data points into telemetry specefic catagories. Add a new parser function. In the filter field, we are going to target any event that has a similar path. This example with use BGP.

```
path.includes("Cisco-IOS-XE-bgp-oper:bgp-state-data")
```

Then we want to extract all data nested under the attributes field and append "bgp" to it so we can identify it. Our source will be:

```
data.data_points[0].attributes
```
and our destination will simply be "bgp".

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/355c1123-62b3-435b-92ca-dce8d0b7a052" width="60%" height="60%">
</p>

We can also remove the source and path field since we already have those elevated. Now take a look at our data!

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/a42fa6d1-4d89-4c77-925e-9ad1cec94cb4" width="60%" height="60%">
</p>

Go ahead and add a similar parser for each of your telemetry paths. Your pipeline should end up looking something like this:

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/7922ae4c-7da6-476a-a68d-7d56324d4c65" width="60%" height="60%">
</p>

## Clean Up

One final eval function to clean everything up should do it. All we want to do is remove the fields we aren't using anymore. 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/4d78f5a3-c9fe-490d-a4bf-e8e18289dba3" width="60%" height="60%">
</p>

Now doesn't that look better!

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/a3758e61-27a4-4a38-86d5-4bf40f4fd736" width="60%" height="60%">
</p>

## Sending to Elasticsearch

Now we should be ready to start sending data to elasticsearch for some visualization. Before we do that, remember those functions we created for the location? We need to prepare our elasticsearch index to accept those coordinates in the correct format. Go ahead and login to Kibana and head to the "Dev Tools" section. In the console, we are going to set the "location" field as a geopoint. You can use the code below, but make sure you replace "telemetry" with whatever the name of your index is.

```
PUT telemetry
{
  "mappings": {
    "properties": {
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```
Press the green arrow to send the command to elasticsearch and you should see the accepted output.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/84752afc-febd-479f-8ee5-73d2136431d1" width="80%" height="80%">
</p>

### Connect your pipeline

Head back to cribl and go to the collect window. Highlight the connection between the OpenTelemetry source and whatever destination you have it connected to. Select "Pipeline"

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/5d199f0f-8a6c-4ee2-85c4-e00f284c44a4" width="70%" height="70%">
</p>

Find the name of your pipeline and select it. Click save. You are now sending all data coming from the OpenTelemetry source (telegraf) through your pipeline, where it transforms the data. Once saved, grab the line connecting your source and destination and drag it over to ElasticSearch.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/c70f8a6f-1248-4e0d-9aeb-bf2f94374c9b" width="70%" height="70%">
</p>

Now you can select your elasticsearch destination and verify the data is being sent correctly. There are mutliple tabs that help. You can capture events like we did when creating the pipeline, look at logs, etc. The screen shot below shows the Live Data tab.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/9e416462-d31e-4e11-b27b-28d669ac5d1e" width="70%" height="70%">
</p>
