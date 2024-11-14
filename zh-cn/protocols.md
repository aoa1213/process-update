# <font size=7>**Protocols.py**</font>

> <font size=4>Function to simulates quantum network protocols to distribute entangled states between users in a network.</font>

# <font size=5>Input:</font>

+ <font size=4>Graph G, user list, number of timesteps, number of repetitions, optional fusion flag.</font>

# <font size=4>Output:</font>

+ <font size=4>Entanglement rate, GHZ generation times, average links used.</font>

# <font size=4>Process:</font>

+ <font size=4> SP_protocol, MPG_protocol, MPC_protocol handle different entanglement routing strategies between users in the graph.</font>
+ <font size=4> _run_protocol runs the entanglement process for given repetitions and timesteps, resetting graph states and updating usage.</font>
+ <font size=4> _SD_protocol and _CC_protocol are used for determining protocol-specific success conditions.</font>
+ <font size=4> _create_bell_pair and _get_star create Bell pairs and determine star-shaped network paths.</font>


```mermaid
flowchart LR
    A[SP_protocol] --> B[Run entanglement along shortest path]
    A --> C[Update usage from subgraph]
    D[MPG_protocol] --> E[Sequentially run entanglement for multiple paths]
    F[MPC_protocol] --> G[Attempt entanglement along all edges]
    H[_run_protocol] --> I[Run entanglement step for timesteps and reps]
    J[_create_bell_pair] --> K[Create Bell pairs along path]
    L[_get_star] --> M[Get star paths connecting nodes]
    N[_count_node] --> O[]
    linkStyle default stroke:#ff0000,stroke-width:2px;
```


```python

import numpy as np


def run_entanglement_step(G, used_nodes, nodes=False):

    """
    simulate the link generation and decoherence for a single timeslot (step)

    Input Pararmeters:
    G          - Networkx graph G(V,E) which defines the topology of the network. see graphs.py for more details
    used_nodes - List of paths of used nodes (only updated if nodes parameter is True)
    nodes      - (optional) include simulating "node" entanglement in model
    """

    r_list = np.random.rand(
        len(G.edges())
    )  
    # array of random numbers between 0 and 1 (size of number of edges), which is the probability of generating entanglement

    for edge, r in zip(G.edges().values(), r_list):
            #G.edges(data=True)

        #   zip() took out pairs of data from G.edges().values() and r_list and assign to edge and r correspondingly
        #   G.edges().values() is the dictionary of each edges in the G graph
        #   e.g. edge =  (0, 1, {“entangled” ： True, "p_edge" : 0.7, "age" : 2, "Qc"(quantum capacity) : 5})
        #   r_list is a array of float number between 0 to 1 with the same size as number of edge, which is actually the probability of 


        if edge["entangled"]:  #  if entangled edge exists increase age
            edge["age"] += 1

            if (
                edge["age"] >= edge["Qc"]
            ):  #   If the edge is now too old then discard it - only required for entangled edges
                #   Qc is the threshold to dicard the entagnled states
                edge["entangled"] = False
                edge["age"] = 0

        if (
            not edge["entangled"] and edge["p_edge"] > r
        ):  #   greater is correct (hint p_edge = 0, rand =  0) and (hint p_edge = 1, rand =  0.999...)
            #   if the current edge is not in entangled state and the probebility of tanglemnet generation is larger than r
            #   r is the random number generated at the start
            edge["entangled"] = True 
            #   entangled is in the dictionary 
            edge["age"] = 0

    if nodes:
        for node_name in G.nodes():
            node = G.nodes[node_name]
            #   G.nodes() returns the identifiers of each node and assign it to node_name
            #   G.nodes[node_name] returns the dictionary of given identifier
            if node["entangled"]:
                node["age"] += 1
            #  If the node is entangled then the exsist time increase

            if node["age"] >= node["Qc"]:
                node["entangled"] = False
                edge["age"] = 0
            #   If the exsist time exceed the quantum capacity then the entagled ends

        for path in used_nodes:
            path["age"] += 1
        #   Iterate the path and 
        used_nodes[:] = [
            path for path in used_nodes if path["age"] < G.nodes[path["destination_node"]]["Qc"]
        ]
        #   path["destination_node"] returns the identifier of the destination node
        #   G.nodes[path["destination_node"]]["Qc"] returne the quatntum capacity of giiven identifier
        #   used_nodes[:] replace the whole context in the list
        """
        used_nodes = [1, 2, 3]
        used_nodes[:] = [4, 5, 6]
        print(used_nodes)  # print out：[4, 5, 6]
        """

```