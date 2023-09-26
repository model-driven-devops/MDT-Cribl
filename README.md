# MDT-Cribl
This repository contains the docker-compose file to launch the following containers:
- Telegraf: Telegraf listens and ingests Cisco MDT on port 57000, decodes, and sends as OpenTelemetry to Cribl
- Cribl: Cribl listens for OpenTelemetry on port 4317 and ships to Elastic.
- Elastic and Kibana: They do what they normally do. Kibana is set to be accessed on 5601.

Values can be changed using the .env file.

## Setting Up Cribl




