# Tracing

[Jaeger](https://jaeger.readthedocs.io/en/latest/) is a distributed tracing system.
## Jaeger

It is disabled by default, to enable it provide configuration in the configuration file.

Key | Description | Default value 
--- | ----------- | ------------- 
jaeger-sampler-type | Sampler type | ""
jaeger-sampler-param | Sampler param | 0
jaeger-sampler-host-port | host:port of the jaeger agent | ""
jaeger-reporter-logspan | Logspan | false
jaeger-write-interval-ms | Write interval in milliseconds | 1000
jaeger-collector-healthcheck-host-port | host:port of the jaeger collector | ""

For more information about the configuration of jaeger, see the jaeger documentation.