# Guava Notes

## Immutable Structures
### ImmutableCollection
It's a bad idea to make any change on elements that are already in a collection. 

## BiMap
A data structure allowing viewing a value-to-key map, with ```inverse()```. It means, no more than one entry would share the same value. 
### high level design
Maintains two array of linked list. Acts like 2 hashmaps: one for key-value, one for value-key.
### BiEntry
Looks like a node in linkedlist. Located in both key-value and value-key linkedlist. Totally private internal class.
### ContainsKey / ContainsValue - boolean
Both the hash (hash & mask) and the value should match.
Find the hash of key, then insert into the head of the bucket. The same as the value.
### put / forcePut
Both call the put with a boolean param indicating if it's forcible. 
- If there is already an entry with the key and the value, return the value. 
- If the value is already existed, throw an error if unforcible. 
-- Else, delete the key-value pair with this value.
- If the key is existed, delete the key-value pair with this key.
- Insert this new key-value pair.

```
in the map, there are:
[ horizon -> ps4 ; csgo -> pc ]
put(horizon, pc) => value existed, error
forcePut(horizon, pc) => delete (csgo -> pc pair), delete (horizon -> ps4 pair), then insert (horizon -> pc).
```
### inverse()
An Inverse class to do thing in the other direction. like inverse().get(key), inside is just seekByValue().
* @SuppressWarnings: to suppress specified kind of warnings.
```
@SuppressWarnings({"unchecked"}) // allow raw type
@SuppressWarnings("deprecated") // allow deprecated methods or classes
```

### ImmutableMap
An interface. The keys, values, entries are all uneditable. Nothing fancy, just make all edit-related functions throw an UnSupportedOperationWarning(). The main function is to create such a map.
```
copyOf() // copy all the entries from an existed map
of() // an empty map. used for what?
of(k1,v1 .... k5, v5) // five methods to create a map of length 1 - 5;
```
An internal class called Builder gives you a flexible way to create an immutablemap. Builder serves like a temprate map. put(), putAll(Map map) are provided to let you create a "normal" map just like "normal" map. Then, Builder.build() will return a immutable map. Then, this new map would be immutable. Nothing would change inside.

### MultiMap
Map<K, Collection<V>> map;
index: 
```
Multipamap.index(Iterable, Function): ImmutableListMultimap<K, V>
```
```
Function<INPUTTYPE, OUTPUTTYPE> func = new Function<INPUTTYPE, OUTPUTTYPE>() {
    public OUTPUTTYPE apply(INPUTTYPE input) {return ...}
};
```
What it does: for every item in the Iterable object, calculate its index. Then, use the index as the key, the object as the value, then store this key-value pair into the multimap. Therefore, each key will lead a set of values.

### ForwardingMap


## EventBus
A hub that registers all listeners with their related methods. When post some events, find all listeners supposed to receive this post and call their stored methods, in a MultiMap.

## Cache
Cache provides a Map-like structure to temporarily store expensive objects. Objects stored in cache can be evicted by different methods or different time. 

### enum singleton
```
enum Singleton {
    INSTANCE;
    private Singleton() {}
}
```
equals to defining a singleton like
```
class Singleton() {
    public Singleton() {};
}
Singleton singleton = new Singleton();
```
Also, enum can implement interfaces, but no extends. In enum, each item will invoke the constructor.
Don't get confused, INSTANCE is just the name of one item in this enum.

### LocalCache
Seems the factory method is vital to util classes. For LocalCache, the public "constructor" would be `CacheBuilder.builder()`.
`StatsCounter` is a class to make inner attribute in Caches to make the stats of caches. It counts hits and misses, and total, then it can canculate the rate of hittings.

## Graph
Three interfaces increased by complexity: Graph, ValueGraph, Network.
Builder, of course, again.
- Builder
A method to insert configurations into the constructor. Each configuration method would return a type of itself, then Builder().config1().config2()..... would still return a object of that type. The real (maybe private) constructor must have several polymorphisms with different numbers of parameters, or just a "containing everything" constructor. Then, however many parameters are set by config methods, they will be used in the constructors. The rest of parameters will stay the defaul values.

### Graph
Base graph. Node can be any type. Edge is made by `EndpointPair`.

### ValueGraph
It's weighted graph...

### Network
Edges must be unique. Describe a topologic among all the nodes. No parallel edges.

### Traverser