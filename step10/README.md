## Step 10. Implement the LRU (Part 1)
For making an LRU storage, first I have to tell what it is.

**LRU (least recently used)** — is a data caching algorithm that displacing out values ​​that have not been requested for the longest time. This mechanism is convenient because we, for example, initialize the cache for 20 elements. And as soon as we try to add the 21st, the longest unused item will be deleted.


This mechanism is convenient because we implement the logic only in one place. We'll create it in the `storage/lru` package, create a new folder, create two files: `storage/lru/lru.go` and `storage/lru/lru_test.go` with a `package lru` content


## Data Structures
We need to think about how to implement the cache. We need some list of elements in which we could move the elements both upside down and take the last one. Plus, we need to be able to delete values ​​from this list. It seems that [container/list](https://golang.org/pkg/container/list/) suits us.

It turns out that we can describe the cache as follows

```Go
type LRU struct {
  size int
  evictList *list.List
  items map[interface{}]*list.Element
}
```
We also need a structure to store data in the list and on the map.
```Go
// entry used to store value in evictList
type entry struct {
	key   interface{}
	value interface{}
}
```

In the file `lru/lru.go` the following code
```Go
package lru

import (
	"container/list"
)

type (
	LRU struct {
		size      int
		evictList *list.List
		items     map[interface{}]*list.Element
	}
	// entry used to store value in evictList
	entry struct {
		key   interface{}
		value interface{}
	}
)

```

## New
In order to create a cache, we need to transfer its size in the parameters and initialize all the storage structures. The following is obtained
```Go
// New initialized a new LRU with fixed size
func New(size int) (*LRU, error) {
	if size <= 0 {
		return nil, errors.New("Size must be greater than 0")
	}
	c := &LRU{
		size:      size,
		evictList: list.New(),
		items:     make(map[interface{}]*list.Element),
	}
	return c, nil
}
```

## Add

```Go
// Add adds a value to the cache. Return true if eviction occured
func (l *LRU) Add(key, value interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.evictList.MoveToFront(ent)
		ent.Value.(*entry).value = value
		return false
	}
	ent := &entry{key, value}
	entry := l.evictList.PushFront(ent)
	l.items[key] = entry
}

```
Add does two things: adds and updates the value by key. In this case, using the container / list API, it manages the position of the item in the list. It's not enough just to delete an item if we items in our cache more than its size.

## Removing the least-used item
```Go
func (l *LRU) removeOldest() {
	ent := l.evictList.Back()
	if ent != nil {
		l.removeElement(ent)
	}
}
func (l *LRU) removeElement(e *list.Element) {
	l.evictList.Remove(e)
	kv := e.Value.(*entry)
	delete(l.items, kv.key)
}
```
According to the establishment logic, we need to remove only the last element from the list. You will need to delete it from both our list and the map.

Let's return to adding an element and insert the oldest element if we exceed the size of the storage. We get the following code

```Go
// Add adds a value to the cache. Return true if eviction occured
func (l *LRU) Add(key, value interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.evictList.MoveToFront(ent)
		ent.Value.(*entry).value = value
		return false
	}
	ent := &entry{key, value}
	entry := l.evictList.PushFront(ent)
	l.items[key] = entry
	evict := l.evictList.Len() > l.size
	if evict {
		l.removeOldest()
	}
	return evict
}
```
We will write a test on it in `lru/lru_test.go`
```Go
// Test that Add returns true/false if an eviction occurred
func TestLRU_Add(t *testing.T) {

	l, err := New(1)
	if err != nil {
		t.Fatalf("err: %v", err)
	}

	if l.Add(1, 1) == true {
		t.Errorf("should not have an eviction")
	}
	if l.Add(2, 2) == false {
		t.Errorf("should have an eviction")
	}
}

```
## Len
Everything is simple with Len (). We need to return only the length of the list to find out, how many elements we have now in the cache
```Go
// Len returns the number of items in cache
func (l *LRU) Len() int {
	return l.evictList.Len()
}
```

# Congratulations!

You realized the creation, addition and removal of the oldest element from the cache and wrote a test to add an element. We will continue to work in the [next part](../step11/README.md)
