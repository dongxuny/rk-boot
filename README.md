# rk-boot
[![build](https://github.com/rookie-ninja/rk-boot/actions/workflows/ci.yml/badge.svg)](https://github.com/rookie-ninja/rk-boot/actions/workflows/ci.yml)
[![codecov](https://codecov.io/gh/rookie-ninja/rk-boot/branch/master/graph/badge.svg?token=BZ6KWGAXNP)](https://codecov.io/gh/rookie-ninja/rk-boot)
[![Go Report Card](https://goreportcard.com/badge/github.com/rookie-ninja/rk-boot)](https://goreportcard.com/report/github.com/rookie-ninja/rk-boot)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Frookie-ninja%2Frk-boot.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Frookie-ninja%2Frk-boot?ref=badge_shield)

## Important note about V2
> RK family is bumping up to V2 which will not be full compatible with V1 including documentation. Please refer to V1 as needed.
>
> From V2, rk-boot will not include any of dependencies which implement rkentry.Entry. As a result, user need to pull rk-boot and rk-xxx or user implemented Entry manually.
> 
> We think it will be a better experience for dependency management.
> 
> For example, if we hope to start Gin web framework, we need to pull both of rk-boot and rk-gin. [example](example/gin)

## Official document
V2 documentation will be updated soon. Please refer to github docs now.

## Concept
rk-boot is a library which support bootstrapping server at runtime via YAML file. It is a little like [spring boot](https://spring.io/quickstart) way.

![arch](docs/img/boot-arch.png)

### Goal
We hope user can achieve bellow goals while designing/implementing microservice.

- 1: **Decide** which dependencies to use. (For example, MySQL, Redis, AWS, Gin are the required dependencies.)
- 2: **Add** dependencies in boot.yaml (where rk-boot will automatically initiate dependency client instances.)
- 3: **Implement** your own codes (without caring about logging, metrics and tracing of dependency client.)
- 4: **Monitor** service and dependency (via a standard dashboard.)

We are planning to achieve this goal by unify dependency input(by boot.yaml) and output(logging format, metrics format etc).

We will add more bootstrapper for popular third-party dependencies.

### Why do I want it?
- Build application with unified project layout at enterprise level .
- Build API with the unified format of logging, metrics, tracing, authorization at enterprise level.
- Make application replace core dependencies quickly.
- Save learning time of writing initializing procedure of popular frameworks and libraries.
- User defined Entry for customization.

## Features V2
We will migrate dependencies from v1 to v2 as quick as possible.

| Category      | Name                                                           | rk-boot version | Installation                              | Example                      |
|---------------|----------------------------------------------------------------|-----------------|-------------------------------------------|------------------------------|
| Web Framework | [gin-gonic/gin](https://github.com/gin-gonic/gin)              | V2              | go get github.com/rookie-ninja/rk-gin/v2  | [gin-example](example/gin)   |
| Web Framework | [gRPC](https://grpc.io/docs/languages/go/)                     | V2              | go get github.com/rookie-ninja/rk-grpc/v2 | [grpc-example](example/grpc) |
| Web Framework | [labstack/echo](https://github.com/labstack/echo)              | V1              |                                           |                              |
| Web Framework | [gogf/gf](https://github.com/gogf/gf)                          | V1              |                                           |                              |
| Web Framework | [gofiber/fiber](https://github.com/gofiber/fiber)              | V1              |                                           |                              |
| Web Framework | [zeromicro/go-zero](https://github.com/zeromicro/go-zero)      | V1              |                                           |                              |
| Web Framework | [gorilla/mux](https://github.com/gorilla/mux)                  | V1              |                                           |                              |
| Database ORM  | [MySQL](https://github.com/rookie-ninja/rk-db/mysql)           | V1              |                                           |                              |
| Database ORM  | [SQLite](https://github.com/rookie-ninja/rk-db/sqlite)         | V1              |                                           |                              |
| Database ORM  | [SQL Server](https://github.com/rookie-ninja/rk-db/sqlserver)  | V1              |                                           |                              |
| Database ORM  | [postgreSQL](https://github.com/rookie-ninja/rk-db/postgres)   | V1              |                                           |                              |
| Database ORM  | [ClickHouse](https://github.com/rookie-ninja/rk-db/clickhouse) | V1              |                                           |                              |
| Caching       | [Redis](https://github.com/rookie-ninja/rk-cache)              | V1              |                                           |                              |

## Quick Start for Gin
We will start [gin-gonic/gin](https://github.com/gin-gonic/gin) server with rk-boot.

- Installation
[rk-boot](https://github.com/rookie-ninja/rk-boot) is required one for all RK family. We pulled rk-gin as dependency since we are testing GIN.

```shell
go get github.com/rookie-ninja/rk-boot/v2
go get github.com/rookie-ninja/rk-gin/v2
```

- boot.yaml
```yaml
---
gin:
  - name: greeter                                          # Required
    port: 8080                                             # Required
    enabled: true                                          # Required
    sw:
      enabled: true                                        # Optional, default: false
    docs:
      enabled: true                                        # Optional, default: false
```

- main.go
```go
// Copyright (c) 2021 rookie-ninja
//
// Use of this source code is governed by an Apache-style
// license that can be found in the LICENSE file.

package main

import (
	"context"
	"github.com/gin-gonic/gin"
	"github.com/rookie-ninja/rk-boot/v2"
	"github.com/rookie-ninja/rk-gin/v2/boot"
	"net/http"
)

// @title Swagger Example API
// @version 1.0
// @description This is a sample rk-demo server.
// @termsOfService http://swagger.io/terms/

// @securityDefinitions.basic BasicAuth

// @contact.name API Support
// @contact.url http://www.swagger.io/support
// @contact.email support@swagger.io

// @license.name Apache 2.0
// @license.url http://www.apache.org/licenses/LICENSE-2.0.html
func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Register handler
	entry := rkgin.GetGinEntry("greeter")
	entry.Router.GET("/v1/hello", hello)

	// Bootstrap
	boot.Bootstrap(context.TODO())

	boot.WaitForShutdownSig(context.TODO())
}

// @Summary Hello
// @Id 1
// @Tags Hello
// @version 1.0
// @produce application/json
// @Success 200 string string
// @Router /v1/hello [get]
func hello(ctx *gin.Context) {
	ctx.JSON(http.StatusOK, gin.H{
		"message": "hello!",
	})
}
```

- validate
```shell script
$ go run main.go

$ curl -X GET localhost:8080/v1/greeter
{"message":"hello!"}

$ curl -X GET localhost:8080/rk/v1/ready
{
  "ready": true
}

$ curl -X GET localhost:8080/rk/v1/alive
{
  "alive": true
}
```

- Swagger UI: [http://localhost:8080/sw](http://localhost:8080/sw)

![image](example/gin/docs/img/simple-sw.png)

- Docs UI via: [http://localhost:8080/docs](http://localhost:8080/docs)

![image](example/gin/docs/img/simple-docs.png)

## Development Status: Stable

## Build instruction
Simply run make all to validate your changes. Or run codes in example/ folder.

- make all
If proto or files in boot/assets were modified, then we need to run it.

## Test instruction
Run unit test with **make test** command.

github workflow will automatically run unit test and golangci-lint for testing and lint validation.

## Contributing
We encourage and support an active, healthy community of contributors &mdash;
including you! Details are in the [contribution guide](CONTRIBUTING.md) and
the [code of conduct](CODE_OF_CONDUCT.md). The rk maintainers keep an eye on
issues and pull requests, but you can also report any negative conduct to
lark@rkdev.info.

Released under the [Apache 2.0 License](LICENSE).


## License
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Frookie-ninja%2Frk-boot.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Frookie-ninja%2Frk-boot?ref=badge_large)