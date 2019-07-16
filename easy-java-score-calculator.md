
# Easy Java Score Calculator

In this section you will learn

1. What a `ScoreCalculator` is.

2. How to implement an _Easy Score Calculator_ in Java.

In the previous module we created our solver application, consisting of domain model, solver config, etc. In this module we will be implementing our solver constraints.

## The Score and Constraints

When solving a problem, OptaPlanner tries a lot of different potential solutions (in a smart way, using sophisticated A.I. heuristic algorithms). For each potential solution, OptaPlanner calculates the _score_ of the solution.

There are various types of _scores_, giving you flexibility wrt the score that you would like to use in your application, e.g.:

- `SimpleScore`: supports 1 score-level.
- `HardSoftScore`: supports 2 score-levels, hard and soft.
- `HardMediumSoftScore` supports 3 score-levels, hard, mediuma and soft,
- `BendableScore`: supports a configurable number of score levels.

.... and there are many more. Which score to use depends on the requirements of your project and the categorization of your constraints.

In this lab, in our _Cloud Balancing_ problem, we require 2 score levels:

- Hard Constraints: These are constraints that must not be broken. A broken hard constraints means that the given solution is _infeasible_. An example is when the processes that are assigned to a computer require more resources than are available on the computer. A solution to a planning problem should therefore never break hard constraints.
- Soft Constraints: These are the constraints we want to optimize on, and usually are not 0. In our _Cloud Balancing_ problem, we want to minimize the costs, and hence the _costs_ is our soft constraint.

To be able to keep track of the _score_ of our solution, the planning solution class, our `CloudBalance` class, needs to hold the score in a variable. We've already added a `@PlanningScore` variable called `score` to our `CloudBalance` planning solution class in the previous module. We've also added a skeleton/dummy score calculator, `CloudBalancingEasyScoreCalculator`, to our application in order to complete our _SolverConfig_ and do an initial run of our application. We well now implement the constrains int the `CloudBalancingEasyScoreCalculator` class.

1. Open CodeReady Workspaces and open your project from the previous lab. You can also import the following project from GitHub and use it as a starting point for this module: [https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-1](https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-1)

2. Open the `CloudBalancingEasyScoreCalculator` class. Locate the method `calculateScore(CloudBalance solution)`.

The first constraint we want to implement are the hard constraints of this problem. We first need to identify these constraints. The question we need to answer is: "What makes a solution to a problem _infeasible_?" In the case of our _Cloud Balancing_ problem, a solution is infeasible if the _resources_ required by _all processes_ assigned to a _computer_ exceed the available _resources_ of the given _computer_. In this problem we have 3 types of resources:
- cpuPower
- memory
- networkBandwidth

To implement our hard constraint, we need to determine which `CloudProcess` has been assigned to which `CloudComputer`, and determine whether the accumulated resource requirements is below the available resources provided by the computer.

Let's first implement the logic to determine which `CloudProcess` has been assigned to which `CloudComputer`. Note that you w

1. You are challenged to implement this part of the logic yourself. Implement the logic that collects the resource requirements of all 3 _resources_ of all the _processes_ for each _computer_.

    - HINT 1: You can access the `List` of `CloudComputer` from the `solution` variable (which is an instance of `CloudBalance`).
    - HINT 2: You can access the `List` of `CloudProcess` from the `solution` variable (which is an instance of `CloudBalance`).
    - HINT 3: You can determine to which `CloudComputer` a `CloudProcess` has been assigned via the `CloudProcess.computer` variable.
    - HINT



    **SOLUTION BELOW** ... Try not to cheat!




2. The solution should look like this:

```
public HardSoftScore calculateScore(CloudBalance cloudBalance) {
  int hardScore = 0;
  int softScore = 0;
  for (CloudComputer computer : cloudBalance.getComputerList()) {
    int cpuPowerUsage = 0;
    int memoryUsage = 0;
    int networkBandwidthUsage = 0;

    // Calculate usage
    for (CloudProcess process : cloudBalance.getProcessList()) {
      if (computer.equals(process.getComputer())) {
        cpuPowerUsage += process.getRequiredCpuPower();
        memoryUsage += process.getRequiredMemory();
        networkBandwidthUsage += process.getRequiredNetworkBandwidth();
      }
    }
  }
  return HardSoftScore.of(hardScore, softScore);
}

```

