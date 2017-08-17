## Step 12: Make the repository consistent. Introducing the LRU

We have primitive `sync/Mutex`, which is enough for the repository to be consistent. This is a normal lok. We will block our storage when we add or remove elements from it with its help.

Plus, we still need to do the Expire mechanism. To do this, we modify the `Driver` structure and add there `Expiration`. And to store the last points, we add to the driver LRU.

```Go
type Driver struct {
		ID           int
		LastLocation Location
		Expiration   int64
		Locations    *lru.LRU
}
```
Also, we'll do the `Expired()` method for the driver to know if the driver needs to be removed or not
```
// Expired returns true if the item has expired.
func (d *Driver) Expired() bool {
	if d.Expiration == 0 {
		return false
	}
	return time.Now().UnixNano() > d.Expiration
}

```
Extending the storage
```Go
	DriverStorage struct {
		mu        *sync.RWMutex # для синхронизации
		drivers   map[int]*Driver
		locations *rtreego.Rtree
		lruSize   int # для того, чтобы инициализировать хранилище по каждому водителю
	}
```
## The new one New
```Go
// New initializes now storage
func New(lruSize int) *DriverStorage {
	s := new(DriverStorage)
	s.drivers = make(map[int]*Driver)
	s.locations = rtreego.NewTree(2, 25, 50)
	s.mu = new(sync.RWMutex)
	s.lruSize = lruSize
	return s
}
```


## Set
This method works the same way for us. We add and update the data. This method returns an error, because there is no Update method in R-tree. But there is `Delete()` and `Insert()`. Therefore, before adding an element to the database, we will try to find out whether it exists or not. If it does not exist, we will initialize the LRU cache and then update all the data in the end. Also we will refuse the keys. We do not need them and we will only add drivers. We also have his ID in order to avoid duplication.

```Go
// Set an Driver to the storage, replacing any existing item.
func (s *DriverStorage) Set(driver *Driver)  {
	s.mu.Lock()
	defer s.mu.Unlock()

	d, ok := s.drivers[driver.ID]
	if !ok {
		d = driver
		cache, err := lru.New(s.lruSize)
		if err != nil {
			return errors.Wrap(err, "could not create LRU")
		}
		d.Locations = cache
		s.locations.Insert(d)
	}
	d.LastLocation = driver.LastLocation
	d.Locations.Add(time.Now().UnixNano(), d.LastLocation)
	d.Expiration = driver.Expiration

	s.drivers[driver.ID] = driver
	return nil
}
```
## Delete
The method is needed to delete data. The method will return an error if we try to delete data that is not in the database.
```Go
// Delete deletes a driver from storage. Does nothing if the driver is not in the storage
func (s *DriverStorage) Delete(id int) error {
	s.mu.Lock()
	defer s.mu.Unlock()
	d, ok := s.drivers[id]
	if !ok {
		return errors.New("does not exist")
	}
	deleted := s.locations.Delete(d)
	if deleted {
		delete(s.drivers, d.ID)
		return nil
	}
	return errors.New("could not remove item")
}
```

## Get
To get the driver by the key. Returns an error if there is no data for the key.
```Go
// Get returns driver by key
func (s *DriverStorage) Get(id int) (*Driver, error) {
	s.mu.RLock()
	defer s.mu.RUnlock()
	d, ok := s.drivers[id]
	if !ok {
		return nil, errors.New("does not exist")
	}
	return d, nil
}
```

Let us test all of the methods above

```Go
func TestDriverStorage(t *testing.T) {
	s := New(10)
	s.Set(&Driver{
		ID: 123,
		LastLocation: Location{
			Lat: 1,
			Lon: 1,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})
	d, err := s.Get(123)
	assert.NoError(t, err)
	assert.Equal(t, d.ID, 123)
	err = s.Delete(123)
	assert.NoError(t, err)
	d, err = s.Get(123)
	assert.Equal(t, err.Error(), "does not exist")

}
```
## Nearest

```Go
// Nearest returns nearest drivers
func (s *DriverStorage) Nearest(point rtreego.Point, count int) []*Driver {
	s.mu.Lock()
	defer s.mu.Unlock()

	results := s.locations.NearestNeighbors(count, point)
	var drivers []*Driver
	for _, item := range results {
		if item == nil {
			continue
		}
		drivers = append(drivers, item.(*Driver))
	}
	return drivers

}
```
And a test for it, which will show that the method works completely, because it will return the nearest drivers really and as much as necessary. The points are taken somewhere in the city center, next to BishkekPark (shopping mall).
```Go
func TestNearest(t *testing.T) {
	s := New(10)
	s.Set(&Driver{
		ID: 123,
		LastLocation: Location{
			Lat: 42.875799,
			Lon: 74.588279,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})
	s.Set(&Driver{
		ID: 321,
		LastLocation: Location{
			Lat: 42.875508,
			Lon: 74.588107,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})
	s.Set(&Driver{
		ID: 666,
		LastLocation: Location{
			Lat: 42.876106,
			Lon: 74.588204,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})
	s.Set(&Driver{
		ID: 2319,
		LastLocation: Location{
			Lat: 42.874942,
			Lon: 74.585908,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})
	s.Set(&Driver{
		ID: 991,
		LastLocation: Location{
			Lat: 42.875744,
			Lon: 74.584503,
		},
		Expiration: time.Now().Add(15).UnixNano(),
	})

	drivers := s.Nearest(rtreego.Point{42.876420, 74.588332}, 3)
	assert.Equal(t, len(drivers), 3)
	assert.Equal(t, drivers[0].ID, 123)
	assert.Equal(t, drivers[1].ID, 321)
	assert.Equal(t, drivers[2].ID, 666)
}
```

## DeleteExpired
Since we decided to do another Expire mechanism, we got a need to remove drivers that are done. The implementation is simple, we just go through all the elements.
```Go
// DeleteExpired removes all expired items from storage
func (s *DriverStorage) DeleteExpired() {
	now := time.Now().UnixNano()
	s.mu.Lock()
	defer s.mu.Unlock()

	for _, v := range s.drivers {
		if v.Expiration > 0 && now > v.Expiration {
			deleted := s.locations.Delete(v)
			if deleted {
				delete(s.drivers, v.ID)
			}
		}
	}
}
```
Let us test it.
```Go
func TestExpire(t *testing.T) {
	s := New(10)
	driver := &Driver{
		ID: 123,
		LastLocation: Location{
			Lat: 42.876420,
			Lon: 74.588332,
		},
		Expiration: time.Now().Add(-15).UnixNano(),
	}
	s.Set(driver)
	s.DeleteExpired()
	d, err := s.Get(123)
	assert.Error(t, err)
	assert.NotEqual(t, d, driver)

}
```

## Congratulations!
We made a consistent data store. In the [next](../step13/README.md) part, we will implement it in our API


