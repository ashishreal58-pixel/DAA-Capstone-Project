# Intelligent Courier Planning System

A Python prototype that plans a courier trip end to end: it orders parcels, chooses the best set to load, and finds a short delivery route over a road network.

**Course:** Design Thinking and Algorithm – Capstone
**Author:** Shreyansh Kumar (Roll No. 2501010019)

---

## Overview

Given a list of parcels and a road network, the system:

1. **Sorts** parcels by deadline, then priority, using **merge sort**.
2. **Selects** the most valuable set of parcels that fits the vehicle (40 kg) using **0/1 knapsack dynamic programming**.
3. **Computes shortest paths** between locations using **Dijkstra's algorithm** (min-heap).
4. **Orders the delivery stops** using a **nearest-neighbour TSP approximation**, starting and ending at the Depot.

Runtime of every major step is benchmarked on datasets from 50 to 800 items.

## Pipeline

```
parcels.csv --> merge sort (deadline, priority) --> knapsack DP (capacity 40 kg)
                                                          |
                                                destinations of chosen parcels
                                                          |
roads.csv --> graph --> Dijkstra distances --> nearest-neighbour TSP --> route + km
```

## Algorithms Used

| Sub-problem | Algorithm | Why | Complexity |
|---|---|---|---|
| Parcel ordering | Merge sort | Stable, guaranteed O(n log n); insertion sort used as baseline | O(n log n) |
| Choose parcels for a trip | 0/1 Knapsack (DP) | Optimal value under the weight limit; greedy can be wrong | O(n · W) |
| Distances between points | Dijkstra (min-heap) | All road lengths are non-negative | O((V+E) log V) |
| Order of stops | Nearest-neighbour TSP | Exact TSP is NP-hard; fast approximation | O(k (V+E) log V) |

## Project Structure

```
.
├── main.py        # Integrated pipeline (sort → knapsack → route)
├── sorting.py     # Merge sort and insertion sort
├── knapsack.py    # 0/1 knapsack DP with backtracking
├── graph.py       # Graph loader, Dijkstra, nearest-neighbour TSP
├── utils.py       # CSV loading and random data generators
├── benchmark.py   # Runtime tests and chart
├── tests.py       # Unit tests
├── data/          # Sample datasets (parcels.csv, roads.csv)
└── output/        # Run output, benchmark results, charts
```

## Requirements

- Python 3.8+
- `matplotlib` (only needed for `benchmark.py`)

```bash
pip install matplotlib
```

## Usage

Run all commands from the project root, since data paths are relative.

```bash
python main.py --capacity 40   # full demonstration
python tests.py                # unit tests
python benchmark.py            # runtime table + chart (saves output/runtime_chart.png)
```

> `benchmark.py` saves its chart into `output/`, so make sure that folder exists before running it.

`--capacity` sets the vehicle capacity in kg (default: 40).

## Input Format

**`data/parcels.csv`**

```csv
id,dest,weight,priority,deadline
P01,Market,8,6,4
P02,Campus,12,9,2
...
```

| Field | Meaning |
|---|---|
| `id` | Parcel identifier |
| `dest` | Destination (a node in the road graph) |
| `weight` | Weight in kg (integer) |
| `priority` | 1–10, higher = more important |
| `deadline` | Hours until due |

**`data/roads.csv`** – undirected weighted edge list, loaded into an adjacency dictionary.

```csv
from,to,km
Depot,Market,4
Depot,Campus,6
...
```

The Depot is both the start and end of every route.

## Sample Output

```
=== PACKING PLAN (capacity 40 kg) ===
P03 -> Hospital (5 kg, prio 10)
P02 -> Campus (12 kg, prio 9)
P01 -> Market (8 kg, prio 6)
P05 -> Library (7 kg, prio 7)
P08 -> Market (6 kg, prio 3)
Total weight: 38 kg | Total priority value: 35
Left for next trip: P09, P06, P04, P07, P10

=== DELIVERY ROUTE ===
Stop order : Depot -> Market -> Campus -> Library -> Hospital -> Depot
Full path  : Depot -> Market -> Campus -> Library -> Mall -> Hospital -> Depot
Route length: 29.0 km
```

The shortest road between Library and Hospital passes through Mall, so Mall appears in the full path even though no parcel is delivered there.

## Performance Results

Timed with `time.perf_counter()` on seeded random data (milliseconds):

| n | Merge sort | Insertion sort | Knapsack DP | Dijkstra | NN-TSP route |
|---:|---:|---:|---:|---:|---:|
| 50 | 0.12 | 0.15 | 0.64 | 0.10 | 1.74 |
| 100 | 0.26 | 0.73 | 1.19 | 0.19 | 3.64 |
| 200 | 0.65 | 2.83 | 2.60 | 0.54 | 8.28 |
| 400 | 1.41 | 7.27 | 3.00 | 0.46 | 10.95 |
| 800 | 1.96 | 28.48 | 6.71 | 1.10 | 23.30 |

Takeaways:

- **Sorting:** at n = 800, merge sort takes about 2 ms vs. about 28 ms for insertion sort, matching O(n log n) vs. O(n²).
- **Knapsack:** grows roughly linearly with n for a fixed capacity.
- **NN-TSP route:** the most expensive step, because Dijkstra is run from every stop.
- **Single Dijkstra run:** very cheap, about 1 ms for 800 nodes.

## Tests

| Test | Checks |
|---|---|
| `test_sorts_agree` | Merge sort and insertion sort give identical order on 60 random parcels |
| `test_knapsack` | Known instance (capacity 7) returns optimal value 9 |
| `test_dijkstra` | Shortest A → D in a 4-node graph is 6 |
| `test_route` | NN route A-B-D-A has the expected order and length 12 |

## Limitations

- Nearest-neighbour is a heuristic and not guaranteed to be optimal.
- Single vehicle and a single trip.
- Parcel weights must be integers.

## Future Improvements

1. Improve the nearest-neighbour tour with 2-opt swaps.
2. Cache Dijkstra results per stop so each is computed only once.
3. Use a 1-D knapsack array to reduce memory from O(nW) to O(W).
4. Support multiple trips/vehicles and delivery time windows.
5. Use A* search for very large road networks.

## Conclusion

Exact algorithms are used where they are cheap (sort, knapsack, Dijkstra), and an approximation is used where the exact problem is NP-hard (TSP). Together they form a working courier planner.
