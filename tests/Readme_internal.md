# MM2 with the Redpanda

## MirrorSourceConnector for single local cluster deployment (no security)

### MirrorSourceConnector configuration

#### Base configuration

```json
{
    "name": "mm2-source-connector",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test01",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter"
    }
}
```

#### Configuration with SMT `InsertHeader`

```json
{
    "name": "mm2-source-connector",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test01",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "insertDcIdHeader",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc1-header",
        "transforms.insertDcIdHeader.value.literal": "DC1"
    }
}
```

#### Configuration with SMT `Filter`

```json
{
    "name": "mm2-source-connector",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test01",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc1-header",
        "predicates": "dc1-header",
        "predicates.dc1-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc1-header.name": "dc1-header"
    }
}
```

#### Configuration with SMT `Filter` add `InsertHeader`

```json
{
    "name": "mm2-source-connector",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test01",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter,insertDcIdHeader",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc1-header",
        "predicates": "dc1-header",
        "predicates.dc1-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc1-header.name": "dc1-header",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc1-header",
        "transforms.insertDcIdHeader.value.literal": "DC1"
    }
}
```

## MirrorSourceConnector for two clsuters deployment (no security) with single Kafka Connect

### DC1->DC2

```json
{
    "name": "mm2-source-dc1-dest-dc2",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test.*",
        "replication.factor": "1",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "dc2.testzone.local:19092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter,insertDcIdHeader",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc-header",
        "predicates": "dc-header",
        "predicates.dc-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc-header.name": "dc2-header",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc1-header",
        "transforms.insertDcIdHeader.value.literal": "DC1",
        "replication.policy.class": "org.apache.kafka.connect.mirror.IdentityReplicationPolicy",
        "sync.topic.configs.enabled": "false",
        "producer.override.bootstrap.servers": "dc2.testzone.local:19092"
    }
}
```

### DC2->DC1

```json
{
    "name": "mm2-source-dc2-dest-dc1",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test.*",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "dc2.testzone.local:19092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter,insertDcIdHeader",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc-header",
        "predicates": "dc-header",
        "predicates.dc-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc-header.name": "dc1-header",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc2-header",
        "transforms.insertDcIdHeader.value.literal": "DC2",
        "replication.policy.class": "org.apache.kafka.connect.mirror.IdentityReplicationPolicy",
        "sync.topic.configs.enabled": "false"
    }
}
```

### Replicated record from DC1 to DC2

MM2 adds custom header

```json
{
  "topic": "source.test01",
  "value": "Hello from DC1",
  "headers": [
    {
      "key": "dc1-header",
      "value": "DC1"
    }
  ],
  "timestamp": 1709758730631,
  "partition": 2,
  "offset": 0
}
```

### Filtered record from DC1 to DC2

MM2 filters/skips DC2 records in DC1 topic

```json
  
```

### MirrorSourceConnector opertons

### Delete connectors

```bash
curl -X DELETE http://localhost:8083/connectors/mm2-source-dc1-dest-dc2
curl -X DELETE http://localhost:8083/connectors/mm2-source-dc2-dest-dc1
```

### Get connector status

```bash
curl http://localhost:8083/connectors/mm2-source-dc1-dest-dc2/status | jq
curl http://localhost:8083/connectors/mm2-source-dc2-dest-dc1/status | jq
```

## MirrorSourceConnector for two clusters deployment (no security) with two Kafka Connect clusters

### DC1->DC2 with two Kafka Connect clusters

```json
{
    "name": "mm2-source-dc1-dest-dc2",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test.*",
        "replication.factor": "1",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "target.cluster.bootstrap.servers": "dc2.testzone.local:19092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter,insertDcIdHeader",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc-header",
        "predicates": "dc-header",
        "predicates.dc-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc-header.name": "dc2-header",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc1-header",
        "transforms.insertDcIdHeader.value.literal": "DC1",
        "replication.policy.class": "org.apache.kafka.connect.mirror.IdentityReplicationPolicy",
        "sync.topic.configs.enabled": "false"
    }
}
```

### DC2->DC1 with two Kafka Connect clusters

```json
{
    "name": "mm2-source-dc2-dest-dc1",
    "config": {
        "connector.class": "org.apache.kafka.connect.mirror.MirrorSourceConnector",
        "topics": "test.*",
        "replication.factor": "3",
        "source.cluster.alias": "source",
        "source.cluster.bootstrap.servers": "dc2.testzone.local:19092",
        "target.cluster.bootstrap.servers": "redpanda-0.testzone.local:31092",
        "key.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "value.converter": "org.apache.kafka.connect.converters.ByteArrayConverter",
        "transforms": "filter,insertDcIdHeader",
        "transforms.filter.type": "org.apache.kafka.connect.transforms.Filter",
        "transforms.filter.predicate": "dc-header",
        "predicates": "dc-header",
        "predicates.dc-header.type": "org.apache.kafka.connect.transforms.predicates.HasHeaderKey",
        "predicates.dc-header.name": "dc1-header",
        "transforms.insertDcIdHeader.type": "org.apache.kafka.connect.transforms.InsertHeader",
        "transforms.insertDcIdHeader.header": "dc2-header",
        "transforms.insertDcIdHeader.value.literal": "DC2",
        "replication.policy.class": "org.apache.kafka.connect.mirror.IdentityReplicationPolicy",
        "sync.topic.configs.enabled": "false"
    }
}
```

### Get connector status for two Kafka Connect clusters

```bash
curl http://localhost:8087/connectors/mm2-source-dc1-dest-dc2/status | jq

curl http://localhost:8083/connectors/mm2-source-dc2-dest-dc1/status | jq
```

### Delete connectors two Kafka Connect clusters

```bash
curl -X DELETE http://localhost:8087/connectors/mm2-source-dc1-dest-dc2
curl -X DELETE http://localhost:8083/connectors/mm2-source-dc2-dest-dc1
```

## Clients

### Consumers

#### DC1 consumer

```bash
rpk topic consume test01 -X brokers=redpanda-0.testzone.local:31092
```

#### DC2 consumer

```bash
rpk topic consume test01 -X brokers=dc2.testzone.local:19092
```

### Producers

### DC1 producer

```bash
rpk topic produce test01 -X brokers=redpanda-0.testzone.local:31092
```

### DC2 producer

```bash
rpk topic produce test01 -X brokers=dc2.testzone.local:19092
```
