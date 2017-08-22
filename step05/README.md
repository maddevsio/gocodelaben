## Step 5. Fracture into small parts

That's when we already have some code, we can endure everything from main.go. Now the first candidate is all the code that belongs to the API. Let's take it out.

We have the following code
```Go
package main

import (
	"log"
	"net/http"
	"strconv"

	"github.com/labstack/echo"
)

type (
	Location struct {
		Latitude  float64 `json:"lat"`
		Longitude float64 `json:"lon"`
	}
	Payload struct {
		Timestamp int64    `json:"timestamp"`
		DriverID  int      `json:"driver_id"`
		Location  Location `json:"location"`
	}
	
	DefaultResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
	}
	
	DriverResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
		Driver  int    `json:"driver"`
	}
	
	NearestDriverResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
		Drivers []int  `json:"drivers"`
	}
)

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
	p := &Payload{}
	if err := c.Bind(p); err != nil {
		return c.JSON(http.StatusUnsupportedMediaType, &DefaultResponse{
			Success: false,
			Message: "Set content-type application/json or check your payload data",
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: false,
		Message: "Added",
	})
}
func getDriver(c echo.Context) error {
	driverID := c.Param("id")
	id, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	return c.JSON(http.StatusOK, &DriverResponse{
		Success: true,
		Message: "found",
		Driver:  id,
	})
}

func deleteDriver(c echo.Context) error {
	driverID := c.Param("id")
	_, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: true,
		Message: "removed",
	})
}
func nearestDrivers(c echo.Context) error {
	lat := c.Param("lat")
	lon := c.Param("lon")
	if lat == "" || lon == "" {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "empty coordinates",
		})
	} 
	_, err := strconv.ParseFloat(lat, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	_, err = strconv.ParseFloat(lon, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	// TODO: Add nearest
	return c.JSON(http.StatusOK, &NearestDriverResponse{
		Success: false,
		Message: "found",
	})
}
```
There are also not enough tests for it. Plus, it is desirable to configure the port somehow, on which our web-server will start.

Let us create an `api` folder and move all the code that applies to the API into `api/api.go`. But first, we need a structure that we will use in the future. For example, in order to connect to the API our database, which we will do later. But right now we will store a copy of echo here, and the address of listened application
```Go
type API struct {
  echo *echo.Echo
  bindAddr string
}
```
In first, this simple structure is enough for us. We will need to move this using class methods. Well, and make a method for creating a new copy of the API.

## New or create an API

In this method, we initialize all our dependencies and configure routs.
```Go
func New(bindAddr string) *API {
	a := &API{}
	a.echo = echo.New()
  	a.bindAddr = bindAddr
	g := a.echo.Group("/api")
	g.POST("/driver/", a.addDriver)
	g.GET("/driver/:id", a.getDriver)
	g.DELETE("/driver/:id", a.deleteDriver)
	g.GET("/driver/:lat/:lon/nearest", a.nearestDrivers)
	return a
}
```

## Start

This method will be needed in order to start our application
```Go
func (a *API) Start() error {
	return a.echo.Start(a.bindAddr)
}
```
## Other methods

All other methods must be reduced to the form `func (a *API) addDriver(c echo.Context) error`

```Go
func (a *API) addDriver(c echo.Context) error {
	p := &Payload{}
	if err := c.Bind(p); err != nil {
		return c.JSON(http.StatusUnsupportedMediaType, &DefaultResponse{
			Success: false,
			Message: "Set content-type application/json or check your payload data",
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: false,
		Message: "Added",
	})
}
func (a *API) getDriver(c echo.Context) error {
	driverID := c.Param("id")
	id, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	return c.JSON(http.StatusOK, &DriverResponse{
		Success: true,
		Message: "found",
		Driver:  id,
	})
}

func (a *API) deleteDriver(c echo.Context) error {
	driverID := c.Param("id")
	_, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: true,
		Message: "removed",
	})
}
func (a *API) nearestDrivers(c echo.Context) error {
	lat := c.Param("lat")
	lon := c.Param("lon")
	if lat == "" || lon == "" {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "empty coordinates",
		})
	}
	_, err := strconv.ParseFloat(lat, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	_, err = strconv.ParseFloat(lon, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	// TODO: Add nearest
	return c.JSON(http.StatusOK, &NearestDriverResponse{
		Success: false,
		Message: "found",
	})
}
```

And we'll move all our models to `api/models.go`
```Go
package api

type (
	Location struct {
		Latitude  float64 `json:"lat"`
		Longitude float64 `json:"lon"`
	}
	Payload struct {
		Timestamp int64    `json:"timestamp"`
		DriverID  int      `json:"driver_id"`
		Location  Location `json:"location"`
	}
	// Структура для возврата ответа по умолчанию
	DefaultResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
	}
	// Для возврата ответа, когда мы запрашиваем водителя
	DriverResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
		Driver  int    `json:"driver"`
	}
	// Для возврата ближайших водителей
	NearestDriverResponse struct {
		Success bool   `json:"success"`
		Message string `json:"message"`
		Drivers []int  `json:"drivers"`
	}
)

```

In main.go we have only this code
```Go
func main() {
	e := echo.New()
	g := e.Group("/api")
	g.POST("/driver/", addDriver)
	g.GET("/driver/:id", getDriver)
	g.DELETE("/driver/:id", deleteDriver)
	g.GET("/driver/:lat/:lon/nearest", nearestDrivers)
	log.Fatal(e.Start(":9111"))
}
```
Which have to be modified to the kind of this
```
func main() {
	a := api.New(":9111")
	log.Fatal(a.Start())
}
```

## Congratulations!

We split our program into several parts. We'll work with the flags, the application configuration, and the Makefile in the [next](../step06/README.md) part.