Although we have defined the logic that collects the resource requirements of our _processes_ per _computer_, we have not yet implemented any constraints. Lets' first implement the _hard constraints_. When looking at the requirements we can identify 3 _hard constraints_:

- the `cpuPowerUsage` should not exceed the _computer_'s available `cpuPower`.
- the `memoryUsage` should not exceed the _computer_'s available `memory`.
- the `networkBandwidtUsage` should not exceed the _computer_'s available `networkBandwidth`.

1. Implement the logic that determines whether the `cpuPower` of the given `CloudComputer` is exceeded by the _processes_ assigned to the computer. If the `requiredCpuPower` exceeds the `cpuPower` of the `CloudComputer`, we will decrease the `hardScore` with the amount of missing CPU Power. (we _decrease_ the score, as we use a _negative_ score to indicate broken constraints in our implemenation. Using _negative_ scores in an OptaPlanner application usually feels more natural than using positive scores, although OptaPlanner supports both).


    **SOLUTION BELOW** ... Try not to cheat!

2. A possible solution is show below. In this

```
int cpuPowerAvailable = computer.getCpuPower() - cpuPowerUsage;
if (cpuPowerAvailable < 0) {
  hardScore += cpuPowerAvailable;
}
```

3. Implement the other 2 hard constrains in the same way.


    **SOLUTION BELOW** ... Try not to cheat!


4. The full solution of the _hard constraint_ implementation looks like this:

```
public HardSoftScore calculateScore(CloudBalance cloudBalance) {
  int hardScore = 0;
  int softScore = 0;
  for (CloudComputer computer : cloudBalance.getComputerList()) {
    int cpuPowerUsage = 0;
    int memoryUsage = 0;
    int networkBandwidthUsage = 0;
    boolean used = false;

    // Calculate usage
    for (CloudProcess process : cloudBalance.getProcessList()) {
        if (computer.equals(process.getComputer())) {
            cpuPowerUsage += process.getRequiredCpuPower();
            memoryUsage += process.getRequiredMemory();
            networkBandwidthUsage += process.getRequiredNetworkBandwidth();
        }
    }

    // Hard constraints
    int cpuPowerAvailable = computer.getCpuPower() - cpuPowerUsage;
    if (cpuPowerAvailable < 0) {
        hardScore += cpuPowerAvailable;
    }
    int memoryAvailable = computer.getMemory() - memoryUsage;
    if (memoryAvailable < 0) {
        hardScore += memoryAvailable;
    }
    int networkBandwidthAvailable = computer.getNetworkBandwidth() - networkBandwidthUsage;
    if (networkBandwidthAvailable < 0) {
        hardScore += networkBandwidthAvailable;
    }
    return HardSoftScore.of(hardScore, softScore);
}

```

With our hard constraints implemented, we can now look at our _soft constraints_. The _soft constraints_ are the constraints we want to optimize on. Usually a planning problem has many soft constraints, but in this case we will only implement one: _the costs_ of our _computers_. (Another possible _soft constraint_ of our use-case could be to reach a 80% resource utilization of our _computers_ to make sure we can safely accomadate for peaks).

Analysing the _soft constraint_, we can see that we have to _minimize_ the _cost_ of our solution. We have to pay for a `CloudComputer` when at least one `CloudProcess` has been assigned to it.

1. Implement the _soft constraint_ of our planning problem.


    **SOLUTION BELOW** ... Try not to cheat!


2. The possible full solution of our score calculation method is the following. Note the `used` boolean that is introduced to keep track of which `CloudComputers` are used. This allows us to easily determine which `CloudComputer` is used, and therefore, which `cost` we need to add to our _soft constraint_.


