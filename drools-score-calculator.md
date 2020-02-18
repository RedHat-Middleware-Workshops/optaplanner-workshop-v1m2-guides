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

~~~java
@ProblemFactCollectionProperty
@ValueRangeProvider(id = "computerRange")
public List<CloudComputer> getComputerList() {
	return computerList;
}
~~~

With the proper annotations set, we can start implementing our first rule. As Drools is a quite specific language, which would justify a full workshop on its own, we will not ask you to implement these rules yourself. Instead, we will provide you the constraint rules and will explain how they work.


1. Open the `cloudBalancingScoreRules.drl` file you just created. We first need to define a `package` for our rules. Add the following line to the top of the file:

~~~java
package org.optaplanner.examples.cloudbalancing.solver;
~~~

2. The Drools rule engine works with Java objects as _Facts_. In order to be able to use Java objects in a rule, we need to import them into the `.drl` file. To write our constraints, we need to evaluate the following _Fact types_:

    - `CloudProcess`
    - `CloudComputer`
    - `HardSoftScoreHolder`

    The `ScoreHolder` is required to allow us to change the score of our solution when a rule matches and fires. Add the following lines to your `.drl` file:

~~~java
import org.optaplanner.core.api.score.buildin.hardsoft.HardSoftScoreHolder;

import org.optaplanner.examples.cloudbalancing.domain.CloudComputer;
import org.optaplanner.examples.cloudbalancing.domain.CloudProcess;
~~~

3. We need access to the `ScoreHolder` in order to manipulate it. The `ScoreHolder` however is not a _Fact_, as it is not part of the _condition_ of any of our constraint rules. Drools allows us to make this kind of data accessible to the rules via _global variables_. These _variables_ are accessible by the rules, but are not reasoned over during rule evaluation. OptaPlanner will automatically insert this _global_ `ScoreHolder` _variable_ into the rules engine when we define this variable in our rules file. Add the following line to your `.drl` file:

~~~java
global HardSoftScoreHolder scoreHolder;
~~~

With our package name, imports and global variable definitions in place, we can now start implementing our constraint rules. We will start by implementing the constraint rule for the `cpuPower` _hard constraint_. After implementing that constraint we will ask you to implement the other 2 _hard constraints_ in a similar way. Finally we wil implement the _soft constraint_ that aims to minimize the `cost` of the solution.

A rule in Drools consist of 3 parts:
- the rule name and properties
- the conditions/constraints, or left-hand-side (LHS)
- the action/consequence, or right-hand-side(RHS)

As stated earlier, a Drools rule implements a _when-then_ semantic. The syntax of a rule looks like this:

~~~
rule {name}
when
  {conditions}
then
  {action}
end
~~~

The hard constraint we're going to implement is the constraint that verifies that the computer's available `cpuPower` is not exceeded by the processes assigned to it. The rule that implements that constraint is the following:

~~~
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
~~~

Let's first explain the _left-hand-side_ of the rule. The first line matches on eveyr `CloudComputer` _Fact_. The `$computer :` and `$cpuPower :` are variable assignments. This allows us to later reference the `CloudComputer` and its `cpuPower` attribute in another constraint, or in the action part of our rule.

The `accumulate` is a keyword to use the built-in accumulate function in Drools. Accumulates allow us to accumulate multiple facts that match one or more conditions and apply functions to them. In this case our `accumulate` function collects all the `CloudProcess` facts of which the assigned `CloudComputer` is equal to the `CloudComputer` we matched in the firt condition of our rule. In other words, we collect all `CloudProcess` facts assigned to this `CloudComputer`. Next, we use the `sum` function of out `accumulate` to sum up the total required `cpuPower` of all processes combined. Finally, we apply the conditiion in which we check that the sum of `requiredCpuPower` is higher than the computer's available `cpuPower`. When that constraint matches, our rule fires. Note that sum of `requiredCpuPower` is also assigned to a variable, i.e. '$requiredCpuPowerTotal'. This allows us to use this variable in the action part of our rule.

In the action part, the consequence that fires when the rule matches, we call the `addHardConstraintMatch` of our `scoreHolder` to add a negative hard constraint score. As in our _Easy Score Calculator_, we set the _hard score_ as a negative value, i.e. the amount of missing `cpuPower`.

1. Add the rule to your `.drl` file and save it.

2. Open your `cloudBalancingSolverConfig.xml` file. In order to use the _Drools Score Calculator_, we need to configure the `scoreDrl` our `scoreDirectorFactory`. Configure the `scoreDirectorFactory` as follow and save the file (note that we've commented out the `easyScoreCalculatorClass`):

~~~xml
<scoreDirectorFactory>
    <!--
    <easyScoreCalculatorClass>org.optaplanner.examples.cloudbalancing.optional.score.CloudBalancingEasyScoreCalculator</easyScoreCalculatorClass>
    -->
    <scoreDrl>org/optaplanner/examples/cloudbalancing/solver/cloudBalancingScoreRules.drl</scoreDrl>
  </scoreDirectorFactory>
~~~

3. Run the `CloudBalancingSolverTest` by running a Maven Build. The output should show the test being executed.



~~~
18:57:56.377 [main] INFO org.optaplanner.core.impl.solver.DefaultSolver - Solving ended: time spent (5000), best score (0hard/-132300soft), score calculation speed (8197/sec), phase total (2), environment mode (REPRODUCIBLE).
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 8.146 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

.................
.................

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  13.420 s
[INFO] Finished at: 2020-02-18T18:57:56+01:00
[INFO] ------------------------------------------------------------------------
~~~

Your final `.drl` file should look like this:

~~~
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
~~~

You've successfully implemented your _hard constraints_ and _soft constraints_ in Drools. In the next module of this workshop we will look at the OptaPlanner _Benchmark_. This component allows us to benchmark various `ScoreCalculator` and heuristic algorithm combinations, as well as how they perform against different data-sets. This allows us to both validate our problem space/size, our implementation and the optimal configuration of algorithms for the given planning problem.
