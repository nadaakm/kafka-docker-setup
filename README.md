# Data Warehouse with Debezium, Kafka, and Zookeeper

This repository provides a Docker Compose setup for a data warehouse environment, including PostgreSQL, Debezium, , Kafka, and Zookeeper. These services are designed to work together to facilitate real-time data ingestion, processing, and storage, leveraging the power of streaming technologies like Debezium and Kafka.

## Overview of Services

- **PostgreSQL (datawarehouse)**: A PostgreSQL database used to store data in the data warehouse.
- **Debezium**: Debezium captures row-level changes in PostgreSQL and streams them via Kafka.
- **Zookeeper**: A centralized service for maintaining configuration and ensuring distributed systems' coordination.
- **Kafka**: A distributed streaming platform that acts as an intermediary between Debezium and other services.
- **Kafka Manager (Kafdrop)**: A user interface to manage and monitor Kafka clusters.
- **nifi**: For automating and managing the flow of data between systems. It is particularly used for data       ingestion, transformation, and routing across various data sources and destinations

## Services in Docker Compose

### 1. **PostgreSQL (datawarehouse)**
- **Image**: `postgres:16`
- **Environment Variables**:
  - `POSTGRES_DB= `
  - `POSTGRES_PASSWORD= `
  - `POSTGRES_USER= `
- **Ports**: Exposes `5432` for PostgreSQL database access.
- **Volumes**: Stores data in the local directory `./volume_datawarehouse_postgres`.

### 2. **Debezium**
- **Image**: `debezium/connect:1.9`
- **Environment Variables**:
  - Connects to Kafka on port `29092`.
  - Uses JSON converters for both key and value data.
- **Dependencies**: Depends on both PostgreSQL (`datawarehouse`) and Kafka.

### 3. **Zookeeper** (Zookeeper, Controller, Broker, Server)
- **Images**:
  - Zookeeper (`zookeeper:3.5.6`) to manage configuration.
- **Zookeeper Client Port**: `2181`.

  
### 4. **Kafka**
- **Image**: `wurstmeister/kafka:latest`
- **Environment Variables**:
  - Broker ID `1`.
  - Connects to Zookeeper via `zookeeper`.
  - Exposes internal (`29092`) and external (`9092`) listeners.
  
### 5. **nifi**
- **Image**: `apache/nifi:latest`
- **Port**: Accessible on port `8080`.
- **Depends on Kafka** for data transformation and ingestion.



## Usage Instructions

1. **Clone the Repository**:
   ```bash
   git clone <repository-url>
   cd <repository-folder>
   ```

2. **Environment Setup**:
   Ensure the following environment variables are set in your `.env` file:
   ```env
   MIGRATION_INSTANCE=my-instance
   DEBEZIUM_ROUTE=my-debezium-route.com
   NIFI_ROUTE=my-nifi-route.com
   KAFKA_MANAGER_ROUTE=my-kafka-manager-route.com
   ```

3. **Start the Services**:
   Run the following command to start all the services using Docker Compose:
   ```bash
   docker-compose up -d
   ```

4. **Access the Services**:
   - PostgreSQL: `localhost:5432`
   - Nifi: Visit `${NIFI_ROUTE}`
   - Kafka Manager (Kafdrop): Visit `${KAFKA_MANAGER_ROUTE}`

5. **Shut Down the Services**:
   To stop the services, run:
   ```bash
   docker-compose down
   ```

## Volumes

The following volumes are used to persist data across container restarts:
- **PostgreSQL**: `./volume_datawarehouse_postgres`
- **Zookeeper**: `./data/zookeeper/data`
- **Kafka**: `./pinot-docker-demo/kafka/data`
- **nifi**: `./nifi`

## Networks

- **Internal**: Used for internal communication between services (PostgreSQL, Kafka, Debezium, Pinot).
- **Web**: Externally exposed via Traefik.

## Additional Information

- This setup uses **Traefik** to manage routing for external services, providing HTTPS and load balancing.
- Ensure you have `docker-compose` installed to run this project.
- For testing Kafka configurations , use the appropriate tools like `kafka-console-consumer`.

---

## Contributing

Feel free to open an issue or submit pull requests to contribute to this project.

## License

This project is licensed under the MIT License.
```

This `README.md` provides an overview, instructions, and key details about the services involved. You can adjust the service URLs and environment variables as per your project setup.