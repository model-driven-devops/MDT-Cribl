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
