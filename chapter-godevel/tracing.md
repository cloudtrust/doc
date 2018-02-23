# Tracing

[Jaeger](https://jaeger.readthedocs.io/en/latest/) is a distributed tracing system.
## Jaeger

It is disabled by default, to enable it provide configuration in the configuration file.

Key | Description | Default value 
--- | ----------- | ------------- 
jaeger-sampler-type |  | ""
jaeger-sampler-param |  | 0
jaeger-sampler-url |  | ""
jaeger-reporter-logspan |  | false
jaeger-write-interval-ms | Flush interval in milliseconds | 1000