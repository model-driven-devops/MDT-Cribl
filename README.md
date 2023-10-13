# MDT-Cribl
This repository contains the docker-compose file to launch the following containers:
- Telegraf: Telegraf listens and ingests Cisco MDT on port 57000, decodes, and sends as OpenTelemetry to Cribl
- Cribl: Cribl listens for OpenTelemetry on port 4317 and ships to Elastic.
- Elastic and Kibana: They do what they normally do. Kibana is set to be accessed on 5601.

Values can be changed using the .env file.

## Setting Up Cribl

This launches a single instance of cirbl-edge and does not pass any config files (Yet). Once you login to Cribl, you can add the OpenTelemetry source and Elasticsearch destination. The only settings that need to change are:

Adding the login information for elasticsearch:

![Screenshot 2023-09-26 at 4 03 47 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/7af60e7e-dead-49e4-abd4-e5f90e06dbec)

Turning off "Validate Certs". This can be added later.

![Screenshot 2023-09-26 at 4 04 57 PM](https://github.com/model-driven-devops/MDT-Cribl/assets/65776483/064f37d2-a90b-4648-b56f-ebcdbe0cec7b)

## Adding MDT subscriptions

To add additional MDT subscriptions, All you need to do is modify your docker file and your telegraf config.

#### telegraf config

In your telegraf config, you need to copy and paste your input for each subscription. Each subscription requires a different port defined.

```
## Subscription 1
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57000"

## Subscription 2
[[inputs.cisco_telemetry_mdt]]
  transport = "grpc"
  service_address = ":57001"

[[outputs.opentelemetry]]
  service_address = "xx.xx.xx.xx:4317"
```

#### docker config

Once you add your subscriptions to telegraf, you need to expose the new ports in your docker-config file:

```
telegraf:
    container_name: telegraf
    image: telegraf:latest
    ports:
      - "57000:57000"
      - "57001:57001"
    volumes:
      - ./conf/telegraf/telegraf.conf:/etc/telegraf/telegraf.conf:ro
      - /var/run/docker.sock:/var/run/docker.sock
```
## Sample MDT Configs

### Interface Statistics

MDT Config
```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface/statistics
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 500
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```
Specifying Interface

```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet8’]/statistics
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 500
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```

Output

```
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_kbps",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_octets",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unicast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards_64",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets_64",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_octets",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_pps",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_broadcast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_multicast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_errors",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_rx_kbps",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_num_flaps",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_multicast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_discards",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_errors_64",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_tx_pps",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_crc_errors",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_unknown_protos_64",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_in_broadcast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_unicast_pkts",
"Cisco-IOS-XE-interfaces-oper:interfaces/interface/statistics_out_discards"
```
### BGP

#### BGP Neighbor Counters
```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 500
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```

Output

```
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/updates",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/updates",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/keepalives",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/route_refreshes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_outq_depth",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/opens",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/notifications",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/keepalives",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_sent/route_refreshes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/opens",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_received/notifications",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters_inq_depth"
```

#### BGP neighbor counters sent
```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

Output

```
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent_updates",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent_notifications",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent_keepalives",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent_route_refreshes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/sent_opens"
```

#### BGP neighbor counters recieved

```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

Output

```
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received_keepalives",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received_route_refreshes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received_opens",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received_updates",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/bgp-neighbor-counters/received_notifications"
```

#### BGP Connection

```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/connection
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

Output

```
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection_total_established",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/connection_total_dropped"
```

#### BGP Prefix Activity

```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /bgp-state-data/neighbors/neighbor/prefix-activity/sent
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

Output
```
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_multipaths",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_current_prefixes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_total_prefixes",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_implicit_withdraw",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_explicit_withdraw",
"Cisco-IOS-XE-bgp-oper:bgp-state-data/neighbors/neighbor/prefix-activity/sent_bestpaths"
```

## EIGRP

#### EIGRP Neighbor
```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /eigrp-oper-data/eigrp-instance/eigrp-interface/eigrp-nbr
 source-address xx.xx.xx.xx
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57001 protocol grpc-tcp
```

## Untested x-path
```
/bgp-state-data/neighbors/neighbor/up-time
/bgp-state-data/address-families/address-family/activities
/bgp-state-data/bgp-route-vrfs/bgp-route-vrf/bgp-route-afs/bgp-route-af/bgp-route-neighbors
/bgp-state-data/bgp-route-vrfs/bgp-route-vrf/bgp-route-afs/bgp-route-af/bgp-peer-groups
```

## On-Change Untested

```
 filter xpath /ios-events-ios-xe-oper:ospf-neighbor-state-change
 filter xpath /ios:native
 filter xpath /ios-events-ios-xe-oper:bgp-peer-state-change
 filter xpath /ios-events-ios-xe-oper:ospf-neighbor-state-change
 filter xpath /ios-events-ios-xe-oper:ospf-interface-state-change
 filter xpath /ios-events-ios-xe-oper:interface-state-change
 filter xpath /ios-events-ios-xe-oper:memory-usage
 filter xpath /ios-events-ios-xe-oper:cpu-usage
 filter xpath /ios-events-ios-xe-oper:interface-admin-state-change
 filter xpath /ios-events-ios-xe-oper:sdcard-fault
 filter xpath /ios-events-ios-xe-oper:system-reboot-complete
 filter xpath /ios-events-ios-xe-oper:system-reboot-issued
 filter xpath /ios-events-ios-xe-oper:flash-fault
 filter xpath /ios-events-ios-xe-oper:system-login-change
 filter xpath /ios-events-ios-xe-oper:system-logout-change
 filter xpath /ios-events-ios-xe-oper:tempsensor-fault
 filter xpath /ios-events-ios-xe-oper:disk-usage
 filter xpath /ios-events-ios-xe-oper:usb-state-change
 filter xpath /ios-events-ios-xe-oper:sfp-state-change
 filter xpath /ios-events-ios-xe-oper:sfp-support-state
 filter xpath /ios-events-ios-xe-oper:fantray-fault
 filter xpath /ios-events-ios-xe-oper:fan-fault
 filter xpath /ios-events-ios-xe-oper:tempsensor-state
```
