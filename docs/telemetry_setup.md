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

## Telemetry Configurations

If you are not familiar with MDT, you need to know the x-path for the particular piece of data you want sent to your collector. If you want anything not documented in this repository, you will have to spend some time figuring out what that path is. To help with this, Cisco has a tool available called Yang suite. It's another docker compose file that you can run seperatly on the NSO node in the CML topology. We won't spend time on it here, because it's not very user friendly and I'm not even sure how I made it work.

The instructions to deploy it can be found here: https://github.com/CiscoDevNet/yangsuite/

### Interface Telemetry

This is the most common MDT configuration you see in user guides because it sends a good amount of useful data to help you monitor interface traffic. The main xpath if you are interested in statistics for all your interfaces on a device is:

```
/interfaces-ios-xe-oper:interfaces/interface/statistics
```
However, I would recommend defining the interface in the xpath and just creating a subscription per interface. Otherwise you'll get a ton of unusable data. Here is an example for the hq-pop router targeting the interface going out to the ISP:
```
/interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet8’]/statistics
```
You can see in the topology image above the incoming interface is GigabitEthernet2. What I ended up doing was creating two subscriptions, one for the ingress interface and one for the egress. Here are the configs using the table above:

#### HQ-POP Ingress Interface
```
telemetry ietf subscription 1
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet2’]/statistics
 source-address 10.0.254.2
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```

#### HQ-POP Egress Interface
```
telemetry ietf subscription 2
 encoding encode-kvgpb
 filter xpath /interfaces-ios-xe-oper:interfaces/interface[name='GigabitEthernet8’]/statistics
 source-address 10.0.254.2
 stream yang-push
 update-policy periodic 1000
 receiver ip address xx.xx.xx.xx 57000 protocol grpc-tcp
```




