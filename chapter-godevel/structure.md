# Repository structure

The Cloudtrust go git repositories follow the structure recommended by the [go best practices](https://peter.bourgon.org/go-best-practices-2016/#repository-structure). For a microservice, it will look like:

```
github.com/cloudtrust/<repo>/
.
├── .gitignore
├── .travis.yml             // Configuration of the travis CI
├── Gopkg.lock              // Dep lock
├── Gopkg.toml              // Dep manifest
├── LICENSE
├── README.md
├── script               
│   └── build.sh            // Build script
├── bin         
│   └── <name>              // Executable produced by build.sh 
├── cmd
│   ├── <name>.go           // main.go
│   └── <name>_test.go
├── conf                    // Configuration files
│   └── DEV
│       └── <name>.yml
└─── pkg
    ├── <name>
    │   ├── component
    │   │   ├── middleware.go
    │   │   ├── middleware_test.go
    │   │   ├── service.go
    │   │   └── service_test.go
    │   ├── endpoint
    │   │   ├── endpoint.go
    │   │   ├── endpoint_test.go
    │   │   ├── middleware.go
    │   │   └── middleware_test.go
    │   ├── module
    │   │   ├── middleware.go
    │   │   ├── middleware_test.go
    │   │   ├── service.go
    │   │   └── service_test.go
    │   └── transport
    │       ├── flatbuffer
    │       │   └── <name>.fbs
    │       ├── grpc
    │       │   ├── grpc.go
    │       │   ├── grpc_test.go
    │       │   ├── middleware.go
    │       │   └── middleware_test.go
    │       └── http
    │           ├── http.go
    │           ├── http_test.go
    │           ├── middleware.go
    │           └── middleware_test.go
    ├── health                            // Health checks
    │   ├── component
    │   │   └ ...
    │   ├── endpoint
    │   │   └ ...
    │   ├── module
    │   │   └ ...
    │   └── transport
    │       └ ...
    └── middleware                        // Middlewares common between services
        └ ...

```