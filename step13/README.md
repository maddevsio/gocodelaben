# Step 13: Implement the repository in the API
We got this kind of structure right now. We need `sync.WaitGroup` in order to synchronize several goroutines, which we will launch later.


```Go
type API struct {
	database  *storage.DriverStorage
	waitGroup sync.WaitGroup
	echo      *echo.Echo
	bindAddr  string
}
```

## The new New
```Go
func New(bindAddr string, lruSize int) *API {
	a := &API{}
	a.database = storage.New(lruSize)
	a.echo = echo.New()
	g := a.echo.Group("/api")
	g.POST("/driver/", a.addDriver)
	g.GET("/driver/:id", a.getDriver)
	g.DELETE("/driver/:id", a.deleteDriver)
	g.GET("/driver/:lat/:lon/nearest", a.nearestDrivers)
	return a
}
```

## WaitStop
As a wrapper over a private waitgroup
```Go
func (a *API) WaitStop() {
	a.waitGroup.Wait()
}
```

### Remove expired

```Go
func (a *API) removeExpired() {
	for range time.Tick(1) {
		a.database.DeleteExpired()
	}
}
```
This method will block our main thread. You can try calling it and launch a web-server, for example. In this case, the web-server will not start. Here, `sync.WaitGroup` also comes to us on a gain

## Start
In this method, we simply launch the web-server and remove the rotten drivers in two goroutines. We lock the main thread using the `WaitStop()` method


```Go
func (a *API) Start() {
	a.waitGroup.Add(1)
	go func() {
		a.echo.Start(a.bindAddr)
		a.waitGroup.Done()
	}()
	a.waitGroup.Add(1)
	go a.removeExpired()
}

```

## addDriver
```Go
func (a *API) addDriver(c echo.Context) error {
	p := &Payload{}
	if err := c.Bind(p); err != nil {
		return c.JSON(http.StatusUnsupportedMediaType, &DefaultResponse{
			Success: false,
			Message: "Set content-type application/json or check your payload data",
		})
	}
	driver := &storage.Driver{}
	driver.ID = p.DriverID
	driver.LastLocation = storage.Location{
		Lat: p.Location.Latitude,
		Lon: p.Location.Longitude,
	}
	if err := a.database.Set(driver); err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: err.Error(),
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: false,
		Message: "Added",
	})
}

```

## getDriver
```Go
func (a *API) getDriver(c echo.Context) error {
	driverID := c.Param("id")
	id, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	d, err := a.database.Get(id)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: err.Error(),
		})
	}
	return c.JSON(http.StatusOK, &DriverResponse{
		Success: true,
		Message: "found",
		Driver:  d,
	})
}
```

## deleteDriver
```Go
func (a *API) deleteDriver(c echo.Context) error {
	driverID := c.Param("id")
	id, err := strconv.Atoi(driverID)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "could not convert string to integer",
		})
	}
	if err := a.database.Delete(id); err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: err.Error(),
		})
	}
	return c.JSON(http.StatusOK, &DefaultResponse{
		Success: true,
		Message: "removed",
	})
}
```

## NearestDrivers
```Go
func (a *API) nearestDrivers(c echo.Context) error {
	lat := c.Param("lat")
	lon := c.Param("lon")
	if lat == "" || lon == "" {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "empty coordinates",
		})
	}
	lt, err := strconv.ParseFloat(lat, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	ln, err := strconv.ParseFloat(lon, 64)
	if err != nil {
		return c.JSON(http.StatusBadRequest, &DefaultResponse{
			Success: false,
			Message: "failed convert float",
		})
	}
	drivers := a.database.Nearest(rtreego.Point{lt, ln}, 10)
	return c.JSON(http.StatusOK, &NearestDriverResponse{
		Success: false,
		Message: "found",
		Drivers: drivers,
	})
}

```
Well, at this stage in order not to produce unnecessary structures, we will change in `storage/storage.go` files, adding `json` tags

```Go
	Location struct {
		Lat float64 `json:"lat"`
		Lon float64 `json:"lon"`
	}
	Driver struct {
		ID           int      `json:"id"`
		LastLocation Location `json:"location"`
		Expiration   int64    `json:"-"`
		Locations    *lru.LRU `json:"-"`
	}

```
Well, in `main.go` you need to add the following
```Go
func main() {
	bindAddr := flag.String("bind_addr", ":8080", "Set bind address")
	size := flag.Int("lru_size", 20, "Set lru size per driver")
	flag.Parse()
	a := api.New(*bindAddr, *size)
	a.Start()
	a.WaitStop()
}
```
## Congratulations!
You're all done. Go to the [next](../step14/README.md) step
