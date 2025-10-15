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

- Time complexity: $O(b^{d+1})$ - BFS must generate all nodes up to depth $d+1$, where $d$ is the depth of the shallowest goal
- Space Complexity: $O(b^{d+1})$ - BFS must store all nodes in the frontier at the current depth, which is exponential in the depth of the search

- The complexity is exponential, so only very shallow search trees can be considered

### Depth-First Search

The frontier is a Stack (LIFO) in this case. This implementation is:

- Complete *IF* the search tree has a finite depth ($m$)
- Not optimal
- Time Complexity: $O(b^m)$ - Generated nodes = $1+b+...+b^m-1 = O(b^m)
- Space Complexity: $O(bm)$ - $m$ depth, succesors are stored at every depth

The execution time is very bad, $m >> d$, not complete and not optimal. The space complexity can be improved upon with a special implementation

### Searching algorithms - Graph search

```
graph_search()
    frontier <- {new_node(initial_state)}
    explored <- {}

    while true do

    if frontier.is_empty() return failure
    
    node <- frontier.take_first()
    if node.is_goal() return node

    explored.add(node)

    for each child in node.expand() do
        if child.state not in explored && child not in frontier
            frontier.add(child)
        end
    end
    end
```

If the state space is not a tree, then a simple tree search can either:
- fall into an infinite loop (e.g., cyclic graphs), or  
- revisit the same states many times, reducing efficiency drastically.

To handle this, we introduce an `explored` set (sometimes called *closed set*).  
It stores all previously visited states, ensuring that each state is expanded only once.  
Thus, every unique state is reached only via the first found path.

#### What if a better path to an already explored state is found later?
- If all step costs are equal (i.e., uniform), the first path to a state is guaranteed to be the shortest, so we don’t need to revisit it.
- However, if step costs vary, a later path might be cheaper.  
  In such cases (like in Uniform-Cost Search or A*), we must reinsert the node into the frontier if a better path is found — this is called path improvement or relaxation.
- For algorithms like DFS, which don’t guarantee optimality, we can “link over” if a better path is found — but this increases space complexity, as nodes may be revisited.

### Informed searching algorithms

With uninformed (*blind*) searching algorithms, the only information we had was the route we took to a specified state, but we didn't have information about how "good" the state was, i.e. how far (approximately) we were from the goal state.

In informed search algorithms, we have a heuristic function ($h(n)$), which is an estimate of how far a given state is from the goal state, for example: Euclidean distance in navigation scenarios

### Greedy Best-First Search

In the frontier, we sort according to $h(n)$, and take out the node with the smallest value.

This is similar to DFS, but it advances towards depth according to the minimalization of $h(n)$.

Condition: $h(n)=0$, for every $n$ goal state

> Conceptually, GBFS is like a depth-first search that "leans" towards the goal by following paths with the lowest estimated remaning costs

This algorithm is:

- Complete, if the search tree has a finite depth
- Not optimal: $h(n)$ is an estimate, actual cost is ignored
- Time Complexity: $O(b^m)$ (the worst)
- Space Complexity: $O(bm)$

In the case of graph searching, the explored set's elements need to be linked over

The worst case is very bad, but this can be improved upon with a good heuristic

### A* Algorithm

A* is an informed search algorithm that uses the evaluation function:

$$
f(n) = g(n) + h(n)
$$

- $g(n)$ = cost from the initial state to node $n$  
- $h(n)$ = heuristic estimate of the cost from $n$ to the goal

The frontier is maintained as a priority queue ordered by $f(n)$.  
At each step, A* removes the node with the smallest $f(n)$ for expansion.

**Notes:**

- If $h(n) = 0$ for all nodes, A* reduces to Dijkstra’s algorithm.  
- A heuristic $h(n)$ is admissible if it never overestimates the true cost to the goal.  
- A heuristic $h(n)$ is consistent (monotone) if, for every node $n$ and successor $n'$:

$$
h(n) \le c(n, a, n') + h(n')
$$

This ensures that $f(n)$ values never decrease along a path, which guarantees optimality in graph search.

### Properties of A*

- Optimality:  
  - Tree search: A* is optimal if $h(n)$ is admissible.  
  - Graph search: A* is optimal if $h(n)$ is consistent.

- Optimal efficiency:  
  - A* expands only those nodes $n$ for which $f(n) < C^*$, where $C^*$ is the cost of the optimal solution.  
  - Any algorithm guaranteed to find the optimal solution must expand at least these nodes.

- Time and space complexity:  
  - Worst-case complexity is **exponential** in the depth of the solution.  
  - A good heuristic can dramatically reduce the number of nodes expanded, improving practical performance.


