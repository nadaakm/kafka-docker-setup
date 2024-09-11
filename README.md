# Docker Compose Setup for Kafka, Zookeeper, Debezium, NiFi, and Kafka Manager

This Docker Compose setup includes the following services:

- **Zookeeper**: A service used by Kafka for managing its distributed coordination.
- **Kafka**: The distributed event streaming platform.
- **Kafka Manager (Kafdrop)**: A web UI for monitoring and managing Kafka clusters.
- **Debezium**: A change data capture tool that streams data changes from PostgreSQL to Kafka.
- **Apache NiFi**: A data integration and processing platform.
- **Traefik**: A reverse proxy used to route traffic for the services.

## Table of Contents

- [Requirements](#requirements)
- [Usage](#usage)
- [Services Overview](#services-overview)
- [Volumes](#volumes)
- [Networks](#networks)
- [Troubleshooting](#troubleshooting)

## Requirements

- Docker and Docker Compose installed.
- Traefik for reverse proxy configuration.
- PostgreSQL JDBC driver (`postgresql-42.7.3.jar`) for NiFi.

## Usage

1. Clone the repository or copy the `docker-compose.yml` file.
2. Make sure to set the required environment variables such as:
   - `${MIGRATION_INSTANCE}`: A unique identifier for your container names and routes.
   - `${KAFKA_MANAGER_ROUTE}`, `${DEBEZIUM_ROUTE}`, and `${NIFI_ROUTE}`: DNS routes for accessing the respective services via Traefik.
3. Run the following command to start the services:

    ```bash
    docker-compose up -d
    ```

4. Access the services through the specified routes:
   - **Kafka Manager (Kafdrop)**: `<KAFKA_MANAGER_ROUTE>`
   - **Debezium**: `<DEBEZIUM_ROUTE>`
   - **NiFi**: `<NIFI_ROUTE>`

## Services Overview

### 1. **Zookeeper**

Zookeeper coordinates and manages Kafka's distributed cluster. The service is configured to run on port `2182`.

- **Data Directory**: `./data/zookeeper/data`
- **Datalog Directory**: `./data/zookeeper/datalog`

### 2. **Kafka**

Kafka is set up to run with two listeners: an internal one for inter-broker communication (`INTERNAL://:29093`) and an external one for clients (`EXTERNAL://:9093`).

- **Log Directories**: `./data/kafka/data`
- **Key Environment Variables**:
  - `KAFKA_MESSAGE_MAX_BYTES`: Maximum message size set to 100MB. (options)
  - `KAFKA_MAX_REQUEST_SIZE`: Maximum request size set to 100MB.  (options)

### 3. **Kafka Manager (Kafdrop)**

Kafka Manager is used for managing Kafka topics, consumers, and partitions. It is accessible through Traefik and configured to connect to Kafka on port `29093`.

- **UI Port**: `9000`
  
### 4. **Debezium**

Debezium is used for change data capture from a PostgreSQL database, streaming the data into Kafka topics.

- **Kafka Bootstrap Servers**: `kafka:29093`
- **Heap Memory**: `-Xms4g -Xmx8g` (can be changed based on server resources)
- **Converters**: JSON converters are used for key and value conversion.

### 5. **Apache NiFi**

NiFi is used for data ingestion, transformation, and routing. It connects to PostgreSQL using the JDBC driver located in the mounted volume.

- **HTTP Port**: `8080`
- **Heap Memory**: `1g` initial, `2g` max
- **Volumes**: Mounted repositories for database, flowfile, content, and provenance data.


## Volumes

The services use the following volumes for data persistence:

- `./data/zookeeper`: Zookeeper data and log storage.
- `./data/kafka`: Kafka log storage.
- NiFi directories:
  - `./nifi/database_repository`
  - `./nifi/flowfile_repository`
  - `./nifi/content_repository`
  - `./nifi/provenance_repository`
  - Ensure that these directories have `777` permissions to allow full access for all users.
- PostgreSQL JDBC driver: `./postgresql-42.7.3.jar`
  - Ensure that this file has `777` permissions for full access.
  - Ensure to change the propretie : nifi.content.repository.archive.enabled=true ==> false
   `curl` command:
    ```bash
    docker exec -it nifi-${MIGRATION_INSTANCE} bash
    cd conf
    nano nifi.properties
    ```

## Networks

- **Internal Network**: Used for communication between Kafka, Zookeeper, and Debezium.
- **Web Network**: An external network for Traefik to route external traffic.

## Troubleshooting

1. **Kafka Message Size Issues**: If you encounter errors related to message size, ensure that the `KAFKA_MESSAGE_MAX_BYTES`, `KAFKA_MAX_REQUEST_SIZE`, and `KAFKA_TOPIC_MAX_MESSAGE_BYTES` are configured correctly.
  

2. **Debezium Connector Management**: Ensure Debezium can connect to Kafka by checking the logs and verifying the correct Kafka broker URL and port are configured.

- **List Connectors**:
  - `GET` [https://DEBEZIUM_ROUTE/connectors/](https://DEBEZIUM_ROUTE/connectors/)
  - `curl` command:
    ```bash
    curl -i -X GET -H "Accept:application/json" https://DEBEZIUM_ROUTE/connectors/
    ```

- **Create a Connector**:
  - `POST` [https://DEBEZIUM_ROUTE/connectors/](https://DEBEZIUM_ROUTE/connectors/)
  - `curl` command:
    ```bash
    curl -i -X POST -H "Accept:application/json" -H "Content-Type:application/json" https://DEBEZIUM_ROUTE/connectors/ --data @debezium-config.json
    ```
  - Replace `@debezium-config1.json` with the path to your Debezium configuration JSON file.

- **Check Connector Status**:
  - `GET` [https://DEBEZIUM_ROUTE/connectors/<<connector_name>>/status](https://DEBEZIUM_ROUTE/connectors/<<connector_name>>/status)
  - `curl` command:
    ```bash
    curl -i -X GET -H "Accept:application/json" https://DEBEZIUM_ROUTE/connectors/<<connector_name>>/status
    ```
  - Replace `<<connector_name>>` with the actual name of the connector.

- **Delete a Connector**:
  - `DELETE` [https://DEBEZIUM_ROUTE/connectors/<<connector_name>>](https://DEBEZIUM_ROUTE/connectors/<<connector_name>>)
  - `curl` command:
    ```bash
    curl -i -X DELETE -H "Accept:application/json" https://DEBEZIUM_ROUTE/connectors/<<connector_name>>
    ```
  - Replace `<<connector_name>>` with the name of the connector you want to delete.

  **Debezium Connector Configuration**

  This configuration file is used for setting up the Debezium connector. Below are the key sections explained:

  - **transforms**: Specifies the transformations applied to the data.
  - **transforms.unwrap.type**: Extracts the new record state from the Debezium message.
  - **transforms.castLatitude.type**: Casts the latitude field to a FLOAT64 data type.
  - **transforms.timestampDate.type**: Converts the date field to a Timestamp format.

  ... and so on for each transformation.

  For more details, refer to the [Debezium documentation](https://debezium.io/documentation/).



3. **NiFi PostgreSQL Integration**: Ensure that the PostgreSQL JDBC driver is correctly mounted in NiFi, and that the database configurations in NiFi processors are accurate.


```markdown
## PostgreSQL Configuration 

To configure PostgreSQL for logical replication , add the following settings to the `postgresql.conf` file:

```ini
# Enable logical replication
wal_level = logical

# Set the maximum number of concurrent replication connections
max_wal_senders = 50

# Set the maximum number of replication slots
max_replication_slots = 100
```

### Steps to Apply Configuration on sfa13 and sfa8 databases to use this stack

1. **Locate the `postgresql.conf` file**:
   - The file is typically located in the PostgreSQL data directory. path/to/database (volume in docker-compose file)

2. **Edit the `postgresql.conf` file**:
   - Open the file in a text editor with sufficient permissions (using `sudo`).

3. **Add or Update the Configuration**:
   - Add the above settings to the file, or update existing values if they are already present.

4. **Restart PostgreSQL**:
   - Apply the changes by restarting the PostgreSQL service. You can do this with a command like:
     ```sh
     docker restart database
     ```

5. **Verify the Configuration**:
   - Ensure the settings are applied correctly by checking the PostgreSQL logs or using the following SQL command:
     ```sh
     docker exec -it pqsl -U user -d database database
     ```

     ```sql
     SHOW wal_level;
     SHOW max_wal_senders;
     SHOW max_replication_slots;
     ```