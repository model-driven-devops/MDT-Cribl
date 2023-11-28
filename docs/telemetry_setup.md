# Telemetry Setup

Alright, now we have a place to send our telemetry. Lets turn it on. For this excercise, I focused on 4 different catagories of telemetry data and set up ranges of subscription numbers to reserve
for the different catagories. If you take a look at the topoolgy, we have 3 sites and 5 different devices we want to use to send telemetry data.

<p align="center">
<img src="https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/6fccadc7-8d0d-4eba-9ba0-54940bf9decc" width="70%" height="70%">
</p>

I reserved 10 subscription numbers for each telemetry catagory and 100 for each of the 5 devices. I understand that probably doesn't make sense, but here is my table with the ranges corresponding to subscription numbers:

| Device | Total Range | Interface Range | BGP Range | SLA Range | OSPF Range |
| --- | --- | --- | --- | --- | --- |
| hq-pop | 0-99 | 0-9 | 10-19 | 20-29 | 30-39 |
| hq-rtr1 | 100-199 | 100-109 | 110-119 | 120-129 | 130-139 |
| hq-rtr2 | 200-299 | 200-209 | 210-219 | 220-229 | 230-239 |
| site1-rtr1 | 300-399 | 300-309 | 310-319 | 320-329 | 330-339 |
| site2-rtr1 | 400-499 | 400-409 | 410-419 | 420-429 | 430-439 |
| telegraf ports | 57000-57004 | 57000 | 57001 | 57002 | 57003 |


Do you have to set things up this way? of course not. Do you need to plan for subscription numbers? Nope. Does it help when you start to deal with massive amounts of data? It sure does! Long story short, once you start parsing and transforming your data, you will notice things that may not look right. Having things organized like this will help you track down any MDT issues and fix them quickly.

Just to note, this table will also help when I eventually Jinjafy the MDT configs.

# Telemetry Configurations

If you are not familiar with MDT, you need to know the x-path for the particular piece of data you want sent to your collector. If you want anything not documented in this repository, you will have to spend some time figuring out what that path is. To help with this, Cisco has a tool available called Yang suite. It's another docker compose file that you can run seperatly on the NSO node in the CML topology. We won't spend time on it here, because it's not very user friendly and I'm not even sure how I made it work.

The instructions to deploy it can be found here: https://github.com/CiscoDevNet/yangsuite/

## Interface Telemetry

This is the most common MDT configuration you see in user guides because it sends a good amount of useful data to help you monitor interface traffic. The main xpath if you are interested in statistics for all your interfaces on a device is:

```
/interfaces-ios-xe-oper:interfaces/interface/statistics
```
However, I would recommend defining the interface in the xpath and just creating a subscription per interface. Otherwise you'll get a ton of unusable data. Here is an example for the hq-pop router targeting the interface going out to the ISP:
```
/interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet8’]/statistics
```
You can see in the topology image above the incoming interface is GigabitEthernet2. What I ended up doing was creating two subscriptions, one for the ingress interface and one for the egress. Here are the configs using the table above:

### hq-pop Ingress Interface
```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet2’]/statistics
 source-address 10.0.254.2
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```

### hq-pop Egress Interface
```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet8’]/statistics
 source-address 10.0.254.2
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```
If you want to verify the configuration is valid, you can run the following commands:

```
show telemetry ietf subscription all
```
```
show telemetry ietf subscription 1 detail
```

We will cover the output in more detail throughout the next section, but below is an example of the expected output:

<details>

