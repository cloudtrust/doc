# Health HTTP routes

Each Go microservice exposes HTTP routes to monitor the application health.
There is one main route returning the application general health, and some subroutes with more details. 

## Main Health route
The main health route is always: ```<component-http-host-port>/health```, where \<component-http-host-port> is taken from the configuration.
A GET request to this route will return a JSON containing a list of the components and their status.

The status values are:
- `OK`: Everything is fine.
- `KO`: Something is broken. 
- `DEGRADED`: Not everything work as intended, but the main functionalities are still available: we are in a degraded mode. A typical example would be a service that works perfectly, but the database storing the metrics is down. We can let the application run, but we will lose the metrics.
- `DEACTIVATED`: The component is deactivated. For example if we do not want to record traces, metrics or errors, we can individually deactivates each component.

Note: The main route purpose is to display a succint report on the application status. The next section explain how to obtain more details about the components, e.g. if a status is `KO`, you probaly want to know why.

Example of returned message:
```
{
  "influx": "KO",
  "redis": "OK",
  "sentry": "DEACTIVATED",
  "jaeger": "OK"
}
```

## Detailed report

Each component has a dedicated route where the results of all health tests are available.
The subroutes are ```<component-http-host-port>/health/<name>```, where name is the name of the component. The names matche the ones returned by the main health route. For example for the message in the previous section, the available names are: "influx", "redis", "sentry" and "jaeger".

The subroutes return a JSON of the form:
```
[
  {
    "name": "ping",
    "duration": "906.881µs",
    "status": "OK"
  }
]
```
There is one entry per test, and each entry lists the name of the test, its duration, its status and an optional error.