```
public HardSoftScore calculateScore(CloudBalance cloudBalance) {
  int hardScore = 0;
  int softScore = 0;
  for (CloudComputer computer : cloudBalance.getComputerList()) {
      int cpuPowerUsage = 0;
      int memoryUsage = 0;
      int networkBandwidthUsage = 0;
      boolean used = false;

      // Calculate usage
      for (CloudProcess process : cloudBalance.getProcessList()) {
          if (computer.equals(process.getComputer())) {
              cpuPowerUsage += process.getRequiredCpuPower();
              memoryUsage += process.getRequiredMemory();
              networkBandwidthUsage += process.getRequiredNetworkBandwidth();
              used = true;
          }
      }

      // Hard constraints
      int cpuPowerAvailable = computer.getCpuPower() - cpuPowerUsage;
      if (cpuPowerAvailable < 0) {
          hardScore += cpuPowerAvailable;
      }
      int memoryAvailable = computer.getMemory() - memoryUsage;
      if (memoryAvailable < 0) {
          hardScore += memoryAvailable;
      }
      int networkBandwidthAvailable = computer.getNetworkBandwidth() - networkBandwidthUsage;
      if (networkBandwidthAvailable < 0) {
          hardScore += networkBandwidthAvailable;
    }

    // Soft constraints
    if (used) {
        softScore -= computer.getCost();
    }
  }
  return HardSoftScore.of(hardScore, softScore);
}

```

Now that we've implemented both the _hard constraints_ and _soft constraints_ of our solution, we can test our application again.

1. Run the `CloudBalancingSolveTest` by running a Maven Build. The output should show the test being executed.

Observe that the debug output of our unit-test now shows a soft score that is -7410 soft:

```
14:08:39.973 [main] DEBUG org.optaplanner.core.impl.localsearch.DefaultLocalSearchPhase -     LS step (0), time spent (28), score (0hard/-7410soft),     best score (0hard/-7410soft), accepted/selected move count (1/8), picked move (org.optaplanner.examples.cloudbalancing.domain.CloudProcess@126253fd
```

Also note that we don't see any _hard constraints_ being broken, nor any improvement in the _soft score_. This is due to the fact that we have a very small problem with only 4 computers and 12 processes. Let's add a bigger problem to our workspace so we can see OptaPlanner's optimization algorithms in action.

1. In the `data/cloudbalancing/unsolved` folder of your project, create new filed called `100computers-300processes.xml`.
2. Copy the content of [this file](https://raw.githubusercontent.com/kiegroup/optaplanner/master/optaplanner-examples/data/cloudbalancing/unsolved/100computers-300processes.xml) into your file and save it.
3. Open your `CloudBalancingSolverTest` class and change the `inputFile` of your `testSolver` method to:
    ```
    File inputFile = new File("data/cloudbalancing/unsolved/100computers-300processes.xml");
    ```
4. Run the `CloudBalancingSolveTest` by running a Maven Build. The output should show the test being executed.

When we now observe the output, we can see that OptaPlanner is constantly finding better and better solutions (solutions with a lower _soft score_) to our problem. We can also see that when OptaPlanner finds a better solution than the current _best solution_, it prints the following statement:

```
15:24:51.930 [main] DEBUG org.optaplanner.core.impl.localsearch.DefaultLocalSearchPhase -     LS step (2230), time spent (1440), score (0hard/-132300soft), new best score (0hard/-132300soft), accepted/selected move count (1/2), picked move (org.optaplanner.examples.cloudbalancing.domain.CloudProcess@10ded6a9 {org.optaplanner.examples.cloudbalancing.domain.CloudComputer@7f2cfe3f -> org.optaplanner.examples.cloudbalancing.domain.CloudComputer@50313382}).
```

Note the sentence _new best score_. This indicates that OptaPlanner has found a better score, and this score will be the new _best_ score. This means that when we stop the solver, this solution will be returned.

We've completed the implementation of our _Easy Score Calculator_ in Java. In the next part of the lab, we will implement the same constraints with the Drools Rule Engine. Implementing constraints in Drools is preferred, as Drools implicitly supports incremental calculation of the score, which greatly improves the _score calculation count_ of the solution. A high _score calcution count (SCC)_ is a requirement in production scenarios to be able to deal with real-world planning problems.
