## Step 7. Building the Storage

In this step, we will do a minimal data store and solve the problem of issuing the nearest driver. In this case, we need

1. Come up with an initial architecture
2. Make it consistent

Let me remind you that we need to store the following data
```Go
type (
  Location struct {
    Lat float64
    Lon float64
  }
  Driver struct {
    ID int 
    LastLocation Location
  }
)
```
So let us write them in `storage/storage.go`

Also, let me remind you that we need to implement the following features:

1. New() - to initialize the storage
2. Set(key, value) - to add or update an element
3. Delete(key) - to delete
4. Nearest(lat, lon) - to get the nearest elements
5. Get(key) - for get the driver

We make a structure for storing drivers
```Go
type DriverStorage struct {
  drivers map[int]*Driver
}
```
Let's write the methods for our repository.
```Go
package storage

type (
	// Location used for storing driver's location
	Location struct {
		Lat float64
		Lon float64
	}
	// Driver model to store driver data
	Driver struct {
		ID           int
		LastLocation Location
	}
)

// DriverStorage is main storage for our project
type DriverStorage struct {
	drivers map[int]*Driver
}

// New creates new instance of DriverStorage
func New() *DriverStorage {
	d := &DriverStorage{}
	d.drivers = make(map[int]*Driver)
	return d
}

// Set sets driver to storage by key
func (d *DriverStorage) Set(key int, driver *Driver) {
	return
}

// Delete removes driver from storage by key
func (d *DriverStorage) Delete(key int) error {
	return nil
}

// Get gets driver from storage and an error if nothing found
func (d *DriverStorage) Get(key int) (*Driver, error) {
	return nil, nil
}

// Nearest returns nearest drivers by locaion
func (d *DriverStorage) Nearest(lat, lon float64) ([]*Driver, error) {
	return nil, nil
}
```

We implement each of the methods

## Set

```Go
// Set sets driver to storage by key
func (d *DriverStorage) Set(key int, driver *Driver) {
	d.drivers[key] = driver
}
```

## Delete

```Go
// Delete removes driver from storage by key
func (d *DriverStorage) Delete(key int) error {
	driver, ok := d.drivers[key]
	if !ok {
		return errors.New("Driver does not exist")
	}
	delete(d.drivers, key)
	return nil
}
```

## Get

```Go
// Get gets driver from storage and an error if nothing found
func (d *DriverStorage) Get(key int) (*Driver, error) {
	driver, ok := d.drivers[key]
	if !ok {
		return nil, errors.New("Driver does not exist")
	}
	return driver, nil
}
```
But we also need tests to make sure that our code works. In order to write less code, we will supply the assert package
```Go
go get github.com/stretchr/testify/assert
```
Then, we will write a test
```Go
func TestStorage(t *testing.T) {
	s := New()
	driver := &Driver{
		ID: 1,
		LastLocation: Location{
			Lat: 1,
			Lon: 1,
		},
	}
	s.Set(driver.ID, driver)
	d, err := s.Get(driver.ID)
	assert.NoError(t, err)
	assert.Equal(t, d, driver)
	err = s.Delete(driver.ID)
	assert.NoError(t, err)
	d, err = s.Get(driver.ID)
	assert.Equal(t, err, errors.New("Driver does not exist"))
}
```

## Implementing the Nearest method

In general, there is a simple logic of the work, but we need to implement the work of the Nearest method. As an option, you can make the following algorithm work.

1. We pass by all drivers
2. Calculate the distance to the driver
3. If the distance is less than the specified radius, then we add the results to the array.

I suggest you implement this method by yourself. 
Skeleton of the method

```Go
// Nearest returns nearest drivers by locaion
func (d *DriverStorage) Nearest(radius, lat, lon float64) ([]*Driver, error) {
	return nil, nil
}
```
How to calculate the distance between two points. The distance is returned in meters.

```Go
// haversin(θ) function
func hsin(theta float64) float64 {
	return math.Pow(math.Sin(theta/2), 2)
}
// Distance function returns the distance (in meters) between two points of
//     a given longitude and latitude relatively accurately (using a spherical
//     approximation of the Earth) through the Haversin Distance Formula for
//     great arc distance on a sphere with accuracy for small distances
//
// point coordinates are supplied in degrees and converted into rad. in the func
//
// distance returned is METERS!!!!!!
// http://en.wikipedia.org/wiki/Haversine_formula
func Distance(lat1, lon1, lat2, lon2 float64) float64 {
	// convert to radians
	// must cast radius as float to multiply later
	var la1, lo1, la2, lo2, r float64
	la1 = lat1 * math.Pi / 180
	lo1 = lon1 * math.Pi / 180
	la2 = lat2 * math.Pi / 180
	lo2 = lon2 * math.Pi / 180
	r = 6378100 // Earth radius in METERS
	// calculate
	h := hsin(la2-la1) + math.Cos(la1)*math.Cos(la2)*hsin(lo2-lo1)
	return 2 * r * math.Asin(math.Sqrt(h))
}
```

Test for method

```Go
func TestNearest(t *testing.T) {
	s := New()
	s.Set(123, &Driver{
		ID: 123,
		LastLocation: Location{
			Lat: 1,
			Lon: 1,
		},
	})
	s.Set(666, &Driver{
		ID: 666,
		LastLocation: Location{
			Lat: 42.875799,
			Lon: 74.588279,
		},
	})
	drivers := s.Nearest(1000, 42.876420, 74.588332)
	assert.Equal(t, len(drivers), 1)
}
```

## Congratulations!

You implemented a method that looks for the nearest drivers by yourself. In the [next](../step08/README.md) lesson, we will understand how our method is effective and what can be done about it.