## Two-Player, Turn-Based, Deterministic, Zero-Sum Games

These games can be modeled similarly to a state-space search, but with some differences. The problem model includes:

- **States:** The set of all possible configurations of the game.  
- **Initial state:** The starting configuration.  
- **State transition function:** Associates each state with a set of possible actions, where each action leads to a successor state.  
- **Goal states:** A subset of states where the game ends.  
- **Utility function:** Assigns a numeric value to each goal state representing the outcome for the MAX player.

**Gameplay structure:**

- There are two players who **take turns** applying actions.  
- The MAX player tries to maximize the utility, while the MIN player tries to minimize it (or equivalently maximize the negative utility).  
- These are zero-sum games, meaning that the sum of the players' utilities is zero at any terminal state.

**Graph model:**

- The game can be represented as a directed graph called the game graph, where nodes are states and edges are actions.  
- Unlike search trees, the game graph may contain cycles and multiple paths to the same state.

### Minimax Algorithm

Assume both players are perfectly rational: they have full knowledge of the game graph and never make mistakes. This is called the perfect rationality hypothesis.

A strategy specifies which action to take in each state.  
Under perfect rationality, the Minimax algorithm computes the optimal strategy for both players.

#### Minimax Value Calculation

For a node $n$:

- **Utility**: if $n$ is a terminal (goal) state, return its utility value.  
- **MAX node**: return the maximum of the minimax values of all successor nodes.  
- **MIN node**: return the minimum of the minimax values of all successor nodes.

The algorithm:

```
maxValue(n):
    if goal_state(n)
        return utility(n)
    
    max <- -inf
    for a in n's neighbors
        max = max(max, minValue(a))
    return max

minValue(n):
    if goal_state(n)
        return utility(n)
    min <- inf
    for a in n's neighbors
        min = min(min, maxValue(a))
    return min
```

Execution:
- Start with `maxValue(initial_node)` if the MAX player moves first.  
- The minimax value represents the optimal utility achievable from a state if the opponent plays perfectly.  
- On each turn, a player moves to the successor node with the highest (MAX) or lowest (MIN) minimax value.

#### Practical Considerations

- **Scalability:**  
  - The full game tree grows exponentially (e.g., chess has ~10^154 nodes), so exhaustive computation is impractical.  
- **Time Complexity:** \( O(b^m) \) where \( b \) = branching factor, \( m \) = depth of the tree.  
- **Loops / cycles:**  
  - In theory, cycles can prevent termination, but in practice:
    1. A fixed search depth is used.
    2. Game rules usually prevent infinite loops.

### Alpha-Beta Pruning

The problem with Minimax is that the number of nodes to examine grows exponentially with the depth of the tree.  
Alpha-Beta pruning reduces the number of nodes explored without affecting the final Minimax values.

#### Idea

- If MAX can guarantee a utility of at least `alpha` along the path to the root, then any branch where MIN can force a utility ≤ `alpha` **cannot occur**, and can be pruned.  
- Similarly, if MIN can guarantee a utility of at most `beta`, then branches where MAX can achieve ≥ `beta` can be pruned.

#### Alpha and Beta

- **Alpha (α):** the best guaranteed utility for MAX along the path from the current node to the root.  
- **Beta (β):** the best guaranteed utility for MIN along the path from the current node to the root.

The algorithm:
```
maxValue(n, alpha, beta):
    if goal_state(n)
        return utility(n)
    
    max <- -inf
    for a in n's neighbors
        max = max(max, minValue(a, alpha, beta))
        if max >= beta return max
        alpha = max(max, alpha)
    return max

minValue(n, alpha, beta):
    if goal_state(n)
        return utility(n)
    min <- inf
    for a in n's neighbors
        min = min(min, maxValue(a, alpha, beta))
        if min >= alpha return min
        beta = min(min, beta)
    return min
```


**Execution:**  
- Call `maxValue(initial_node, -inf, inf)` if MAX moves first.  
- Alpha and beta values are **inherited from parent nodes** and updated based on the best guaranteed utility along the path.  
- If a node’s value triggers a cutoff (MAX ≥ beta or MIN ≤ alpha), the remaining child nodes are **pruned**, since they cannot affect the final decision.

#### Time Complexity

- **Best case (optimal move ordering):** \(O(b^{m/2})\) — roughly half the nodes of full Minimax.  
- **Average / random ordering:** \(O(b^{3m/4})\) — still a significant improvement over \(O(b^m)\).  
- **Practical optimization:** Sorting or evaluating “likely best moves” first frequently approaches the optimal case.
