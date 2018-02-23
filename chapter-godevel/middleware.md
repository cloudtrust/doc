# Middlewares

The go microservices are built with go-kit and structured in 4 layers:
- transport
- endpoint
- component
- module

Each of these layer can contain middlewares. A typical Cloudtrust microservice will have at least logging, tracing, metric and error tracking middleware. Each of the middleware may be present 0 or more time per layer. For example in the [flaki-service](https://github.com/cloudtrust/flaki-service), there is a metric middleware at the endpoint, component and module level. This middleware tracks the response time of each level, so if there is performance issues or surprisingly slow response from the application, we can quickly pinpoint the source of the problem.

As it is common with go-kit microservices, the components are wired up in the main function. This is why Make...Middleware returns a function with the signature ```func(endpoint) endpoint```. 
For a middleware at transport level the signature will be ```func(grpc.Handler) grpc.Handler``` for gRPC, and ```func(http.Handler) http.Handler```for HTTP.
For middleware at component and module level, the signature is ```func(next Service) Service```, where Service is the interface that the service implements.

Below, there is an example of the middlewares at endpoint level.

## Logging
The logging middleware uses a go-kit JSON logger to log events. The logger is previously initialised with predefined key/value in the main function, i.e. ```MakeLoggingMiddleware(log.With(logger, "time", log.DefaultTimestampUTC, "caller", log.DefaultCaller "middleware", "endpoint", "service", <service name>)```.
Each call to log will log the previously defined key/values, as well as the new ones: correlation_id and took.

```go
// MakeLoggingMiddleware makes a logging middleware.
func MakeLoggingMiddleware(logger log.Logger) endpoint.Middleware {
	return func(next endpoint.Endpoint) endpoint.Endpoint {
		return func(ctx context.Context, req interface{}) (interface{}, error) {
			defer func(begin time.Time) {
				logger.Log("correlation_id", ctx.Value("correlation_id").(string), "took", time.Since(begin))
			}(time.Now())
			return next(ctx, req)
		}
	}
}
```

## Metrics
The metric middleware tracks the duration of the operations at the different levels. The results are saved in an Influx DB time series database.

```go
// MakeMetricMiddleware makes a middleware that measure the endpoints response time and
// send the metrics to influx DB.
func MakeMetricMiddleware(h metrics.Histogram) endpoint.Middleware {
	return func(next endpoint.Endpoint) endpoint.Endpoint {
		return func(ctx context.Context, req interface{}) (interface{}, error) {
			defer func(begin time.Time) {
				h.With("correlation_id", ctx.Value("correlation_id").(string)).Observe(time.Since(begin).Seconds())
			}(time.Now())
			return next(ctx, req)
		}
	}
}
```
## Tracing
The tracing middleware helps instrumenting the application by making traces. With them we can follow all request, subrequest between microservices.

```go
// MakeTracingMiddleware makes a middleware that handle the tracing with jaeger.
func MakeTracingMiddleware(tracer opentracing.Tracer, operationName string) endpoint.Middleware {
	return func(next endpoint.Endpoint) endpoint.Endpoint {
		return func(ctx context.Context, request interface{}) (interface{}, error) {
			if span := opentracing.SpanFromContext(ctx); span != nil {
				span = tracer.StartSpan(operationName, opentracing.ChildOf(span.Context()))
				defer span.Finish()

				span.SetTag("correlation_id", ctx.Value("correlation_id").(string))

				ctx = opentracing.ContextWithSpan(ctx, span)
			}
			return next(ctx, request)
		}
	}
}
```

## Error tracking
The error tracking middleware collects errors and send them to Sentry, where there is a dashboard where we can visualise the errors in real-time.

