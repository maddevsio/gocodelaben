## Step 11. Implement the LRU (Part 2)
In the last part, we implemented the implementation of the methods `New`, `Add`, `removeOldest`, `removeElement`, `Len` and wrote a test for the `Add` method of work. In this part, we will continue to build the `LRU` cache.


## Purge
The task of this method is to completely delete all data in the repository. To do this, we just need to go through all the elements in the map and delete them
```Go
// Purge completely clears cache
func (l *LRU) Purge() {
	for k := range l.items {
		delete(l.items, k)
	}
	l.evictList.Init()
}

```

## Get
Get is to get the item by its key. To take it better from the map and return, whether this element is in the vault or not.
```Go
// Get looks up a key's value from the cache
func (l *LRU) Get(key interface{}) (value interface{}, ok bool) {
	if ent, ok := l.items[key]; ok {
		l.evictList.MoveToFront(ent)
		return ent.Value.(*entry).value, true
	}
	return
}
```

## Contains
In order to know whether we have an element in the cache or not

```Go
// Contains check if key is in cache without updating
// recent-ness or deleting it for being state.
func (l *LRU) Contains(key interface{}) (ok bool) {
	_, ok = l.items[key]
	return ok
}
```
A test is for knowing that the method works.
```Go

// Test that Contains doesn't update recent-ness
func TestLRU_Contains(t *testing.T) {
	l, err := New(2)
	if err != nil {
		t.Fatalf("err: %v", err)
	}

	l.Add(1, 1)
	l.Add(2, 2)
	if !l.Contains(1) {
		t.Errorf("1 should be contained")
	}

	l.Add(3, 3)
	if l.Contains(1) {
		t.Errorf("Contains should not have updated recent-ness of 1")
	}
}
```
## Remove
The task of this method is to remove the element completely by the key, if it exists in our cache. In this case, we need to know whether the element has retired or not.
```Go
// Remove removes prodided key from the cache, returning if the
// key was contained
func (l *LRU) Remove(key interface{}) bool {
	if ent, ok := l.items[key]; ok {
		l.removeElement(ent)
		return true
	}
	return false
}
```

## GetOldest и RemoveOldest

These methods can be useful in order to get from the outside and/or delete the most "old" values in the cache
```Go
// RemoveOldest removes oldest item from cache
func (l *LRU) RemoveOldest() (interface{}, interface{}, bool) {
	ent := l.evictList.Back()
	if ent != nil {
		l.removeElement(ent)
		kv := ent.Value.(*entry)
		return kv.key, kv.value, true
	}
	return nil, nil, false
}

// GetOldest returns oldest item from cache
func (l *LRU) GetOldest() (interface{}, interface{}, bool) {
	ent := l.evictList.Back()
	if ent != nil {
		kv := ent.Value.(*entry)
		return kv.key, kv.value, true
	}
	return nil, nil, false
}
```

Plus to this test
```Go
func TestLRU_GetOldest_RemoveOldest(t *testing.T) {
	l, err := New(128)
	if err != nil {
		t.Fatalf("err: %v", err)
	}
	for i := 0; i < 256; i++ {
		l.Add(i, i)
	}
	k, _, ok := l.GetOldest()
	if !ok {
		t.Fatalf("missing")
	}
	if k.(int) != 128 {
		t.Fatalf("bad: %v", k)
	}

	k, _, ok = l.RemoveOldest()
	if !ok {
		t.Fatalf("missing")
	}
	if k.(int) != 128 {
		t.Fatalf("bad: %v", k)
	}

	k, _, ok = l.RemoveOldest()
	if !ok {
		t.Fatalf("missing")
	}
	if k.(int) != 129 {
		t.Fatalf("bad: %v", k)
	}
}
```

## Keys

This method is needed in order to get all the keys in our cache, for example, to get the value from the cache later.
```Go
// Keys returns a slice of the keys in the cache
func (l *LRU) Keys() []interface{} {
	keys := make([]interface{}, len(l.items))
	i := 0
	for ent := l.evictList.Back(); ent != nil; ent = ent.Prev() {
		keys[i] = ent.Value.(*entry).key
		i++
	}
	return keys
}
```

## That's all. 
We finished with the cache build. We will write a test to run out all possible scenarios and make sure that the code works.

```Go
func TestLRU(t *testing.T) {
	l, err := New(128)
	assert.NoError(t, err)

	for i := 0; i < 256; i++ {
		l.Add(i, i)
	}
	assert.Equal(t, 128, l.Len())

	for i, k := range l.Keys() {
		if v, ok := l.Get(k); !ok || v != k || v != i+128 {
			t.Fatalf("bad key: %v", k)
		}
	}
	for i := 0; i < 128; i++ {
		_, ok := l.Get(i)
		assert.False(t, ok)
	}
	for i := 128; i < 256; i++ {
		_, ok := l.Get(i)
		assert.True(t, ok)
	}
	for i := 128; i < 192; i++ {
		ok := l.Remove(i)
		assert.True(t, ok)
		ok = l.Remove(i)
		assert.False(t, ok)
		_, ok = l.Get(i)
		assert.False(t, ok)
	}

	l.Get(192) // expect 192 to be last key in l.Keys()

	for i, k := range l.Keys() {
		if (i < 63 && k != i+193) || (i == 63 && k != 192) {
			t.Fatalf("out of order key: %v", k)
		}
	}

	l.Purge()
	if l.Len() != 0 {
		t.Fatalf("bad len: %v", l.Len())
	}
	if _, ok := l.Get(200); ok {
		t.Fatalf("should contain nothing")
	}
}
```

## Congratulations!
You have implemented LRU cache and you are sure that it works. But there is a problem with the fact that it is not consistent. Data can be duplicated in it. In the [next](../step12/README.md) part we will make a consistent storage
