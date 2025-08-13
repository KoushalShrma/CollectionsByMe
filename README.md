# Java Collections Framework – DSA Quick Reference (PDF-Ready)

Instructions (optional):
1. Save this file as `java-collections-dsa-reference.md`.
2. Convert to PDF:
   - Using Pandoc:  
     `pandoc java-collections-dsa-reference.md -o Java_Collections_DSA_Reference.pdf`
   - VSCode: Open the Markdown, use a Markdown PDF extension.
   - IntelliJ: Right-click → "Save to PDF" (with proper plugin).
3. Print in landscape for best table fit.

Legend  
O(1)* = Amortized average; logN = O(log n); n = size  
Ordering: Insertion Order = order of insertion; Natural Order = Comparable/Comparator sorting.  

---

## 1. Core Interface Overview

| Interface | Guarantees / Nature | Allows Duplicates | Ordering | Typical DSA Uses | Pick When |
|-----------|--------------------|-------------------|----------|------------------|-----------|
| List      | Indexed sequence   | Yes               | Insertion | Arrays w/ growth, DP tables, windows, adjacency lists | Need index random access or sequential storage |
| Set       | Uniqueness         | No                | Depends (Hash: none, Linked: insertion, Tree: sorted) | Visited sets, uniqueness, range queries (TreeSet) | Need fast membership / uniqueness |
| Queue     | FIFO abstraction   | Yes               | Processing order | BFS frontier, level-order, buffer | Process in arrival order |
| Deque     | Double-ended       | Yes               | Ends access order | Sliding windows, monotonic queues, stack emulation | Need both stack & queue ops |
| Map       | Key→Value          | Keys unique       | Depends (Hash: none, Linked: insertion/access, Tree: sorted) | Frequency counting, parent tracking, memo, adjacency map | Need associative lookup |
| Specialized | Enum, Bit, Priority | Depends | Specialized | Priority selection, enum flags, dense boolean sets | Domain-specific optimization |

---

## 1.1 List Details

| Implementation | Internal | Key Strengths | Weaknesses | Complexity (Core) | Best For |
|----------------|----------|---------------|------------|-------------------|----------|
| ArrayList      | Dynamic array | Fast get/set; cache friendly | Slow mid insert/remove O(n) | get/set O(1), add(end) amort O(1), insert/remove mid O(n) | Random access, heavy reads |
| LinkedList     | Doubly linked | Fast add/remove at ends/iterator | O(n) random access; GC overhead | add/remove at ends O(1), get O(n) | Frequent iterator removals |
| Arrays.asList  | Fixed-size view | Quick wrap | No structural add/remove | Same as underlying array | Lightweight view (rare in contests) |

Common Methods (DSA Focus):
```
add(E), add(i,E), addAll(coll)
get(i), set(i,val)
remove(i) / remove(Object)
contains(x), indexOf(x)
size(), isEmpty()
subList(a,b), listIterator()
```

Conversions (List):
```
Array -> List: List<T> list = new ArrayList<>(Arrays.asList(arr));
List -> Array: T[] a = list.toArray(new T[0]);
List -> Set: Set<T> s = new HashSet<>(list);
List -> Deque: Deque<T> dq = new ArrayDeque<>(list);
Numeric List -> int[]: int[] a = list.stream().mapToInt(Integer::intValue).toArray();
```

Decision Tips (List):
- Random access heavy → ArrayList
- Many iterator-based deletions → LinkedList
- Stack/Deque behavior → Prefer ArrayDeque (not LinkedList)

---

## 1.2 Set Details

| Implementation | Ordering | Null? | Core Strength | Special Ops | Complexity | Use When |
|----------------|----------|-------|---------------|-------------|------------|----------|
| HashSet        | None     | Yes (1) | Fast membership | – | add/contains/remove O(1) avg | Visited tracking |
| LinkedHashSet  | Insertion | Yes (1) | Deterministic iteration | – | ~HashSet + overhead | Need stable order |
| TreeSet        | Sorted (Natural/Comparator) | No (elements) | Range / nearest queries | floor, ceiling, etc. | O(log n) | Ordered queries |
| EnumSet        | Enum ordinal bit vector | No null | Compact + fastest | bulk ops | O(1) basic | Enum flags set |
| BitSet         | Bit array | N/A | Dense int domain | nextSetBit | O(n/64) per scan | Large boolean sets |

