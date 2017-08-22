# Step 3: Design the HTTP API

We need the following HTTP API methods, based on the task description in the task [Step 0](../step00/README.md)
## API Methods

1. POST / driver / - add driver
2. GET / driver /: id - get information about the driver
3. DELETE / driver /: id - remove the driver
4. GET / driver /: lat /: lon / nearest - get the nearest drivers.

We will use the [echo](http://echo.labstack.com) framework to build the API

We will need empty methods for this, which we will implement later.

```
func addDriver(c echo.Context) error {
	return nil
}
func getDriver(c echo.Context) error {
	return nil
}
func deleteDriver(c echo.Context) error {
	return nil
}
func nearestDrivers(c echo.Context) error {
	return nil
}
```

It turns out to this kind of code.
```
package main

import "github.com/labstack/echo"

func main() {
	e := echo.New()
	g := e.Group("/api")
	g.POST("/driver/", addDriver)
	g.GET("/driver/:id", getDriver)
	g.DELETE("/driver/:id", deleteDriver)
	g.GET("/driver/:lat/:lon/nearest", nearestDrivers)
	log.Fatal(e.Start(":9111"))
}

func addDriver(c echo.Context) error {
	return nil
}
func getDriver(c echo.Context) error {
	return nil
}
func deleteDriver(c echo.Context) error {
	return nil
}
func nearestDrivers(c echo.Context) error {
	return nil
}
```
### Requests and answers

We need the following structures to receive requests
```Go
type (
    Location struct {
        Latitude float64 `json:"lat"`
        Longitude float64 `json:"lon"`
    }
    Payload struct {
      Timestamp int64 `json:"timestamp"`
      DriverID int `json:"driver_id"`
      Location Location `json:"location"`
    }
)
```
We use the following to return the answers
```Go
type (
	// DefaultResponse used for returning default response
	DefaultResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
	}
	// DriverResponse returns driver
	DriverResponse struct {
		Success bool            `json:"success"`
		Message string          `json:"message"`
		Driver  int `json:"driver"`
	}
	// NearestDriverResponse returns nearest drivers
	NearestDriverResponse struct {
		Success bool              `json:"success"`
		Message string            `json:"message"`
		Drivers []int `json:"drivers"`
	}
)
```

## Congratulations!

We have the basic structures for receiving/sending data and the "stub" methods. We implement them in the [next](../step04/README.md) part

