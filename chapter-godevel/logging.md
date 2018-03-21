# Logging

Cloudtrust has a centralised logging solution using the [Elastic Stack](https://www.elastic.co/webinars/introduction-elk-stack).
To achieve that, the logs are duplicated to Stdout and sent to a [Redis](https://redis.io/) server.

## Standardised Keys

It is essential to have normalised keys, so when can easily search for them in Elasticsearch.

Key | Description
--- | -----------
ts | UTC timestamp (use log.DefaultTimestampUTC)
caller | caller (use log.DefaultCaller)
component_name | name of the component (e.g. keycloak_bridge)
component_version | version of the component (e.g. 1.0)
environment | environment (e.g. DEV)
git_commit | the commit that was built
error | the error
msg | the log message
level | the log level
unit | the unit that logs (e.g. config, influx, ...)
service | the service that logs (e.g. health)
middleware | the middleware that logs
module | the module that logs
transport | the transport layer (i.e. http or grpc)
took | the response time

## Redis

By default the Redis configuration is empty, which mean we do not use Redis. The loggers only log to stdout. 
If the Redis configuration keys are present in the configuration file, the logs are duplicated to stdout and Redis.
Note that the redis logs are formatted in [Logstash](https://www.elastic.co/products/logstash) format.

Key | Description | Default value
--- | ----------- | -------------
redis-host-port | Redis server host:port | ""
redis-password | Redis password | ""
redis-database | Redis database | 0
redis-write-interval-ms | Write interval in milliseconds | 1000
