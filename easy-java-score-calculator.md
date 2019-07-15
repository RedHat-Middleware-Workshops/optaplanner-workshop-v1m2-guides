
# Easy Java Score Calculator

In this section you will learn

1. What a `ScoreCalculator` is.

2. How to implement an _Easy Score Calculator_ in Java.

3. How a `ValueRangeProvider` controls which planning values can be assigned to a planning entity.

In the previous lab we created our initial domain model, consisting of a planning entity (`CloudProcess`), a planning variable (the many-to-one `computer` relationship between `CloudProcess` and `CloudComputer`), and the planning solution (`CloudBalance`).

## The Score and Constraints

When solving a problem, OptaPlanner tries a lot of different potential solutions (in a smart way, using sophisticated A.I. heuristic algorithms). For each potential solution, OptaPlanner calculates the _score_ of the solution.

There are various types of _scores_, giving you flexibility wrt the score that you would like to use in your application, e.g.:

- `SimpleScore`: supports 1 score-level.
- `HardSoftScore`: supports 2 score-levels, hard and soft.
- `HardMediumSoftScore` supports 3 score-levels, hard, mediuma and soft,
- `BendableScore`: supports a configurable number of score levels.

.... and there are many more. Which score to use depends on the requirements of your project and the categorization of your constraints.

In this lab, in our _Cloud Balancing_ problem, we require 2 constraints and there 2 score levels:

- Hard Constraints: These are constraints that must not be broken. A broken hard constraints means that the given solution is _infeasible_. An example is when the processes that are assigned to a computer require more resources than are available on the computer.
- Soft Constraints: These are the constraints we want to optimize on, and usually are not 0. In our _Cloud Balancing_ problem, we want to minimize the costs, and hence the costs is our soft constraint.


To be able to keep track of the _score_ of our solution, the planning solution class, our `CloudBalance` class, needs to hold the score in a variable.

1. Open CodeReady Workspaces and open your project.

2. Add the following variable to your `CloudBalance` class and generate getters and setters for this new variable.

```
private HardSoftScore scoreHolder;
```
3. Annotate the getter of the score with the `@PlanningScore` annotation:

```
@PlanningScore
public HardSoftScore getScore() {
  return score;
}
```

4. Run a Maven Build to verify your project still compiles correctly.

With our _score_ variable added, our planning solution can now keep track of the score of a solution and compare it with the score of the current best solution.

## Planning Entity Collection

OptaPlanner needs to know which methods of our planning solution class provides the collection of planning entities. Since our planning entity is the `Process` class, we need to find the method that provides the collection of processes. In our `CloudBalance` class, this is the `getProcessList`. Annotate this method with the `@PlanningEntityCollectionProperty`:

```
@PlanningEntityCollectionProperty
public List<CloudProcess> getProcessList() {
  return processList;
}
```

## Planning Variable Ref

In order to be able to assign values to the planning entity's planning variable, OptaPlanner needs to know the possible values it can assign to the variable. This is done via a component called the `ValueRangeProvider`.

A `ValueRangeProvider` is usually defined on the `PlanningSolution`, although there are use-case in which the provider is defined on the `PlanningEntity`, for example when the possible value range is coupled to the entity itself. In our use-case however, the value range is the list of all available `CloudComputer`s, and is defined on our planning solution class `CloudBalance`

1. Open CodeReady Workspaces and open your project.

2. Open the `CloudBalance` class and locate its `getComputerList` method. This method provides the value range of our planning variable.

3. Annotate the method with the following annotation:
```
@ValueRangeProvider(id = "computerRange")
public List<CloudComputer> getComputerList() {
```

4. We now need to define the value range provider on our planning variable. Open the `CloudProcess` class, i.e. our planning entity. Locate the `getComputer` method that we've annotated earlier with our `@PlanningVariable` annotation. Add the _value range provider_ references to the annotation to refer to the `ValueRangeProvider` we just configured on our planning solution class.
```
@PlanningVariable(valueRangeProviderRefs = {"computerRange"})
```

5. Run a Maven Build to verify that the project compiles correctly.

In the next part of the lab, we will create the code and infrastructure to load our sample data.
