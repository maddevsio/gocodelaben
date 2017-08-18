# Step 8. Writing the first benchmark: why do we need it

As you can guess, the method you implemented is not quite effective. And in order to make sure of this, we'll write a benchmark in storage/storage_test.go
```Go
func BenchmarkNearest(b *testing.B) {
	s := New()
	for i := 0; i < 100; i++ {
		s.Set(i, &Driver{
			ID: i,
			LastLocation: Location{
				Lat: float64(i),
				Lon: float64(i),
			},
		})
	}
	for i := 0; i < b.N; i++ {
		s.Nearest(1000, 123, 123)
	}
}
```
We'll check the work on 100, 1000, 10,000 elements in the storage
```Go
cd storage
go test -bench=.
```
For 100 elements
```
BenchmarkNearest-4         50000             24002 ns/op
PASS
ok      github.com/maddevsio/gocodelabru/step09/storage 1.460s
```
For 1000 elements
```Go
BenchmarkNearest-4          5000            272552 ns/op
PASS
ok      github.com/maddevsio/gocodelabru/step09/storage 1.402s
```
For 10000 elements
```Go
BenchmarkNearest-4           500           2799431 ns/op
PASS
ok      github.com/maddevsio/gocodelabru/step09/storage 1.714s
```

We can make a conclusion that the more elements in the storage, the longer we are looking for the nearest drivers.

## Congratulations!
You now know how to write benchmarks. In the [next](../step09/README.md) step we will optimize the work of the method of issuing the nearest drivers