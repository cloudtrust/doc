# Instrumenting

Each service can record metrics and send them to an [Influx](https://www.influxdata.com/time-series-platform/influxdb/) time series DB. We can record several types of metric, like for example the number of HTTP/GRPC requests, the services response time,...

## Influx

By default the influx configuration is empty, which mean that the metrics are disabled. To activate them, provide configuration for infux in the configuration file.

Key | Description | Default value 
--- | ----------- | ------------- 
influx-url | Influx URL | ""
influx-username | InfluxDB username | ""
influx-password | InfluxDB password | ""
influx-database | InfluxDB database name | ""
influx-precision | Write precision of the points | ""
influx-retention-policy | Retention policy of the points | ""
influx-write-consistency | Number of servers required to confirm write | ""
influx-write-interval-ms | Flush interval in milliseconds | 1000