TreeSet Range Examples:
```
set.ceiling(x); set.floor(x);
set.higher(x); set.lower(x);
set.subSet(a, b); set.headSet(x); set.tailSet(x);
```

Conversions (Set):
```
Any Collection -> HashSet: new HashSet<>(c);
HashSet -> TreeSet: new TreeSet<>(hashSet);
TreeSet -> LinkedHashSet (freeze sorted order): new LinkedHashSet<>(treeSet);
Set -> List: new ArrayList<>(set);
```

Decision Tips (Set):
- Uniqueness + speed → HashSet
- Preserve insertion order → LinkedHashSet
- Need nearest / range → TreeSet
- Enum domain → EnumSet
- Dense small int domain → BitSet / boolean[]

---

## 1.3 Queue & Deque

| Implementation | Type | Null Allowed? | Strengths | Weaknesses | Complexity | Use Cases |
|----------------|------|---------------|-----------|------------|------------|-----------|
| ArrayDeque     | Deque | No | Fast ends, stack & queue, low overhead | No random index | add/remove ends O(1) | BFS, sliding window |
| LinkedList     | Deque | Yes | Supports iterator removals mid-list | More GC, slower | Ends O(1) | Rare specialized |
| PriorityQueue  | Heap (min by default) | No | Fast min (or custom) extraction | No order iteration | offer/poll O(log n), peek O(1) | Top-K, scheduling |
| ArrayBlockingQueue | Bounded FIFO | No | Thread-safe | Concurrency overhead | O(1) | Multi-thread (rare DSA) |

Queue / Deque Methods:
```
offer(e), add(e)
poll(), remove()
peek(), element()
addFirst/Last, offerFirst/Last
pollFirst/Last, peekFirst/Last
push(e), pop() // stack
```

PriorityQueue:
```
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a->a[1]));
pq.offer(x); pq.peek(); pq.poll();
```

Conversions:
```
Collection -> ArrayDeque: Deque<T> dq = new ArrayDeque<>(c);
Queue -> PriorityQueue: PriorityQueue<T> pq = new PriorityQueue<>(q);
PriorityQueue -> Sorted List:
List<T> sorted = new ArrayList<>();
PriorityQueue<T> copy = new PriorityQueue<>(pq);
while(!copy.isEmpty()) sorted.add(copy.poll());
```

Decision Tips:
- Need FIFO → ArrayDeque
- Need stack → ArrayDeque (push/pop)
- Need min/max extraction → PriorityQueue
- Monotonic window → ArrayDeque + custom logic

---

## 1.4 Map

| Implementation | Ordering | Null Key? | Strength | Complexity | Use When |
|----------------|----------|-----------|----------|------------|----------|
| HashMap        | None     | Yes (1) | Fast lookup | avg O(1) | General purpose |
| LinkedHashMap  | Insertion / Access (if accessOrder=true) | Yes (1) | Predictable order, LRU ready | ~HashMap + overhead | LRU, ordered output |
| TreeMap        | Sorted by key | No (unless comparator handles) | Range / nearest key | O(log n) | Ordered queries |
| EnumMap        | Enum ordinal array | No | Fastest enum key map | O(1) | Enum keyed frequency |
| ConcurrentHashMap | Segmented/Bucketed | Yes (1) | Thread-safe | O(1) avg | Concurrency only |

Core Operations:
```
put(k,v), putIfAbsent(k,v)
get(k), getOrDefault(k, def)
containsKey(k)
remove(k), remove(k,v)
merge(k,val,(oldv,newv)->...)
computeIfAbsent(k, k2 -> new ArrayList<>())
keySet(), values(), entrySet()
```

