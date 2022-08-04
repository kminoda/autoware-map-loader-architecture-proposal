# Introduction
I would like to propose a new architecture for map loading. 

Current Autoware is not scalable in terms of the size of the map, since it loads the whole point cloud map at once. Thus, we are working on a new type of method for loading a point cloud map: dynamic map loading (DML). 


Through our experiment, however, it turned out that the current interface is not suitable for efficient DML. 今のアーキテクチャでDMLをしようとすると、数秒に一回周囲の地図を毎回読み込む必要があり、これがメモリ帯域などの計算リソースを圧迫してしまう。これは回避するには、差分読込を実装する必要があるが、現在のインターフェースでこれを実現することはできない（削除ができないから）。

Thus, we, TIER IV, would like to propose to the AWF community a new interface for map loading.
Note that this is a proposal for an additional interface (service) as an option, and is not intended to remove the current interface.

Here we also assume that the point cloud maps are divided beforehand, e.g. into 20m x 20m grids.

# Proposed architecture
The proposed architecture 

![The pipeline figure for design document](./figures/proposed_architecture.drawio.svg)

- [autoware_map_msgs/srv/LoadPCDMaps](https://github.com/kminoda/autoware-map-loader-architecture-proposal/blob/main/autoware_map_msgs/srv/LoadPCDMaps.srv)
- [autoware_map_msgs/msg/PCDMetaInfoArray](https://github.com/kminoda/autoware-map-loader-architecture-proposal/blob/main/autoware_map_msgs/msg/PCDMetaInfoArray.msg)

# Possible scenarios
## Whole map loading
This is the only scenario that the current Autoware supports, in which the client nodes load the whole available map at once.
Since the new interface is just an optional one, you can 

## Area loading
In case you want to load specific area from the map, 

## Differential area loading
As mentioned above, area loading is highly inefficient for DML, since the high load computation (SSD read, IO, ND voxelization) runs onboard.

Do note that this is different from area query. 


Given that we may need to load maps as efficient as possible to meet the limitaion of computational resources, we strongly believe that the new map architecture should support this type of query.

# Other architecture candidates
## Area-query, area-response
One of the other architectures we had in mind is this. 
[Put a figure here]

### Pros
- Simple

### Cons
- Requires high computational complexity algorithm for extracting points within the designated area

## Area-query, grids-response

[Put a figure here]

### Pros
- Fairly simple
- 

### Cons
- Still cannot support differential area loading ()