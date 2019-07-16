# Drools Score Calculator

In this section you will learn

1. How to integrate OptaPlanner with Drools.
2. How to write constraints in the Drools Rule Language (DRL).

In the previous lab we've implemented an _Easy Score Calculator_ in Java. As we stated earlier, these kind of score calculators recalculate the score from scratch on every _move_ performed by OptaPlanner. This makes the score calculation extremely slow. In production scenarios, a high _Score Calculation Count (SCC)_ is extremely important in order to be able to handle real-life problems and problem sizes.

In this module we will look at Drools, an extremely high performant, light-weight, rules engine. Drools provides implicit support for incremental score calculation. Second, constraints can be implemented in individual, de-coupled, rules, which makes implementing, reasoning over, and maintaining constraint rules easier than when using a Java-based score-calculator.


## Drools Score Calculator

There are multiple ways in which we can utilize Drools rules in OptaPlanner. We can simply load the rules from a `.drl` (Drools Rule Language) file in our application, or we can separate the rules from our application and package them in a, so called, `KJAR` or _Knowledge JAR_.

In this lab we will define our rules in a `.drl` file.

1. Open CodeReady Workspaces and open your project from the previous lab. You can also import the following project from GitHub and use it as a starting point for this module: [https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-2](https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-2)

2. In the `src/main/resources/org/optaplannner/examples/cloudbalancing/solver` directory, create a new file called `cloudBalancingScoreRules.drl`.

DRL is a declarative rules language, as opposed to imperative languages like Java. This means that a rule file is not processed from top to botton, and that a rule file does not contain _"if-else"_ statements. Instead, rules are defined as _"when-then"_ statements, and the the order of rule evaluation and execution is data-driven and determined by the rules engine.

The engine reasons over _Facts_ (data objects) that are inserted into the _Session_ or _Working Memory_ of the engine. In the case of OptaPlanner this means that we need to insert all the objects of our problem into the rules engine, i.e. all our `CloudProcess` and `CloudComputer` instances. This makes our data available for evaluation. _Planning Entities_ are inserted into the rules engine by OptaPlanner automatically. However, we do need to insert all other facts that we want to make available to our rules. For this, OptaPlanner provides the `@ProblemFactCollectionProperty` annotation. We can annotate the _getter methods_ in our `PlanningSolution` class, that return a `Collection` that we want to insert into our rule engine, with this annotation.

1. Open the `CloudBalance` class and locate the `getComputerList` method. Annotate this method with the `@ProblemFactCollectionProperty`:
```
@ProblemFactCollectionProperty
@ValueRangeProvider(id = "computerRange")
public List<CloudComputer> getComputerList() {
	return computerList;
}
```

With the proper annotations set, we can start implementing our first rule. As Drools is a quite specific language, which would justify a full workshop on its own, we will not ask you to implement these rules yourself. Instead, we will provide you the constraint rules and will explain how they work.


1. Open the `cloudBalancingScoreRules.drl` file you just created.



















```
package org.optaplanner.examples.cloudbalancing.solver;

import org.optaplanner.core.api.score.buildin.hardsoft.HardSoftScoreHolder;

import org.optaplanner.examples.cloudbalancing.domain.CloudBalance;
import org.optaplanner.examples.cloudbalancing.domain.CloudComputer;
import org.optaplanner.examples.cloudbalancing.domain.CloudProcess;

global HardSoftScoreHolder scoreHolder;

// ############################################################################
// Hard constraints
// ############################################################################

rule "requiredCpuPowerTotal"
    when
        $computer : CloudComputer($cpuPower : cpuPower)
        accumulate(
            CloudProcess(
                computer == $computer,
                $requiredCpuPower : requiredCpuPower);
            $requiredCpuPowerTotal : sum($requiredCpuPower);
            $requiredCpuPowerTotal > $cpuPower
        )
    then
        scoreHolder.addHardConstraintMatch(kcontext, $cpuPower - $requiredCpuPowerTotal);
end

rule "requiredMemoryTotal"
    when
        $computer : CloudComputer($memory : memory)
        accumulate(
            CloudProcess(
                computer == $computer,
                $requiredMemory : requiredMemory);
            $requiredMemoryTotal : sum($requiredMemory);
            $requiredMemoryTotal > $memory
        )
    then
        scoreHolder.addHardConstraintMatch(kcontext, $memory - $requiredMemoryTotal);
end

rule "requiredNetworkBandwidthTotal"
    when
        $computer : CloudComputer($networkBandwidth : networkBandwidth)
        accumulate(
            CloudProcess(
                computer == $computer,
                $requiredNetworkBandwidth : requiredNetworkBandwidth);
            $requiredNetworkBandwidthTotal : sum($requiredNetworkBandwidth);
            $requiredNetworkBandwidthTotal > $networkBandwidth
        )
    then
        scoreHolder.addHardConstraintMatch(kcontext, $networkBandwidth - $requiredNetworkBandwidthTotal);
end

// ############################################################################
// Soft constraints
// ############################################################################

rule "computerCost"
    when
        $computer : CloudComputer($cost : cost)
        exists CloudProcess(computer == $computer)
    then
        scoreHolder.addSoftConstraintMatch(kcontext, - $cost);
end

```

You've successfully implemented your _hard constraints_ and _soft constraints_ in Drools. In the next module of this workshop we will look at the OptaPlanner _Benchmark_. This component allows us to benchmark various `ScoreCalculator` and heuristic algorithm combinations, as well as how they perform against different data-sets. This allows us to both validate our problem space/size, our implementation and the optimal configuration of algorithms for the given planning problem.
