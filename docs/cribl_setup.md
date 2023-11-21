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
  * [Adding GeoPoint](#adding-geopoint)
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

## Adding GeoPoint

If you are like me, you appreciate good visuals. I love when I can see data on a map. To do so, you need to feed elasticsearch a GeoPoint for each device. This is not super easy to do and there isn't really anything built into cribl ourside of the GeoIP function, which for private networks isn't really useful. To add a GeoPoint as part of your pipeline, we need to use two deperate functions. First, we need to create a lookup table. In Cribl, navigate to "More" and select "Knowledge". 

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/3308c2fc-dfe7-47f5-817f-56ba5cd8088c" width="%40" height="40%">
</p>

## Adding Parsers