<summary>Interface Statistics Output</summary>

 [{"instrumentation_library":{"name":"unknown","version":"unknown"},"metrics":[{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_kbps","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3}]},"__type":"gauge"},
 {"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_num_flaps","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},
 {"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_octets","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1815370463}]},"__type":"gauge"},
 {"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3721050}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_discards","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_pps","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":2}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_unicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":3653323}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos_64","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_multicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards","description":"","unit":"","data":{"data_points":[{"attributes":{"path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_multicast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_pps","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_crc_errors","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_broadcast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301","host":"f054a84bef4c"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":485148603}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_broadcast_pkts","description":"","unit":"","data":{"data_points":[{"attributes":{"subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_kbps","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards_64","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site1-rtr1","subscription":"301","host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors_64","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets_64","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","name":"GigabitEthernet2","path":"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics","source":"site1-rtr1","subscription":"301"},"time_unix_nano":1700500278651000000,"exemplars":[],"value":485148603}]},"__type":"gauge"}],"schema_url":""}]
</details>

This should be enough to get you started. Now you can go to each device and fill in the values based on the table.

## BGP Telemetry

This has been a tricky one. Trying to get decent data from BGP telemetry is, well, interesting. Below are the configurations that have worked so far on hq-rtr1, which is running BGP in our simulation, along with the expected output:

### HQ-RTR1 BGP Neighbor Connection

xpath
```
/bgp-state-data/neighbors/neighbor/connection
```
MDT Config
```
telemetry ietf subscription 110
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/connection
 source-address 172.16.0.2
 source-vrf internal_1
 stream yang-push
 update-policy periodic 1500
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
````
You'll notice I added the internal vrf to my MDT config and changed the port to 57001. Again, you do not have to do this port thing but I find it makes troubleshooting a little easier.

<details>

<summary>BGP Neighor Connection Output</summary>

[{"instrumentation_library":{"name":"unknown","version":"unknown"},"metrics":[{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection_total_dropped","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"hq-rtr1","subscription":"130","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection"},"time_unix_nano":1701194817910000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection_total_established","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"hq-rtr1","subscription":"130","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection"},"time_unix_nano":1701194817910000000,"exemplars":[],"value":1}]},"__type":"gauge"}],"schema_url":""}]
</details>

This one is a little cleaner, but you'll notice you end up with:
- connection_total_established
- connection_total_dropped

Cool! Lets add another one. 

### site2-rtr1 BPG Neighbor Counters

xpath
```
/bgp-state-data/neighbors/neighbor/bgp-neighbor-counters
```

MDT Config
```
telemetry ietf subscription 411
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters
 source-address 192.168.2.1
 source-vrf internal_1
 stream yang-push
 update-policy periodic 1500
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

<details>

<summary>BGP Neighbor Counters</summary>
[{"instrumentation_library":{"name":"unknown","version":"unknown"},"metrics":[{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/opens","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/updates","description":"","unit":"","data":{"data_points":[{"attributes":{"path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":2}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/updates","description":"","unit":"","data":{"data_points":[{"attributes":{"afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":9}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_outq_depth","description":"","unit":"","data":{"data_points":[{"attributes":{"neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/route_refreshes","description":"","unit":"","data":{"data_points":[{"attributes":{"source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":2}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_inq_depth","description":"","unit":"","data":{"data_points":[{"attributes":{"afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/notifications","description":"","unit":"","data":{"data_points":[{"attributes":{"neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/keepalives","description":"","unit":"","data":{"data_points":[{"attributes":{"afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":55286}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/route_refreshes","description":"","unit":"","data":{"data_points":[{"attributes":{"host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/opens","description":"","unit":"","data":{"data_points":[{"attributes":{"vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":1}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/notifications","description":"","unit":"","data":{"data_points":[{"attributes":{"afi_safi":"vpnv4-unicast","host":"f054a84bef4c","neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":0}]},"__type":"gauge"},{"name":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/keepalives","description":"","unit":"","data":{"data_points":[{"attributes":{"neighbor_id":"10.255.255.1","path":"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters","source":"site2-rtr1","subscription":"411","vrf_name":"default","afi_safi":"vpnv4-unicast","host":"f054a84bef4c"},"time_unix_nano":1701195777035000000,"exemplars":[],"value":55295}]},"__type":"gauge"}],"schema_url":""}]
 
</details>

Woot. This is a lot more data. This sends you the following:
- bgp-neighbor-counters_sent/opens
- bgp-neighbor-counters_sent/notifications
- bgp-neighbor-counters_received/keepalives
- bgp-neighbor-counters_received/route_refreshes
- bgp-neighbor-counters_outq_depth
- bgp-neighbor-counters_sent/updates
- bgp-neighbor-counters_sent/route_refreshes
- bgp-neighbor-counters_received/opens
- bgp-neighbor-counters_received/updates
- neighbor/bgp-neighbor-counters_inq_depth

Now, if this is too much unneccessary data you can either modify your MDT config to only capture a more specefic xpath, such as:
```
filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent
```
or
```
filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/recieved
```

