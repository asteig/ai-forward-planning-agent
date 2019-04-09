# Building a Forward-Planning Agent

Part of Udacity's Artificial Intelligence Nanodegree.

Planning is an important topic in AI because intelligent agents are expected to automatically plan their own actions in uncertain domains. Planning and scheduling systems are commonly used in automation and logistics operations, robotics and self-driving cars, and for aerospace applications like the Hubble telescope and NASA Mars rovers.

This project is split between implementation and analysis. First I combined symbolic logic and classical search to implement an agent that performs progression search to solve planning problems. Then I experimented with different search algorithms and heuristics, and used the results to answer questions about designing planning systems.

## Inconsistent Effects
Return True if an effect of one action negates an effect of the other

```python
def _inconsistent_effects(self, actionA, actionB):

    for effectA in actionA.effects:
        
        for effectB in actionB.effects:
            
            if effectA == ~effectB:
                return True
```

## Interference
Return True if the effects of either action negate the preconditions of the other 

```python
def _interference(self, actionA, actionB):
    
    for effect in actionA.effects:
        
        for precondition in actionB.preconditions:
            
            if precondition == ~effect:
                return True
```

## Competing Needs

Return True if the preconditions of the actions are all pairwise mutex in the parent layer 

```python
def _competing_needs(self, actionA, actionB):
    
    for preconditionA in actionA.preconditions:

        for preconditionB in actionB.preconditions:

            if self.parent_layer.is_mutex(preconditionA, preconditionB):
                return True

```


## Inconsistent Support
Return True if all ways to achieve both literals are pairwise mutex in the parent layer

```python
def _inconsistent_support(self, literalA, literalB):
    
    for actionA in self.parents[literalA]:

        for actionB in self.parents[literalB]:

            if not self.parent_layer.is_mutex(actionA, actionB):
                return False

    return True
```
## Negation
Return True if two literals are negations of each other.

```python
def _negation(self, literalA, literalB):
    
    if literalA == ~literalB and literalB == ~literalA:
        return True
```


# Heuristics 

## Level Sum

Calculate the level sum heuristic for the planning graph

The level sum is the sum of the level costs of all the goal literals
combined. The "level cost" to achieve any single goal literal is the
level at which the literal first appears in the planning graph. Note
that the level cost is **NOT** the minimum number of actions to
achieve a single goal literal.

For example, if Goal1 first appears in level 0 of the graph (i.e.,
it is satisfied at the root of the planning graph) and Goal2 first
appears in level 3, then the levelsum is 0 + 3 = 3.


```python
def h_levelsum(self):

    graph = self.fill()

    levelsum = 0

    for goal in self.goal:
        levelsum = levelsum + self.levelcost(graph, goal)

    return levelsum
```

## Max Level

Calculate the max level heuristic for the planning graph

The max level is the largest level cost of any single goal fluent.
The "level cost" to achieve any single goal literal is the level at
which the literal first appears in the planning graph. Note that
the level cost is **NOT** the minimum number of actions to achieve
a single goal literal.

For example, if Goal1 first appears in level 1 of the graph and
Goal2 first appears in level 3, then the levelsum is max(1, 3) = 3.

```python
def h_maxlevel(self):

    graph = self.fill()

    costs = []

    for goal in self.goal:
        costs.append(self.levelcost(graph, goal))

    return max(costs)
```

## Set Level
Calculate the set level heuristic for the planning graph

The set level of a planning graph is the first level where all goals
appear such that no pair of goal literals are mutex in the last
layer of the planning graph.

```python
def h_setlevel(self):

    while not self._is_leveled:
        
        layer = self.literal_layers[-1]

        if self.goal.issubset(layer):
            
            no_pairmutex = True

            for goal1 in self.goal:
                for goal2 in self.goal:
                    if layer.is_mutex(goal1, goal2):
                        no_pairmutex = False
                        break

            if no_pairmutex:
                return len(self.literal_layers) - 1

        self._extend()

    return len(self.literal_layers) - 1   
```
