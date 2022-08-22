We, TIER IV, would like to propose a new architecture for point cloud map loading.

Goal: Expand the size limit of the point cloud maps to improve the scalability of Autoware

# Introduction
Current Autoware is not scalable in terms of the size of the map, since it loads the whole point cloud (PCD) map at once. As far as we know, the size limit of PCD map in the current Autoware is around 2GB which is determined by the maximum size of a topic message in cycloneDDS. Thus, we are working on a new type of algorithm for loading a PCD map: dynamic map loading (DML). 

https://user-images.githubusercontent.com/44218668/176372161-1db133ec-ec5c-460e-a678-ddbd38cbbd94.mp4

Through our experiment, however, it turned out that the current interface is not suitable for efficient DML. For example, the naive DML (shown in the above video) newly loads all the PCD maps within the range of 200m from the ego-vehicle every several seconds, which may be too inefficient to perform in real time on limited computational resources.

Thus, we propose a differential DML, which reuses the overlapped PCD grids from the previous loading area to reduce the computation. For example, in the case below, the naive DML loads 38 grids (shown in the gray area), while the differential DML loads 6 grids (shown in blue). Note that the differential DML has to remove 6 grids (shown in red) in this case. 

<img src="./figures/differential_area_loading.gif" alt="drawing" width="400"/>

Unfortunately, the current map interface cannot provide sufficient information for client nodes to handle this complex management of PCD maps.
Thus, we, TIER IV, would like to propose to the AWF community a new interface to enable more flexible map loading.

Note that this is a proposal for an additional interface (service) as an option, and is not intended to remove any current interface.
Here we also assume that the PCD map is divided beforehand, e.g. into 20m x 20m grids (see [an additional proposal for map dividing format](./map_dividing_format.md)).

# Possible map loading scenarios
Here we briefly introduce possible map loading scenarios.

## Whole area loading
This is the only scenario that the current Autoware supports, in which the client nodes load the whole available map at once.
As long as the size of point cloud map does not induce any issues (i.e. communication size limit mentioned above), this scenario would be the most simple solution. 
Note that you can perform whole map loading in the same way as in the current Autoware, since the proposed architectures have no influence on the existing interface.

<img src="./figures/whole_map_loading.png" alt="drawing" width="400"/>

## Partial area loading
This scenario considers a case when a node (e.g. `pose_initializer`) only wants a limited area from the available PCD map. We assume that, given the area query, the node loads the PCD grids that overlap with the area query. This scenario may be needed when the point cloud map is too large to load at once.

<img src="./figures/area_loading_2.png" alt="drawing" width="400"/>


## Differential area loading
In this scenario, a node (e.g. `ndt_scan_matcher`) loads additional PCD grid maps (shown in blue) as well as removes maps that are no longer necessary (shown in red) at each step.
By reusing the maps that the node already has (shown in grey), the node can significantly reduce the computation that occurs in loading and preprocessing the map.
If we assume that a client node wants a set of map grids $M(t)$ for $t=t$ with differential area loading, the client node at $t=t+1$...
- loads $M(t+1) \backslash M(t)$
- unloads $M(t) \backslash M(t+1)$

where $A/B = \lbrace x\in A | x \notin B \rbrace$.

As discussed above, this scenario is needed when the point cloud map is too large and the partial loading scenario is not efficient enough.

<img src="./figures/differential_area_loading.gif" alt="drawing" width="400"/>


# Proposed architectures
We have two proposals, both of which can achieve the above-mentioned three scenarios. Since they both have their pros and cons, we would like to ask for your opinions from various perspectives. 

## Proposal A: sending ids as a query
![Proposed architecture](./figures/proposed_architecture_a.drawio.svg)

The architecture of proposal A is shown below. A client that want to use the new interface ("`client 1`" in the figure) first subscribes a message (see [here](./autoware_map_msgs/msg/PCDMetaInfoArray.msg) for the definition) that contains a dictionary of each grids' ID and its region information.
Using this information, the client selects the maps it wants and throw the query to the  `map_loader` with [autoware_map_msgs/srv/LoadPCDMaps](./autoware_map_msgs/srv/LoadPCDMaps.srv). `map_loader` loads the required maps and send them back as a response.

Note that in this case, we are also considering creating a library that covers all three scenarios mentioned above. 

The three scenarios mentioned above can be achieved as follows:
- Scenario 1 (whole area loading): use the `/map/pointcloud_map` topic as before (as shown in `client 2` in the above figure).
- Scenario 2 (partial area loading): calculate the map IDs you want in client side, using the given metadata list, and request them via the proposed service interface
- Scenario 3 (differential area loading): calculate the map IDs you want in client side, using the given metadata list, and request them via the proposed service interface

### Pros:
- more simple than proposal B, and easier to understand

### Cons:
- unnecessarily too general to achieve the above-mentioned scenarios
- heavier implementation cost for the client side (which can be reduced by the library, but still requires a maintenance cost)


## Proposal B: passing area and map ids that the client already has
![Proposed architecture](./figures/proposed_architecture_b.drawio.svg)

The architecture of proposal B is shown below. In this proposal, the client (`client 1` in the figure) sends the following two data as a query:
1. mode (0: partial area loading, 1: differential area loading)
2. the map area that the client wants (i.e. spherical area)

In case of `mode=0`, given the above two data, the `map_loader` returns all the map grids and their IDs that overlaps with the queried area.
In addition, when the client wants to use differential map loading (e.g. for more efficient computation), the client should additionally sends the following query:

3. map ids that the client already has

In case of `mode=1`, given the above three data, the `map_loader` returns...
- the map grids and their IDs (for the grid that overlaps with the queried area and that is not included in the third data)
- the map grids' IDs (for the grid that overlaps with the queried area and that is included in the third data)

The three scenarios mentioned above can be achieved as follows:
- Scenario 1 (whole area loading): use the `/map/pointcloud_map` topic as before (as shown in `client 2` in the above figure).
- Scenario 2 (partial area loading): use the proposed service interface (leave `already_loaded_ids` empty in this case)
- Scenario 3 (differential area loading): use the proposed service interface


The differential DML is expected to use `mode=1`.
(See also: [autoware_map_msgs/srv/LoadPCDMapsGeneral](./autoware_map_msgs/srv/LoadPCDMapsGeneral.srv))


### Pros:
- necessary and sufficient for achieving the above-mentioned three scenarios (not too general)
- able to reduce the implementation cost of the client side

### Cons:
- complicated interface and thus more difficult to understand than proposal A


## Other candidates that we had in mind
See [here](./other_candidates.md)
