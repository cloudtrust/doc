# Logging

Cloudtrust has a centralised logging solution using the [Elastic Stack](https://www.elastic.co/webinars/introduction-elk-stack).
To achieve that, the logs are duplicated to Stdout and sent to a [Redis](https://redis.io/) server.

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