TreeMap Extras:
```
ceilingKey(k), floorKey(k), higherKey(k), lowerKey(k)
subMap(a,b), headMap(k), tailMap(k)
firstKey(), lastKey()
```

Conversions:
```
Map -> Keys: List<K> keys = new ArrayList<>(map.keySet());
Map -> Values: List<V> vals = new ArrayList<>(map.values());
Map -> Entries: List<Map.Entry<K,V>> es = new ArrayList<>(map.entrySet());
HashMap -> TreeMap: new TreeMap<>(hashMap);
HashMap -> LinkedHashMap (snapshot order of iteration): new LinkedHashMap<>(hashMap);
Parallel arrays -> Map: for (int i=0;i<keys.length;i++) map.put(keys[i], vals[i]);
```

Decision Tips:
- Simple freq or memo → HashMap
- Need LRU / deterministic output → LinkedHashMap
- Need nearest key / ranges → TreeMap
- Enum keys → EnumMap

---

## 1.5 Specialized & Utility

| Type | Purpose | Note |
|------|---------|------|
| Stack (legacy) | Stack (LIFO) | Use Deque instead |
| Vector / Hashtable | Legacy synchronized | Avoid in DSA |
| BitSet | Dense integer flag set | Memory efficient |
| EnumSet / EnumMap | Enum domains | Very fast |
| Collections / Arrays | Utility static methods | sort, binarySearch, fill, etc. |
| Streams | Declarative ops | Avoid in hot loops (perf) |

---

## 2. Time Complexity Snapshot

| Structure | Access | Search (by value) | Insert | Remove | Notes |
|-----------|--------|-------------------|--------|--------|-------|
| ArrayList | O(1) get | O(n) | Amort O(1) end | O(n) mid | Resize doubling |
| LinkedList | O(n) | O(n) | O(1) ends | O(1) ends | Node overhead |
| HashSet / HashMap | – | O(1) key | O(1) avg | O(1) avg | Worst O(n) rare |
| LinkedHashSet / Map | – | O(1) key | O(1) avg | O(1) avg | Order maintenance |
| TreeSet / TreeMap | – | O(log n) | O(log n) | O(log n) | Balanced tree |
| PriorityQueue | – | O(n) arbitrary | O(log n) | O(log n) (root) | Heap property |
| ArrayDeque | Ends only O(1) | O(n) scan | O(1) ends | O(1) ends | No index random |
| BitSet | O(1) bit test | O(n/64) scan | O(1) set | O(1) clear | Dense integer |
| EnumSet / EnumMap | O(1) | O(1) | O(1) | O(1) | Underlying bit/array |

---

## 3. Pattern → Collection

| Goal | Recommended Structure(s) | Reason |
|------|--------------------------|--------|
| Fast membership test | HashSet / boolean[] / BitSet | O(1) / memory efficiency |
| Maintain unique insertion order | LinkedHashSet | Order + uniqueness |
| Next greater/smaller key | TreeMap / TreeSet | Navigable ops |
| k-th smallest dynamic | PriorityQueue | Heap property |
| Median stream | Two Heaps (max + min PQ) | Balance halves |
| LRU Cache | LinkedHashMap(accessOrder=true) | Automatic reordering |
| Multi-map (k->list) | Map<K,List<V>> + computeIfAbsent | Simple & flexible |
| Frequency map | HashMap + merge/getOrDefault | Counting |
| Sliding window distinct | HashMap/HashSet | Track counts |
| Adjacency list graph | List<List<Integer>> or Map<Node,List<Node>> | Space vs index flexibility |
| Top-K frequent | HashMap + min-PQ | Controlled heap size |
| Order statistics beyond min/max | TreeSet/TreeMap | LogN navigation |

---

## 4. Idiomatic Code Snippets

Counting Frequencies:
```java
map.merge(x, 1, Integer::sum);
// or
map.put(x, map.getOrDefault(x, 0) + 1);
```

Building Adjacency:
```java
graph.computeIfAbsent(u, k -> new ArrayList<>()).add(v);
```

