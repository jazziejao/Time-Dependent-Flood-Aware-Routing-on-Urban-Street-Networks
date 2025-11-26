- Cite the work as (**status**: paper accepted, pending publication and DOI)
```
Jao, J., Vallar, E., Hameed, I., Liu, Y., & Era, M. (2025). _FLOAT: Flood-Aware Trajectories for Modeling the Time-Dependent Evacuation Routing with Risk consideration_.
```

- ``input/df_rr04_6.csv``: input street network
- `batch_results`: output folder of the batch experiment for flood-aware vs shortest path algorithm.
---
# Requirements
- An edgelist representing the road street network.
- This edgelist must have the following columns:
	1. ID of Source Intersection
	2. ID of Target Intersection
	3. Flood Vector
		- Must be defined within uniform time window
	4. Street Length
	5. Geometry

---
![[showcase1.jpg]]


---
# FloodAwareRouter

## Overview

`FloodAwareRouter` is a Python class designed for routing optimization in flood-prone areas. It computes routes that balance travel time against flood risk, with support for required edge visits, one-way street handling, and dynamic flood level visualization.

## Class Initialization
```python
FloodAwareRouter(
    df_edges: pd.DataFrame,
    G: nx.Graph,
    random_seed=None,
    threshold=0.1,
    alpha=0.3,
    lambda_tradeoff=0.001,
    risk_limit=1,
    animation_interval=600,
    oneway_penalty_weight=0.0
)
```

### Parameters

- **df_edges** (`pd.DataFrame`): DataFrame containing edge information with columns including 'u', 'v' for node pairs
- **G** (`nx.Graph`): NetworkX graph representing the road network with edge attributes:
  - `length`: Edge length in meters
  - `maxspeed`: Maximum speed in m/s
  - `flood_levels`: Array of flood levels over time
  - `elev_mean`: Mean elevation (optional)
  - `oneway`: Binary flag for one-way streets (optional)
  - `geometry`: LineString geometry for visualization
- **random_seed** (`int`, optional): Seed for reproducibility
- **threshold** (`float`): Flood level threshold above which speed is reduced (default: 0.1)
- **alpha** (`float`): Speed reduction factor for flooded edges (default: 0.3)
- **lambda_tradeoff** (`float`): Weight for time vs risk tradeoff (default: 0.001)
- **risk_limit** (`float`): Maximum acceptable cumulative risk for a route (default: 1)
- **animation_interval** (`int`): Interval between animation frames in ms (default: 600)
- **oneway_penalty_weight** (`float`): Penalty weight for one-way streets (default: 0.0)

## Core Methods

### `compute_edge_cost(u, v, t)`

Calculates the cost of traversing an edge at a given time, incorporating travel time, flood risk, and one-way penalties.

**Parameters:**
- `u`, `v`: Node IDs defining the edge
- `t`: Time index for flood level lookup

**Returns:**
- Tuple of `(cost, travel_time, risk)`

**Risk Components:**
- **Flood component**: Normalized flood level (capped at 1.0)
- **Elevation component**: Based on elevation relative to flood depth
- **Exposure component**: Based on travel time spent in flooded area

---

### `flood_aware_routing(t=0)`

Computes a flood-aware route from a depot to a required edge and back, respecting risk limits.

**Parameters:**
- `t` (`int`): Time index for flood conditions

**Returns:**
- List of node IDs representing the complete route, or empty list if no valid route exists

**Behavior:**
- Handles one-way streets by enforcing directional constraints
- For two-way edges, attempts both directions and selects the first valid path
- Validates total route risk against `risk_limit`

---

### `export_route_metrics(route, filename='route_metrics.csv', global_start_time=0)`

Exports detailed route metrics to CSV and GeoJSON formats.

**Parameters:**
- `route`: List of node IDs representing the path
- `filename` (`str`): Output filename (CSV format)
- `global_start_time` (`int`): Starting time in seconds for flood index calculation

