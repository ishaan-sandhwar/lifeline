<p align="center">
  <img src="docs/banner.svg" alt="LifeLine: smart city disaster response and evacuation simulator" width="100%">
</p>

<div align="center">

![C++17](https://img.shields.io/badge/C%2B%2B-17-00599C?style=for-the-badge&logo=cplusplus&logoColor=white)
![React](https://img.shields.io/badge/React-Vite-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![License](https://img.shields.io/badge/License-MIT-3fb950?style=for-the-badge)
[![Live demo](https://img.shields.io/badge/Live_demo-Render-46E3B7?style=for-the-badge&logo=render&logoColor=black)](https://lifeline-31iq.onrender.com)

**A DSA-driven full-stack project: a fictional city of 40 locations and 83 roads, modelled as a weighted graph, with a C++ engine that routes emergency vehicles, simulates disasters, computes evacuation capacity with max-flow, dispatches rescue units and analyses network resilience.**<br>
Built by a five-member team for the DSA course at Lovely Professional University.

[🌐 Live demo](https://lifeline-31iq.onrender.com) · [🎬 Feature tour](#-feature-tour) · [🧩 How it works](#-how-it-works) · [🚀 Quick start](#-quick-start) · [🧪 Testing](#-testing) · [📄 Report](docs/report_final.pdf) · [🎞️ Slides](docs/LifeLine_Presentation.pptx)

</div>

<p align="center">
  <img src="docs/stats.svg" alt="40 locations and 83 roads, 96 automated checks, Dijkstra settles 35 nodes against 10 for A*, 25,200 people per hour evacuated in a one-ring flood, more than 10 algorithms behind the REST API" width="100%">
</p>

<p align="center">
  <img src="docs/screenshots/ui_evacuation.png" alt="One-ring flood at River Bridge South: rerouted path, blocked roads and min-cut evacuation bottlenecks" width="100%"><br>
  <sub>A one-ring flood at River Bridge South: the engine reroutes around blocked roads and finds the evacuation bottlenecks.</sub>
</p>

> [!NOTE]
> The live demo runs on a free-tier Render instance, so the first load can take about a minute to cold-start.

## ✨ What makes it different

<table>
  <tr>
    <td width="50%" valign="top">
      <h4>🧱 Written from scratch</h4>
      Binary heaps, a djb2 hash map, a trie and Union-Find are hand-written instead of the STL's. No <code>std::priority_queue</code> or <code>std::unordered_map</code> in the algorithms; <code>std::vector</code> is plain storage.
    </td>
    <td width="50%" valign="top">
      <h4>🧭 10+ algorithms, one engine</h4>
      Dijkstra, A*, Bellman-Ford, Floyd-Warshall, Edmonds-Karp, BFS spread, Tarjan, Prim and Kruskal, knapsack DP and greedy dispatch, all behind the REST API.
    </td>
  </tr>
  <tr>
    <td valign="top">
      <h4>📦 One self-contained binary</h4>
      A zero-dependency C++17 backend serves the REST API and the built React app. The only third-party code is two vendored headers: cpp-httplib and nlohmann/json.
    </td>
    <td valign="top">
      <h4>✅ Checked against something</h4>
      96 automated checks across 5 suites, compared against hand-checked examples and an independent Python verifier (see <a href="#-testing">Testing</a>).
    </td>
  </tr>
</table>

## 🎬 Feature tour

### 🧭 Routing (Dijkstra vs A*)

Route **Riverside Colony → Sunrise Hospital** with algorithm **Compare**: both find the same 4.78 km optimal path, but Dijkstra settles **35 nodes** vs A*'s **10** — the haversine heuristic focuses the search toward the goal. (On a graph this small, the heuristic's per-node cost means A* is not faster in wall-clock time; the saving is in nodes settled.) Hover any road for its name and length.

<table>
  <tr>
    <td><img src="docs/screenshots/ui_route.png" alt="Dijkstra route from Riverside Colony to Sunrise Hospital"></td>
    <td><img src="docs/screenshots/ui_compare.png" alt="Dijkstra vs A* comparison panel"></td>
  </tr>
</table>

### 🌊 Disaster + evacuation

1. Set spread radius to **1 ring**, epicenter **River Bridge South**, click **Trigger disaster**. Roads inside the zone turn red/dashed with expanding wave rings.
2. The same route now goes 5.42 km over the *North* bridge instead of the 4.78 km south route — the engine reroutes around blocked roads automatically.
3. The **Evacuation Capacity** panel shows the max people/hour the road network can push to safe shelters (**25,200/hour** here, limited by 9 min-cut roads), plus the exact **min-cut bottleneck roads**.
4. Bump radius to **2 rings**: the flood cuts both southern approaches and the west→east route becomes impossible — the map shows why.
5. **Clear** → everything reopens, the shortest route returns.

<p align="center">
  <img src="docs/screenshots/ui_disaster.png" alt="Disaster zone with rerouted path and evacuation capacity panel" width="100%">
</p>

### 🚑 Dispatch + supplies

1. With the flood active, click **🚑 Dispatch units**. Amber dashed lines show each vehicle's actual drive path; badges show severity (epicenter zone = highest).
2. Most severe incidents get vehicles **first** (triage max-heap), and each gets its **nearest** free compatible vehicle by road distance (greedy + Dijkstra).
3. Any ⚠ unreachable incidents are ones whose roads the flood cut — this ties directly back to the min-cut analysis.
4. **📦 Load truck**: 50 kg capacity; the knapsack DP picks the highest-relief-value subset (not the greedy value/weight order!).
5. The **Activity Log** is persisted — restart the server and it's still there (SQLite or JSONL).

<table>
  <tr>
    <td><img src="docs/screenshots/ui_dispatch.png" alt="Dispatched units with amber drive paths"></td>
    <td><img src="docs/screenshots/ui_supply.png" alt="Knapsack supply loading for a 50 kg truck"></td>
  </tr>
</table>

### 🕸️ Resilience + search

1. Type `riv` in the search box — trie prefix search suggests all four River locations instantly; click one to fly there.
2. **🕸 Analyze network** on the healthy city: *0 bridges, 0 articulation points* — the city was deliberately designed with no single point of failure (minimum degree 3, 2-edge-connected mesh).
3. Trigger the flood (2 rings) and analyze again: **6 critical roads + 7 critical junctions** appear in violet, and River Bridge South is isolated with no hospital reachable. Same Tarjan DFS, different road state.
4. **🌿 Restoration plan**: the 34.17 km green backbone — with the proof line "Prim = Kruskal, two algorithms, one optimum."

<table>
  <tr>
    <td><img src="docs/screenshots/ui_resilience.png" alt="Critical roads and junctions highlighted in violet"></td>
    <td><img src="docs/screenshots/ui_mst.png" alt="Restoration plan: minimum spanning backbone in green"></td>
  </tr>
</table>

## 🧩 How it works

<p align="center">
  <img src="docs/architecture.svg" alt="LifeLine architecture: the Indrapur city graph feeds a C++17 engine with core graph, data structures, routing and evacuation; response and resilience modules; a REST API; and a React command center" width="100%">
</p>

## 🚀 Quick start

```
git clone https://github.com/ishaan-sandhwar/lifeline.git
cd lifeline/backend
make            # builds ./lifeline (zero external dependencies)
./lifeline      # REST server on http://localhost:8080
```

Open **<http://localhost:8080>** — the demo map loads automatically.

- Optional SQLite history: `make USE_SQLITE=1` (needs `libsqlite3-dev`); the default build logs to a JSONL file instead.
- Console mode (no browser needed): `./lifeline console`
- Custom port: `./lifeline 9090`
- Unit tests: `make test`

<details>
<summary><b>🪟 Windows (native, MSYS2)</b></summary>

1. Install MSYS2 from <https://www.msys2.org> (one-time).
2. Open the **"MSYS2 UCRT64"** shell and install the toolchain:

```
pacman -S --needed mingw-w64-ucrt-x86_64-gcc make git
```

3. Clone, build, run (the Makefile auto-adds `-lws2_32` on Windows):

```
git clone https://github.com/ishaan-sandhwar/lifeline.git
cd lifeline/backend
make
./lifeline.exe
```

Prefer WSL? `wsl --install`, then follow the Linux steps above.

</details>

<details>
<summary><b>⚛️ Frontend (React)</b></summary>

A built `dist/` ships in the repo and is served by the C++ binary itself, so Node is **not** needed at runtime. To rebuild or develop:

```
cd frontend-react
npm install
npm run build        # -> dist/, which ./lifeline auto-serves at /
npm run dev          # hot reload: Vite on :5173, /api proxied to :8080
```

`frontend/map.html` remains as a zero-build fallback UI.

</details>

## 🧪 Testing

```
cd backend
make test
```

5 suites (`routing`, `evacuation`, `dispatch`, `resilience`, `analytics`), **96 checks**, run on the real Indrapur graph and on small hand-built graphs with known answers. What the results are compared against:

| Check | Compared against |
| --- | --- |
| A* distances | Dijkstra on sampled city pairs (optimality) |
| Bellman-Ford and Floyd-Warshall | Dijkstra on all 1,600 city pairs, also during a disaster |
| Max-flow and min-cut | Hand-checked flow networks and the max-flow = min-cut theorem |
| Bridges, articulation points, MST totals, diameter, centrality | An independent pure-Python verifier, `tools/verify_city.py`; the healthy-city bridge, articulation-point and MST numbers were also checked against networkx |

To print the verifier's ground truth yourself: `python3 tools/verify_city.py`.

## 📚 Reference

<details>
<summary><b>📂 Repo layout</b></summary>

```
lifeline/
├── backend/
│   ├── include/            httplib.h, json.hpp (third-party, header-only)
│   ├── src/
│   │   ├── core/           graph.h/.cpp, city_loader (frozen interface)
│   │   ├── ds/             min_heap, max_heap, hash_map, trie (all custom)
│   │   ├── routing/        dijkstra (+dijkstraAll), astar,
│   │   │                   bellman_ford, floyd_warshall
│   │   ├── evacuation/     Edmonds-Karp max flow + min cut
│   │   ├── dispatch/       triage + greedy dispatch, knapsack supplies
│   │   ├── resilience/     UnionFind, Tarjan, Prim + Kruskal
│   │   ├── simulation/     BFS disaster spread
│   │   ├── db/             history (SQLite optional, JSONL fallback)
│   │   ├── api/            REST server (cpp-httplib)
│   │   └── main.cpp        server mode + console mode
│   ├── data/city_graph.json
│   ├── tests/              5 suites, 96 checks
│   └── Makefile            `make`, `make USE_SQLITE=1`, `make test`
├── frontend-react/         React (Vite) command center
│   └── dist/               prebuilt — served by ./lifeline at /
├── frontend/map.html       zero-build fallback UI
├── docs/                   API spec, report (PDF), slides, screenshots
└── tools/
    ├── generate_city.py    regenerates city_graph.json (keeps A* admissible)
    └── verify_city.py      independent Python ground truth for the tests
```

</details>

<details>
<summary><b>🤔 Design decisions (FAQ)</b></summary>

**Why is the A\* heuristic admissible?** Every road length ≥ straight-line distance — enforced by `tools/generate_city.py` — so h never overestimates. Admissible + consistent ⇒ A* explores fewer nodes yet returns the same optimal distance.

**Why no `std::priority_queue`?** `src/ds/min_heap.h` is our own binary heap: sift-up/sift-down, O(log n) push/pop, lazy deletion instead of decrease-key.

**What is a disaster's "spread"?** Level-order BFS from the epicenter; each ring = one hop (`src/simulation/disaster.cpp`).

**Why Edmonds-Karp, not plain Ford-Fulkerson?** Augmenting paths are picked with **BFS** (shortest in edges), which bounds the algorithm to O(V·E²) and avoids the pathological slow cases (`src/evacuation/max_flow.cpp`).

**How are the bottleneck roads found?** The **min cut** — a BFS over the residual graph after max flow. Their capacities sum to the max flow; the unit tests assert this on the real city.

**Why a super-source/super-sink?** Many danger nodes → many shelters is a multi-source multi-sink problem; S and T with ∞-capacity arcs reduce it to single-pair max flow.

**Why a MAX-heap for triage, and how are ties broken?** Custom binary max-heap in `src/ds/max_heap.h`; a monotonic sequence number makes equal severities FIFO — stable triage.

**Is greedy assignment globally optimal?** No — it's locally optimal per incident. The globally optimal assignment is min-cost matching (Hungarian algorithm, O(n³)). Greedy was chosen because dispatch is an *online* problem — incidents arrive over time — and greedy is what real CAD systems approximate.

**Why ONE Dijkstra per incident instead of per vehicle?** `dijkstraAll()` gives distances to ALL vehicles in a single run; the road graph is undirected, so distance(vehicle→incident) = distance(incident→vehicle).

**How does knapsack loading work?** Classic 0/1 knapsack DP over integer weights; the chosen set is recovered by backtracking the DP table (`src/dispatch/supply.cpp`).

**Where is the custom hash map used?** Fleet counting in the dispatch response — separate chaining, djb2 hash, load-factor 0.75 doubling.

**Why is SQLite optional?** Compile-time `#ifdef USE_SQLITE` switch with a JSONL fallback — zero-dependency default build; identical History interface either way (`src/db/history.cpp`).

**How does ONE DFS find both bridges and articulation points?** disc/low arrays; bridge iff `low[child] > disc[u]`, articulation point iff `low[child] >= disc[u]`, root rule = 2+ children (`src/resilience/critical.cpp`).

**Union-Find complexity?** Path halving + union by rank ⇒ amortised inverse-Ackermann, effectively O(1) (`src/resilience/union_find.h`).

**Prim vs Kruskal — why run both?** Prim O(E log V) suits dense graphs, Kruskal O(E log E) suits sparse; both run and their totals are asserted equal — with distinct edge weights the MST is unique. Kruskal's edge ordering is a **heapsort** through our own MinHeap, not `std::sort`.

**Trie vs hash map for autocomplete?** Prefix queries in O(L) — a hash map can't enumerate "everything starting with riv" without a full scan.

</details>

<details>
<summary><b>🗺️ Project phases</b></summary>

- [x] **P1 Routing** — graph core, Dijkstra, A*, REST, demo map
- [x] **P2 Evacuation** — disaster spread (BFS), live road blocking, Edmonds-Karp max flow, min-cut bottlenecks
- [x] **P3 Dispatch** — custom max-heap triage, one-Dijkstra-per-incident greedy assignment, 0/1 knapsack supply loading, persistent history (SQLite via `make USE_SQLITE=1`, zero-dependency JSONL otherwise)
- [x] **P4 Resilience** — Union-Find (path halving + rank), Tarjan bridges/articulation points, Prim + Kruskal MST (heap-sorted, totals cross-checked), Trie autocomplete
- [x] **P5 Frontend + Integration** — React (Vite) command center served by the C++ backend, Bellman-Ford + Floyd-Warshall algorithm lab, city analytics

</details>

## 👥 Team

Built by a five-member team for the DSA course (PETV140) at Lovely Professional University, July 2026: five phases, one module per member, developed in parallel against a frozen `core/graph.h` interface.

**Ishaan Sandhwar · Harsh Sharma · Deepak Kumar Behera · Piyush Priyanshu · Aditya Shukla**

## 📜 License

[MIT](LICENSE)
