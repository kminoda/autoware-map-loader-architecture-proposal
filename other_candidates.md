# Other architecture candidates
Here we briefly explain about other architecture candidates that we had in mind. 


## Area-query, area-response
![Candidate architecture 2](./figures/candidate_architecture_2.drawio.svg)

The most simple idea for partial area loading is to load all the points within the queried area ( $M(t) = \lbrace p|p\in A(t) \rbrace$ where $M(t)$ is a set of points in a map, $p$ is a point, and $A(t)$ is an area query). 

### Pros
- simple

### Cons
- requires high computational complexity algorithm for extracting points within the designated area

## Area-query, grids-response
![Candidate architecture 3](./figures/candidate_architecture_3.drawio.svg)

### Pros
- fairly simple

### Cons
- still cannot support differential area loading


## ROS 2 topics
![Candidate architecture 1](./figures/candidate_architecture_1.drawio.svg)

The whole area loading and partial area loading can be easily achieved with ROS2 topics, but implementing the differential area loading is somewhat more complicated. 
One of the points we considered in this proposal is to guarantee that each client receives the initial map. This is because the ROS 2 topic is a one-way communication, and depending on the QoS settings and the order in which nodes are activated, it is possible that a client may miss receiving the initial map. Unlike the partial area loading scenario, failure in loading the initial map in differential area loading scenario can be critical for the client node.

### Pros
- less implementation cost for the clients

### Cons
- may be difficult to guarantee that all the messages are successfully subscribed
- implementations of multiple client nodes will be dependent (since all the nodes subscribes to one single topic)
