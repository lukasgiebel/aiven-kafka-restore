# Aiven Kafka Restore Utility

Utility for restoring Kafka topic data backed up with [Aiven Kafka GCS Connector](https://github.com/aiven/aiven-kafka-connect-gcs)

[GitHub repository](https://github.com/aiven/aiven-kafka-restore)

## How It Works

This tool can download and restore Kafka messages stored in GCS by Aiven Kafka GCS Connector.

## Limitations

The are current limitations with the tool:

  * The tool supports the default filename format only (topic-partition-start_offset). Prefix for subdirectories is supported, however.
  * The tool supports the default record format only (key,value,offset,timestamp).
  * The tool is unable to restore consumer group offsets, but does output offset difference between the original and the new cluster. This difference can be used to adjust consumers to the correct location, but does require manual work.

## Usage

`python3 -m kafka_restore -c <configuration_file> -t <topic>`

## Configuration file format

Example configuration file contents are as follows:

```
{
    "kafka": {
        "kafka_url": "kafka-example.aivencloud.com:11397",
        "ssl_ca_file": "ca.crt",
        "ssl_access_certificate_file": "service.crt",
        "ssl_access_key_file": "service.key"
    },
    "object_storage": {
        "type": "gcs",
        "bucket": "example-backup-bucket",
        "credentials_file": "example-gcs-credentials.json"
    }
}
```

Optional prefix for subdirectories can be specified using `prefix` key under the object storage config.

## Restore procedure

  1. Stop all consumer and produced pipelines.
  2. Re-create the topic on a new cluster. The configuration must match the original topic in number of partitions.
  3. Run the restore tool to recover topic data. Record the offset differences between the original and the new cluster.
  4. Perform the above steps for all topics one wants to restore.
  5. Either configure consumers (re-)start from the latest offset.
  6. If consumer location needs to be adjusted and one has the offset information, the offset difference can be applied to adjust consumer start location.
  7. Once the consumers are running, you can (re-)enable producers as well.