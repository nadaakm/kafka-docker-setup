# Data Warehouse with Debezium, Kafka, Pinot, and Zookeeper

This repository provides a Docker Compose setup for a data warehouse environment, including PostgreSQL, Debezium, Apache Pinot, Kafka, and Zookeeper. These services are designed to work together to facilitate real-time data ingestion, processing, and storage, leveraging the power of streaming technologies like Debezium and Kafka, as well as distributed storage with Apache Pinot.

## Overview of Services

- **PostgreSQL (datawarehouse)**: A PostgreSQL database used to store data in the data warehouse.
- **Debezium**: Debezium captures row-level changes in PostgreSQL and streams them via Kafka.
- **Apache Pinot**: A real-time distributed OLAP datastore for low-latency analytics, useful for querying the data.
- **Zookeeper**: A centralized service for maintaining configuration and ensuring distributed systems' coordination.
- **Kafka**: A distributed streaming platform that acts as an intermediary between Debezium and other services.
- **Kafka Manager (Kafdrop)**: A user interface to manage and monitor Kafka clusters.

## Services in Docker Compose

### 1. **PostgreSQL (datawarehouse)**
- **Image**: `postgres:16`
- **Environment Variables**:
  - `POSTGRES_DB=datawarehouse`
  - `POSTGRES_PASSWORD=datawarehouse`
  - `POSTGRES_USER=datawarehouse`
- **Ports**: Exposes `5432` for PostgreSQL database access.
- **Volumes**: Stores data in the local directory `./volume_datawarehouse_postgres`.

### 2. **Debezium**
- **Image**: `debezium/connect:1.9`
- **Environment Variables**:
  - Connects to Kafka on port `29092`.
  - Uses JSON converters for both key and value data.
- **Dependencies**: Depends on both PostgreSQL (`datawarehouse`) and Kafka.

### 3. **Apache Pinot** (Zookeeper, Controller, Broker, Server)
- **Images**:
  - Zookeeper (`zookeeper:3.5.6`) to manage configuration.
  - Pinot Controller, Broker, Server (`apachepinot/pinot:1.0.0`) to handle distributed storage and querying.
- **Zookeeper Client Port**: `2181`.
- **Pinot Ports**: Internal and web network usage.
  
### 4. **Kafka**
- **Image**: `wurstmeister/kafka:latest`
- **Environment Variables**:
  - Broker ID `1`.
  - Connects to Zookeeper via `pinot-zookeeper`.
  - Exposes internal (`29092`) and external (`9092`) listeners.
  
### 5. **Kafka Manager (Kafdrop)**
- **Image**: `obsidiandynamics/kafdrop`
- **Port**: Accessible on port `9000`.
- **Depends on Kafka** for managing and monitoring Kafka topics.

## Usage Instructions

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd <repository-folder>
   ```

2. **Environment Setup**:
   Ensure the following environment variables are set in your `.env` file:
   ```env
   DATAWAREHOUSE_EXTERNAL_PORT=5432
   TAG_INSTANCE=my-instance
   DEBEZIUM_ROUTE=my-debezium-route.com
   CONTROLLER_ROUTE=my-controller-route.com
   BROKER_ROUTE=my-broker-route.com
   ZOOKEEPER_ROUTE=my-zookeeper-route.com
   KAFKA_ROUTE=my-kafka-route.com
   KAFKA_MANAGER_ROUTE=my-kafka-manager-route.com
   ```

3. **Start the Services**:
   Run the following command to start all the services using Docker Compose:
   ```bash
   docker-compose up -d
   ```

4. **Access the Services**:
   - PostgreSQL: `localhost:5432`
   - Pinot Controller: Visit `${CONTROLLER_ROUTE}`
   - Kafka Manager (Kafdrop): Visit `${KAFKA_MANAGER_ROUTE}`

5. **Shut Down the Services**:
   To stop the services, run:
   ```bash
   docker-compose down
   ```

## Volumes

The following volumes are used to persist data across container restarts:
- **PostgreSQL**: `./volume_datawarehouse_postgres`
- **Pinot Zookeeper**: `./pinot-docker-demo/zookeeper`
- **Pinot Server**: `./pinot-docker-demo/pinot/server`
- **Kafka**: `./pinot-docker-demo/kafka/data`

## Networks

- **Internal**: Used for internal communication between services (PostgreSQL, Kafka, Debezium, Pinot).
- **Web**: Externally exposed via Traefik.

## Additional Information

- This setup uses **Traefik** to manage routing for external services, providing HTTPS and load balancing.
- Ensure you have `docker-compose` installed to run this project.
- For testing Kafka configurations or querying Pinot, use the appropriate tools like `kafka-console-consumer` and `pinot-admin`.

---

## Contributing

Feel free to open an issue or submit pull requests to contribute to this project.

## License

This project is licensed under the MIT License.
```

This `README.md` provides an overview, instructions, and key details about the services involved. You can adjust the service URLs and environment variables as per your project setup.  