**Output Files:**
- CSV file with segment-by-segment metrics
- GeoJSON file with geometric data for mapping

**Metrics Exported:**
- Edge endpoints (from/to)
- Length and max speed
- Flood level
- Effective travel time
- Risk score
- Cost value
- Geometry (LineString)

**Summary Statistics:**
- Total travel time
- Total flood risk
- Number of flooded segments
- Cumulative flood level

---

### `is_route_continuous(route)`

Validates that a route forms a continuous path through the graph.

**Parameters:**
- `route`: List of node IDs

**Returns:**
- Boolean indicating whether the route is valid and continuous

**Checks:**
- All consecutive node pairs exist as edges in the graph
- No discontinuities between segments

---

### `visualize_route_as_gif(route_gdf, full_network_df, filename='route_animation.gif', global_start_time=0)`

Creates an animated GIF showing route progression with dynamic flood visualization.

**Parameters:**
- `route_gdf` (`gpd.GeoDataFrame`): Route segments with geometry and metrics
- `full_network_df` (`pd.DataFrame`): Complete road network with flood data
- `filename` (`str`): Output GIF filename
- `global_start_time` (`int`): Starting time in seconds

**Features:**
- Frame-by-frame visualization of route traversal
- Background network colored by current flood levels
- Current position marker (red dot)
- Cumulative statistics overlay (flood accumulation, time elapsed)
- Basemap integration (CartoDB Positron)

**Requirements:**
- Geometries must be in EPSG:3857 (Web Mercator) projection
- Full network must include time-indexed flood columns (format: `rr04-XX`)

---

### `run_all(global_start_time=0, export_prefix='output')`

Executes a complete routing workflow including shortest path comparison, flood-aware routing, validation, and export.

**Parameters:**
- `global_start_time` (`int`): Starting time in seconds
- `export_prefix` (`str`): Prefix for output files

**Workflow:**
1. Randomly selects a depot node and required edge
2. Computes shortest path (length-based) route
3. Computes flood-aware route
4. Exports metrics for both routes
5. Validates that both routes visit the required edge
6. Checks route continuity

**Output Files:**
- `{export_prefix}_shortest.csv` and `.geojson`: Shortest path metrics
- `{export_prefix}_floodaware.csv` and `.geojson`: Flood-aware route metrics

---

## Internal Helper Methods

### `_get_effective_speed(flood_level, speed)`

Returns reduced speed if flood level exceeds threshold.

---

### `_route_visits_required(route)`

Checks whether a route traverses the required edge in either direction.

---

## Usage Example
```python
# Initialize router
router = FloodAwareRouter(
    df_edges=edges_df,
    G=road_network,
    random_seed=42,
    threshold=0.1,
    lambda_tradeoff=0.5,
    risk_limit=2.0
)

# Set specific depot and required edge (optional)
router.depot = 12345
router.required_edge = (67890, 11111)

# Run complete workflow
router.run_all(global_start_time=1800, export_prefix='mission_1')

# Or compute individual route
route = router.flood_aware_routing(t=30)
router.export_route_metrics(route, 'custom_route.csv', global_start_time=1800)
```

---

## Dependencies

- `pandas` (pd)
- `numpy` (np)
- `networkx` (nx)
- `geopandas` (gpd)
- `matplotlib.pyplot` (plt)
- `imageio`
- `contextily` (ctx)
- `shapely.geometry` (LineString)
- `shapely.wkt`

---

## Notes

- Time indices are calculated as `int(global_start_time // 60)` and capped at 59
- The cost function balances time and risk using `lambda_tradeoff`: higher values prioritize time, lower values prioritize safety
- One-way penalty is fixed at 10.0 in `compute_edge_cost` (adjustable in code)
- Risk calculation uses hyperbolic tangent for smooth elevation effects
- Animation frames are temporarily stored in `frames/` directory and removed after GIF creation