---
layout: post
title: "Deep Dive: How Google's TBR Matched Markets Algorithm Searches for Treatments and Controls"
subtitle: "searching strategies explained"
date: 2026-01-02 23:09:12
header-style: text
catalog: true
author: "Yuan"
tags: [TBR, TBRMM, Gready Search, Hill Climb, Priority Queue]
---
{% include linksref.html %}
In Causal Impact analysis (specifically Time-Based Regression or TBR), the hardest part isn't running the regression—it's designing the experiment. Finding the perfect combination of Treatment locations and Control locations is a massive combinatorial optimization problem.

In this post, I will break down the engineering behind how the TBR Matched Markets library automates this search to find the most statistically robust experimental design.

# 0. Data Preparation: Cleaning & Feasibility
- keep only the most recent n_pretest_max data points for the experiments.
- calculate the impact for each geos. ()
```python
def estimate_required_impact(y):
      return TBRMMDiagnostics(y,
                              parameters).estimate_required_impact(
                                  parameters.rho_max)
    # Consider only the most recent n_pretest_max time points
    data.df = data.df.iloc[:, -parameters.n_pretest_max:]
    # Calculate the required impact estimates for each geo.
    geo_req_impact = data.df.apply(estimate_required_impact, axis=1)

    self.geo_req_impact = geo_req_impact
    self.data = data
    self.parameters = parameters

```

# 1. The Scoring Function: "Safety First"

How does the algorithm decide if Design A is better than Design B? It uses a Lexicographical Scoring system embodied in the TBRMMScore class.

Instead of a single number, the score is a tuple. Python compares tuples element-by-element, meaning the first element is the most important "gatekeeper."

```python
Scoring(
    corr_test,  # 1st Priority: Did it pass the correlation test? (0 or 1)
    aa_test,    # 2nd Priority: Did it pass the A/A test? (0 or 1)
    bb_test,    # 3rd Priority: Did it pass the Brownian Bridge test? (0 or 1)
    dw_test,    # 4th Priority: Did it pass the Durbin-Watson test? (0 or 1)
    corr,       # 5th Priority: How high is the correlation? (0.0 to 1.0)
    inv_imp     # 6th Priority: How sensitive is the test? (Higher is better)
)
```

The Logic: A design with a 0.99 correlation that fails the A/A Test (Priority 2) will always lose to a design with 0.80 correlation that passes all diagnostic tests. This ensures we never trade statistical validity for a pretty chart.

# 2. The Data Structure: A Fixed-Size Priority Queue
Searching generates thousands of potential designs. We cannot store them all. We need a "Leaderboard" that only keeps the top $N$ best results.

The algorithm uses a custom HeapDict.
- **Key**: Usually the group size or category.
- **Value**: A **Min-Heap** of fixed size.

When a new design is found, we push it onto the heap. If the heap is full, the worst design (the smallest in the min-heap) is instantly discarded. This keeps memory usage constant regardless of how long the search runs.

```python
class HeapDict:
  """A dictionary of priority queues of a given limited size."""

  def __init__(self, size: int):
    self._size = size
    self._result = collections.defaultdict(list)

  def push(self, key: DictKey, item: Any):
    """
    Push item. If the queue exceeds size, the worst item (smallest)
    is automatically popped/discarded.
    """
    queue = self._result[key]
    if len(queue) < self._size:
      heapq.heappush(queue, item)
    else:
      # Efficiently push new item and pop smallest item in one operation
      heapq.heappushpop(queue, item)
    self._result[key] = queue
```

# 3. The Search Algorithms

## 3.1. Greedy Search (Hill Climbing)
This method is fast and effective. It uses a "Stepwise" approach, alternating between optimizing the control group and expanding the treatment group.
- Phase A: Optimize Control (Matching)

    - Goal: For a fixed Treatment group, find the best possible Control group.

    - Action: Iterate through available geographies. Use symmetric_difference to flip a geo's status (add it if it's out, remove it if it's in).

    - Stop Condition: Keep swapping until no single change improves the TBRMMScore (a local maximum).

- Phase B: Optimize Treatment (Growth)

    - Goal: Expand the experiment size.

    - Action: Add one new geography to the Treatment group that results in the best starting score.

    - Loop: Once a new geo is added, go back to **Phase A** to re-optimize the control group for this new, larger treatment group.

## 3.2. Exhaustive Search (Pruning)
This method attempts to find the global best solution by systematically checking valid combinations. However, a true brute-force search is impossible. To solve this, it uses Pruning ("Fail Fast" logic).

- The "Fail Fast" Logic: It relies on the monotonicity of constraints (Budget and Volume).

    - If {New York} is too expensive, then {New York, Boston} is definitely too expensive.

- Implementation:

    1. The algorithm remembers "bad" patterns (small groups that failed).

    2. Before checking a larger group, it runs skip_if_subset().

    3. If the new group contains a known "bad" pattern, it skips the expensive regression calculation entirely.

This allows the exhaustive search to explore millions of combinations while only calculating statistics for the viable ones.

---
