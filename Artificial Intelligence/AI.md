# Artificial Intelligence

Collection of notes for my "Artificial Intelligence" course. No not the hyped up version.

## Search-based problem solving

Search-based problem solving is a type of goal-oriented agent framework. It involves finding a sequence of actions that leads from an initial state to a goal state. This is a general framework in which:
- A goal or goals are set
- The problem is described, i.e. state-space representation
- Solving the problem is searching within the state-space
- Executing the solution, i.e. executing the action sequence

This approach can be applied when the environment is:
- Static - The environment does not change
- Discrete - It's describeable in a finite or coutably infinite amount of states
- Deterministic - The next state is fully defined by the current state and a chosen action, i.e. there are no random components
- Completely observable

### State space

The state space is an abstract representation of a problem that captures all possible configurations of the system and how they can change. It encodes only the information relevant to solving the problem.

The following need to be described in a state space:
- The set of possible states (graph nodes)
- The set of possible actions (graph verticies)
- The initial state
- The set of goal states
- The cost function, *g(n)*: The numerical cost associated to a state transition

The state space can be represented as a directed graph, where edges correspond to actions and edge weights (if any) represent the cost of performing those actions.

**Example:**

We want to travel from Arad to Budapest. This problem can be represented and solved using graph search as follows:

1. Create a state space representation of the map.
    - Each city represents a state.
    - Each road between cities represents an action that causes a transition to a new state.
    - The cost of each action is the distance (or travel time) between the two cities.
    - The initial state is Arad, and the goal state is Budapest.

2. Search the state space to find a sequence of actions (a route) that leads from Arad to Budapest.
    - The search algorithm explores possible paths through the graph.

3. Execute the solution by following the sequence of actions found.

The total cost of the solution is the sum of the costs of all transitions (i.e. the total distance traveled).

### Searching algorithms - Tree search

```
tree_search():
    frontier <- {new_node(initial_state)}       // create a new node representing the initial state

    while true do                               // infinite loop

    if frontier.is_empty() return failure       // if there are no nodes left to explore, we fail to find goal

    node <- frontier.pop()                      // we take one node off the frontier
    if node.is_goal_state() return node         // if the node is a goal state, we succeed and return
    else frontier.add(node.expand())            // if the node is not a goal state, we expand the node
                                                // (all states reachable by one action) and add them to
                                                // the frontier

    end                                         // loop around
```
We search for the shortest path from an initial state to a goal state.

We grow the search tree from the initial state by adding neighbor states until we find a goal state.

The implementation of the `frontier` is what defines the strategy of the search:
- Depth-First Search - frontier is a stack
- Breadth-First Search - frontier is a queue
- A* - frontier is a priority queue by `g(n) + h(n)`

### Analysis of searching algorithms

- Complete:

An algorithm is complete if it is guaranteed to find a solution whenever a goal state is reachable by traversing a finite number of states.

- Optimal:

An algorithm is optimal only if it is complete and if, among all possible goal states, it return the one with the lowest total path cost.

We do not measure the time complexity and space complexity of an algorithm relative to the size of the state space, rather:

- `b`: The Branching Factor, i.e. the maximum number of successors any node can have
- `m`: The maximum depth of the search tree
- `d`: The depth of the shallowest node

### Breadth-First Search

In this implementation of a tree search, the frontier is a Queue (FIFO). This implementation is:

- Complete: It reaches every state that can be reached by traversing a finite amount of states

- Optimal *IF* all actions have the same cost (or equivalently, if the path cost is a non-decreasing function of depth), otherwise, a shallower but more expensive path might be found before a cheaper one.

- Time complexity: $O(b^(d+1))$ - BFS must generate all nodes up to depth $d+1$, where $d$ is the depth of the shallowest goal
- Space Complexity: $O(b^(d+1))$ - BFS must store all nodes in the frontier at the current depth, which is exponential in the depth of the search

- The complexity is exponential, so only very shallow search trees can be considered

### Depth-First Search

The frontier is a Stack (LIFO) in this case. This implementation is:

- Complete *IF* the search tree has a finite depth ($m$)