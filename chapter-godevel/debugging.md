# Debug

The golang pprof package serves runtime profiling data via the HTTP server (see [documentation](https://golang.org/pkg/net/http/pprof/)). If ```pprof-route-enabled``` is true, we enable the HTTP pprof routes.

Key | Description | Default value 
--- | ----------- | ------------- 
pprof-route-enabled | whether the pprof debug routes are enabled | true
