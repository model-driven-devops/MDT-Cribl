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
