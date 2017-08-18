# Step 0. Setting of the problem and preliminary steps


### A task

I want you to solve the problem with us, which is described in this [post](https://blog.maddevs.io/how-we-built-a-backend-system-for-uber-like-map-with-animated-cars-on-it-using-go-29d5dcd517a#.po7uwiqqk) in our blog. We need a database in order to store the tracks of drivers. At the same time, you need to store several tracks for each driver

### Data format
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
### Storage Requirements

1. Data should be added without duplication.
2. Need to store the last N points for each driver

Adding data will be via HTTP API

### HTTP API Requirements

1. Add the data to the storage
2. Get information on the driver
3. Get the nearest drivers by location

### Analysis of the problem

We will solve the problem evolutionarily. In the process of implementation, we will start with simple and primitive solutions that are not always effective in speed, but we will introduce certain primitives during the course, explaining them. The process will be something like this:

1. We solve the problem by a simple and blunt method
2. We think how it can be optimized
3. We study a new data structure and understand it
4. We are designing a solution with a new structure
4. Doing it
5. Writing tests

### How to work with the project

The project needs to clone the project for yourself.

``` 
cd $GOPATH
mkdir -p src/github.com/maddevsio
cd src/github.com/maddevsio
git clone git@github.com:maddevsio/gocodelabru.git
cd gocodelab

``` 

## On the structure of the project

Codelab is divided into several steps. Each subsequent step can contain an example of what you should get after the previous one. These are kind of checkpoints to check if everything is done correctly or not.

## Congratulations!

We have learned what task we will solve and how to solve it. Let's start the task in the [next step](../step01/README.md)
