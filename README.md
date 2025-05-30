# Kafka Connect Elasticsearch Source

[![CI](https://github.com/anshuman852/kafka-connect-elasticsearch-source/workflows/CI/badge.svg)](https://github.com/anshuman852/kafka-connect-elasticsearch-source/actions)
[![Release](https://github.com/anshuman852/kafka-connect-elasticsearch-source/workflows/Release/badge.svg)](https://github.com/anshuman852/kafka-connect-elasticsearch-source/releases)

Kafka Connect Elasticsearch Source: fetch data from elastic-search and sends it to kafka. The connector fetches only new
data using a strictly incremental / temporal field (like a timestamp or an incrementing id). It supports dynamic schema
and nested objects/ arrays.

## Features

- **Smart Bootstrap**: On first run, fetches only the latest N documents instead of all historical data
- **Incremental Processing**: Uses strictly incrementing fields for efficient data fetching
- **Dynamic Schema**: Supports nested objects and arrays
- **Configurable Filtering**: Whitelist/blacklist fields, JSON casting
- **Persistent Offsets**: Maintains cursor position across restarts

## Requirements

- Elasticsearch 6.x and 7.x
- Java >= 8
- Maven

## Installation

### From Release (Recommended)

1. Download the latest release from [GitHub Releases](https://github.com/anshuman852/kafka-connect-elasticsearch-source/releases)
2. Extract the plugin ZIP to your Kafka Connect plugins directory
3. Restart Kafka Connect

### From Source

Compile the project with:

```bash
mvn clean package -DskipTests
```

You can also compile and run both unit and integration tests (docker is mandatory) with:

```bash
mvn clean package
```

Copy the jar with dependencies from the target folder into connect classpath (
e.g ``/usr/share/java/kafka-connect-elasticsearch`` ) or set ``plugin.path`` parameter appropriately.

## Configuration

### Basic Example

Using kafka connect in distributed way, a sample config file to fetch ``my_awesome_index*`` indices and to produce
output topics with ``es_`` prefix:

```json
{       
  "name": "elastic-source",
   "config": {
             "connector.class":"com.github.dariobalinzo.ElasticSourceConnector",
             "tasks.max": "1",
             "es.host" : "localhost",
             "es.port" : "9200",
             "index.prefix" : "my_awesome_index",
             "topic.prefix" : "es_",
             "incrementing.field.name" : "@timestamp",
             "bootstrap.latest.docs.count": "10"
        }
}
```

### Kafka Connect Commands

To start the connector with curl:

```bash
curl -X POST -H "Content-Type: application/json" --data @config.json http://localhost:8083/connectors | jq
```

To check the status:

```bash
curl localhost:8083/connectors/elastic-source/status | jq
```

To stop the connector:

```bash
curl -X DELETE localhost:8083/connectors/elastic-source | jq
```

### Initial Cursor Bootstrap

When the connector starts and no previous offset is found, it will **not** ingest all historical data. Instead, it fetches the latest N documents (configurable via `bootstrap.latest.docs.count`, default 10), sets the cursor to the newest document, and only processes new documents from that point forward.

#### Property: `bootstrap.latest.docs.count`
- **Type:** integer
- **Default:** 10
- **Importance:** low
- **Description:** How many latest documents to fetch for initial cursor bootstrap. On first run (no offset), the connector will fetch the latest N documents (sorted by the incrementing field descending), set the cursor to the newest, and only process documents created after that.

**Config example:**
```properties
bootstrap.latest.docs.count=10
```

## Configuration Reference

### Elasticsearch Configuration

``es.host``
ElasticSearch host. Optionally it is possible to specify many hosts using ``;`` as separator (``host1;host2;host3``)

* Type: string
* Importance: high
* Dependents: ``index.prefix``

``es.port``
ElasticSearch port

* Type: string
* Importance: high
* Dependents: ``index.prefix``

``es.scheme``
ElasticSearch scheme (http/https)

* Type: string
* Importance: medium
* Default: ``http``

``es.user``
Elasticsearch username

* Type: string
* Default: null
* Importance: high

``es.password``
Elasticsearch password

* Type: password
* Default: null
* Importance: high

``incrementing.field.name``
The name of the strictly incrementing field to use to detect new records.

* Type: any
* Importance: high

``incrementing.secondary.field.name``
In case the main incrementing field may have duplicates, 
this secondary field is used as a secondary sort field in order 
to avoid data losses when paginating (available starting from versions >= 1.4).

* Type: any
* Importance: low

``bootstrap.latest.docs.count``
How many latest documents to fetch for initial cursor bootstrap. On first run (no offset), the connector will fetch the latest N documents (sorted by the incrementing field descending), set the cursor to the newest, and only process documents created after that.

* Type: int
* Default: 10
* Importance: low

``es.tls.truststore.location``
Elasticsearch truststore location

* Type: string
* Default: null
* Importance: high

``es.tls.truststore.password``
Elasticsearch truststore password

* Type: password
* Default: null
* Importance: high

``es.tls.keystore.location``
Elasticsearch keystore location

* Type: string
* Default: null
* Importance: high

``es.tls.keystore.password``
Elasticsearch keystore password

* Type: password
* Default: null
* Importance: high

### Connector Configuration

``poll.interval.ms``
Frequency in ms to poll for new data in each index.

* Type: string
* Default: 5000
* Importance: high

``batch.max.rows``
Maximum number of documents to include in a single batch when polling for new data.

* Type: string
* Default: 10000
* Importance: medium

``topic.prefix``
Prefix to prepend to index names to generate the name of the Kafka topic to publish data

* Type: string
* Importance: high

``index.prefix``
List of indices to include in copying.

* Type: string
* Default: ""
* Importance: medium

``index.names``
List of elasticsearch indices (es1,es2,es3)

* Type: string
* Default: null
* Importance: medium

### Filtering Configuration

``filters.whitelist``
Whitelist filter for fields (e.g. order.qty;order.price;status )

* Type: string
* Default: null
* Importance: low

``filters.blacklist``
Blacklist filter for fields (e.g. order.qty;order.price;status )

* Type: string
* Default: null
* Importance: low

``filters.json_cast``
Cast to json string instead of parsing nested objects (e.g. order.qty;order.price;status )

* Type: string
* Default: null
* Importance: low

``fieldname_converter``
Determine which name converter should be used for document fields: avro converter as standard

* Type: string
* Default: avro
* Importance: low

## Releases

Releases are automatically created when you push a tag starting with `v` (e.g., `v1.6.0`). The release will include:

- JAR file with dependencies
- Plugin ZIP file for easy installation
- Automated release notes

To create a release:

```bash
git tag v1.6.0
git push origin v1.6.0
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE) for details.

## Issues and Support

- Issues tracker: https://github.com/anshuman852/kafka-connect-elasticsearch-source/issues
- Feel free to open an issue to discuss new ideas or report bugs
