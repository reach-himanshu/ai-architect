# DSA: Heaps & Priority Queues

A Heap is a specialized tree-based structure that satisfies the heap property. Python's `heapq` module implements a **Min-Heap**.

## 1. Heap Property
- **Min-Heap**: Parent ≤ Children. Root is the minimum.
- **Max-Heap**: Parent ≥ Children. Root is the maximum.

## 2. Operations & Complexity
| Operation | Complexity |
|:---|:---:|
| Insert | $O(log n)$ |
| Get Min | $O(1)$ |
| Extract Min | $O(log n)$ |
| Heapify | $O(n)$ |

## 3. Python's `heapq` Module

```python
import heapq

nums = [5, 3, 8, 1, 2]
heapq.heapify(nums)  # In-place transform to heap

heapq.heappush(nums, 0)  # Add element
smallest = heapq.heappop(nums)  # Remove and return smallest
```

## 4. Max-Heap Trick
`heapq` is a min-heap. For max-heap, negate values.

## 5. Use Cases
- **Priority Queues**: Task scheduling by priority.
- **Dijkstra's Algorithm**: Shortest path.
- **Top-K Problems**: Finding K largest/smallest elements.
