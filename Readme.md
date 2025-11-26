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
