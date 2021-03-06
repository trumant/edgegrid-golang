# edgegrid

A Golang [Akamai](https://developer.akamai.com/api/) API client.

[Akamai {OPEN} EdgeGrid Authentication](https://developer.akamai.com/introduction/Client_Auth.html) for Golang.

`edgegrid` implements the [Akamai {OPEN} EdgeGrid Authentication](https://developer.akamai.com/introduction/Client_Auth.html) scheme for the Golang `net/http` package, and also offers a convenience API for interacting with the Akamai [GTM](https://developer.akamai.com/api/luna/config-gtm/overview.html) and [PAPI](https://developer.akamai.com/api/luna/papi/overview.html) APIs.

## Installation

Assuming Golang is properly installed:

```
go get github.com/comcast/edgegrid-golang
```

## Basic Usage

Basic usage assumes you've established up the following environment variables.

```
AKAMAI_EDGEGRID_HOST=
AKAMAI_EDGEGRID_CLIENT_TOKEN=
AKAMAI_EDGEGRID_ACCESS_TOKEN=
AKAMAI_EDGEGRID_CLIENT_SECRET=
```

To log request/response details:

```
AK_LOG=true
```

### GTMClient

```
import "edgegrid"

client := edgegrid.NewGTMClient()

// get all domains
client.Domains()

// get domain
client.Domain("example.akadns.net")

// create domain
domain := &Domain{
  // your data here; see resources.go
}
client.DomainCreate(domain)

// update domain
domain := &Domain{
  // your data here; see gtm_resources.go
}
client.DomainUpdate(domain)

// delete domain
// NOTE: this may 403, as it appears Akamai professional services is needed to perform DELETEs
client.DomainDelete("domain")

// get datacenters info for domain
client.DataCenters("domain")

// get datacenter info by dc id
client.DataCenter("domain", "123")

// get datacenter info by dc nickname
client.DataCenterByName("domain", "some_nickname")

// create datacenter
dc := &DataCenter{
  // your data here; see gtm_resources.go
}
client.DataCenterCreate("domain", dc)

// update datacenter
dc := &DataCenter{
  // your data here; see gtm_resources.go
}
client.DataCenterUpdate("domain", dc)

// delete datacenter by id
client.DataCenterDelete("domain", "123")

// get property info
client.Property("domain", "property")

// create property
prop := &Property{
  // your data here; see gtm_resources.go
}
client.PropertyCreate("domain", prop)

// adjust aspects of a property
prop := &Property{
  // your data here; see gtm_resources.go
}
client.PropertyUpdate("domain", prop)

// delete a property
client.PropertyDelete("domain", "property")
```

### PAPIClient

```
import "edgegrid"

client := edgegrid.NewPAPIClient()

// get all PAPI groups
client.Groups()
```

### Alternative Usage

In addition to the `edgegrid.GTMClient` and `edgegrid.PAPIClient` convenience clients, `edgegrid.Auth` offers a standalone implementation of the [Akamai {OPEN} EdgeGrid Authentication](https://developer.akamai.com/introduction/Client_Auth.html) scheme for the Golang net/http library and can be used independent of the `GTM` and `PAPI` convenience clients.

`edgegrid.Auth` offers a utility for establishing a valid `Authorization` header for use with authenticated requests to Akamai using your own HTTP client.

Example:

```
package main

import (
  "fmt"
  "github.com/comcast/edgegrid-golang"
  "io/ioutil"
  "net/http"
  "os"
)

func main() {
  client := &http.Client{}
  req, _ := http.NewRequest("GET", "<SOME_AKAMAI_URL>", nil)
  accessToken := os.Getenv("AKAMAI_EDGEGRID_ACCESS_TOKEN")
  clientToken := os.Getenv("AKAMAI_EDGEGRID_CLIENT_TOKEN")
  clientSecret := os.Getenv("AKAMAI_EDGEGRID_CLIENT_SECRET")
  params := edgegrid.NewAuthParams(req, accessToken, clientToken, clientSecret)
  auth := edgegrid.Auth(params)

  req.Header.Add("Authorization", auth)

  resp, _ := client.Do(req)
  contents, _ := ioutil.ReadAll(resp.Body)

  fmt.Printf("%s\n", string(contents))
}
```

## Run tests

Tests use the built-in `testing` package:

```
make test
```