PriorityQueue Custom Comparator:
```java
PriorityQueue<int[]> pq =
  new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
```

Safe Remove During Iteration:
```java
for (Iterator<Integer> it = set.iterator(); it.hasNext();) {
  int v = it.next();
  if (cond(v)) it.remove();
}
```

Monotonic Deque (Max Window):
```java
while (!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast();
dq.offerLast(i);
if (dq.peekFirst() <= i - k) dq.pollFirst();
```

TreeMap Ceiling Entry:
```java
Map.Entry<Integer,Integer> e = tree.ceilingEntry(x);
```

---

## 5. General Conversion Patterns

| From | To | Snippet |
|------|----|---------|
| Any Collection | ArrayList | `new ArrayList<>(c)` |
| Any Collection | LinkedList | `new LinkedList<>(c)` |
| Any Collection | HashSet | `new HashSet<>(c)` |
| Ordered Collection | LinkedHashSet | `new LinkedHashSet<>(c)` |
| Comparable Elements Collection | TreeSet | `new TreeSet<>(c)` |
| Comparable Elements Collection | PriorityQueue | `new PriorityQueue<>(c)` |
| int[] | List<Integer> | `List<Integer> list = Arrays.stream(arr).boxed().toList();` |
| List<T> | T[] | `T[] a = list.toArray(new T[0]);` |
| Set<T> | List<T> | `new ArrayList<>(set)` |
| Map | Keys list | `new ArrayList<>(map.keySet())` |
| Map | Values list | `new ArrayList<>(map.values())` |
| Map | Entry list | `new ArrayList<>(map.entrySet())` |
| List<Pair<K,V>> | Map | `list.stream().collect(Collectors.toMap(p->p.k,p->p.v))` |
| Map | TreeMap | `new TreeMap<>(map)` |
| List | PriorityQueue | `new PriorityQueue<>(list)` |
| PriorityQueue | Sorted List | Poll all into list |
| Deque | Stack semantics | `push(), pop()` |
| Stack (legacy) | Deque | `new ArrayDeque<>(stack)` |
| BitSet | List<Integer> | iterate with `nextSetBit` |
| List<Integer>(0/1) | BitSet | set bits where value=1 |

---

## 6. Quick Decision Matrix

| Question / Need | Choose |
|-----------------|--------|
| Duplicates + order + fast read | ArrayList |
| Frequent end insert/delete + stack/queue | ArrayDeque |
| Unique unordered fast membership | HashSet |
| Unique with insertion order | LinkedHashSet |
| Unique sorted / range queries | TreeSet |
| Key→Value general | HashMap |
| Key→Value sorted/range | TreeMap |
| Key→Value insertion/access order | LinkedHashMap |
| Repeated min/max extraction | PriorityQueue |
| LRU behavior | LinkedHashMap(accessOrder=true) |
| Many enum keys | EnumSet / EnumMap |
| Dense int membership | BitSet / boolean[] |

---

## 7. Null & Threading Notes

| Structure | Null Support | Thread Safety Note |
|-----------|--------------|--------------------|
| HashMap / HashSet / LinkedHash* | One null key; many null values | Not thread-safe |
| TreeMap / TreeSet | Null elements disallowed (keys) | Not thread-safe |
| ArrayDeque / PriorityQueue | No null elements | Not thread-safe |
| EnumMap / EnumSet | No null | Not thread-safe |
| Concurrent* variants | Usually restrict null | Thread-safe (not needed for single-thread DSA) |

---

## 8. Memory Considerations

| Structure | Memory Notes |
|-----------|--------------|
| ArrayList | Over-allocated capacity (growth factor ~1.5) |
| LinkedList | Node object + 2 refs each (prev/next) |
| HashMap/Set | Buckets + Node/TreeNode objects, loadFactor (default .75) |
| LinkedHashMap/Set | + doubly linked list pointers |
| TreeMap/TreeSet | Node (key, value/ref, 2 children, color) |
| EnumMap/EnumSet | Compact array/bit vector |
| BitSet | long[] backing (64 bits per element batch) |
| PriorityQueue | Array backing (heap) |

Pre-sizing HashMap:
```java
int cap = (int)(expected / 0.75f) + 1;
Map<K,V> map = new HashMap<>(cap);
```

---

## 9. Common Pitfalls

| Pitfall | Avoidance |
|---------|-----------|
| Modifying collection in enhanced for | Use iterator.remove() |
| Arrays.asList on primitive array | Use streams to box or manual loop |
| Using PriorityQueue for sorted iteration directly | Poll or copy & sort |
| Confusing floor vs lower (TreeSet/Map) | floor = <= ; lower = < |
| Overusing LinkedList | Prefer ArrayList unless removal pattern demands |
| Streams in hot loops | Prefer for-loops |
| Null in TreeSet/TreeMap | Avoid (NPE / illegal) |

---

## 10. Mini Cheat Snippets

Two-Sum:
```java
Map<Integer,Integer> idx = new HashMap<>();
for (int i=0;i<nums.length;i++){
  int need = target - nums[i];
  if (idx.containsKey(need)) return new int[]{idx.get(need), i};
  idx.put(nums[i], i);
}
```

Top-K Frequent:
```java
Map<Integer,Integer> freq = new HashMap<>();
for (int x: nums) freq.merge(x,1,Integer::sum);
PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a->a[1]));
for (var e: freq.entrySet()){
  pq.offer(new int[]{e.getKey(), e.getValue()});
  if (pq.size() > k) pq.poll();
}
List<Integer> res = new ArrayList<>();
while(!pq.isEmpty()) res.add(pq.poll()[0]);
Collections.reverse(res);
```

Sliding Window Distinct:
```java
Map<Integer,Integer> count = new HashMap<>();
int left=0;
for (int right=0; right<n; right++){
  count.merge(a[right],1,Integer::sum);
  while (count.get(a[right]) > 1){
    count.merge(a[left], -1, Integer::sum);
    if (count.get(a[left]) == 0) count.remove(a[left]);
    left++;
  }
}
```

Monotonic Deque (Window Max):
```java
Deque<Integer> dq = new ArrayDeque<>();
for (int i=0;i<n;i++){
  while(!dq.isEmpty() && dq.peekFirst() <= i - k) dq.pollFirst();
  while(!dq.isEmpty() && nums[dq.peekLast()] <= nums[i]) dq.pollLast();
  dq.offerLast(i);
  if (i >= k-1) ans.add(nums[dq.peekFirst()]);
}
```

Binary Search (List):
```java
int lo=0, hi=list.size()-1;
while (lo <= hi){
  int mid = (lo + hi) >>> 1;
  int cmp = Integer.compare(list.get(mid), target);
  if (cmp < 0) lo = mid + 1;
  else if (cmp > 0) hi = mid - 1;
  else return mid;
}
```

---

## 11. When to Avoid Streams

| Situation | Reason |
|-----------|--------|
| Tight loops (≥1e6 ops) | Overhead vs plain loops |
| Complex stateful window | Imperative clearer |
| Need fine-grained short-circuit with mutation | Loops simpler |

---

## 12. Summary Memory of Core Choices (Flash)

| Need | Choice |
|------|--------|
| Random access sequence | ArrayList |
| Ends operations / stack / queue | ArrayDeque |
| Unique unordered | HashSet |
| Unique insertion order | LinkedHashSet |
| Unique sorted / range | TreeSet |
| Key→Value unordered | HashMap |
| Key→Value ordered (output) | LinkedHashMap |
| Key→Value sorted / range | TreeMap |
| Min/Max extraction | PriorityQueue |
| Dense boolean membership | BitSet / boolean[] |

Flow:
```
Need key->value? -> Map (ordering? range? choose appropriate)
Else need sequence? random access -> ArrayList; ends only -> ArrayDeque
Need uniqueness? -> Set (ordering? sorted?)
Need priority processing? -> PriorityQueue
```

---
