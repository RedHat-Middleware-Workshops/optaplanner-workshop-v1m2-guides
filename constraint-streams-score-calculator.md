# Constraint Streams Score Calculator

In this section you will learn

1. What the OptaPlanner Constraint Streams API is.
2. How to write constraints using Constraint Streams.

In the previous lab we've implemented an _Drools Score Calculator_ in Drools rules. The Drools score calculator is an incremental score calculator that only calculates score delta's for OptaPlanner moves, which makes score calculation fast. However, a disadvantage of Drools can be that one has to learn a new language, namely DRL (Drools Rule Language).

To overcome this challenge, and to accomodate for developers that are like and are used to the Java Streams API, OptaPlanner comes with the _Constraint Streams_ API. Constraint streams are a Functional Programming form of incremental score calculation in plain Java that is easy to read, write and debug. The API should feel familiar if you’ve worked with Java 8 Streams or SQL. Under the covers, constraint streams are transpiled into the Drools Canonical Model, and hence, use the Drools high performant rules execution engine at runtime.

In this module you will write the constraint rules of  our Cloud Balancing problem in Constraint Streams.


## Constraint Streams

As stated, Constraint Streams feel similar to the Java 8 Streams API and follow a functional programming paradigm. Let's write our constraint rules in Constraint Streams.

1. Open CodeReady Workspaces and open your project from the previous lab. You can also import the following project from GitHub and use it as a starting point for this module: [https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-3](https://github.com/RedHat-Middleware-Workshops/optaplanner-workshop-v1m2-labs-step-3)

2. In the `src/main/java/org/optaplannner/examples/cloudbalancing/optional/score` directory, create a new Java class called `CloudBalancingConstraintProvider.java`.

3. To implement our constraints, our class needs to implement the OptaPlanner `ConstraintProvider` API. This API requires us to implement a single method called `defineConstraints`, which returns an array of `Constraint`.

~~~java
public class CloudBalancingConstraintProvider implements ConstraintProvider {

	@Override
	public Constraint[] defineConstraints(ConstraintFactory arg0) {
		return null;
	}
}
~~~


Instead of writing all the `Constraint` definitions directly into a single method, it is common practice to define a method per constraint, that builds that specific constraint. We can then call these methods in our `defineConstraints` method to create the overall constraint set.

The [OptaPlanner documentation](https://docs.optaplanner.org/7.33.0.Final/optaplanner-docs/html_single/index.html#constraintStreams) contains in-depth explanation of _Constraint Streams_. In this lab, we will look at the fundamental concepts.

### Building Blocks

Constraint streams are chains of different operations, called building blocks. Each constraint stream starts with a `from(…​)` building block, which defines the object type on which the `Constriant` needs to evaluate, and is terminated by either a penalty or a reward. The following example shows the simplest possible constraint stream:

~~~java
private Constraint penalizeInitializedShifts(ConstraintFactory factory) {
  return factory.from(CloudProcess.class)
					.penalize("Initialized process", HardSoftScore.ONE_SOFT);
}
~~~

This constraint stream iterates over all known and initialized instances of `CloudComputer`. To include uninitialized instances, replace the from() building block with fromUnfiltered():

~~~java
private Constraint penalizeAllShifts(ConstraintFactory factory) {
        return factory.fromUnfiltered(CloudProcess.class)
                .penalize("A shift", HardSoftScore.ONE_SOFT);
    }
~~~

The purpose of constraint streams is to build up a score for a solution. To do this, every constraint stream must be terminated by a call to either a penalize() or a reward() building block. The penalize() building block makes the score worse and the reward() building block improves the score.

For more detail, please consult the [documentation](https://docs.optaplanner.org/7.33.0.Final/optaplanner-docs/html_single/index.html#constraintStreams).

Let's define our first `Constraint`. As with the other score calculators, the first constraint we want to implement is the hard constraint that verifies that the computer's available `cpuPower` is not exceeded by the processes assigned to it. The implementation of that `Constraint` is the following:


The hard constraint we're going to implement is the constraint that verifies that the computer's available `cpuPower` is not exceeded by the processes assigned to it. The rule that implements that constraint is the following:

~~~java
private Constraint requiredCpuPowerTotal(ConstraintFactory constraintFactory) {
		return constraintFactory.from(CloudProcess.class)
						.groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredCpuPower))
						.filter((computer, requiredCpuPower) -> requiredCpuPower > computer.getCpuPower())
						.penalize("requiredCpuPowerTotal",
										HardSoftScore.ONE_HARD,
										(computer, requiredCpuPower) -> requiredCpuPower - computer.getCpuPower());
}
~~~

Let's explain this `Constraint`. First, we use the `constraintFactory` to start building a new `Constraint`. Next, using the `from(CloudProcess.class)`, we select all the initialized `CloudProcess` _Planning Entities_. Using the `groupBy` construct, we group all the `CloudProcess` entities per `CloudComputer`, and we `sum` their `requiredCpuPower`. This gives as a set of `BiConstrainStream<CloudComputer, Integer>`, which simply defines the a stream of `CloudComputers` and the CPU power required by the processes assigned to them. Next, we `filter` this stream, filtering out only those `CloudComputers` that have less `cpuPower` than the `requiredCpuPower`. And finally, we `penalize` those computers by `(requiredCpuPower - computer.getCpuPower())`, i.e. the amount of missing `cpuPower`.


Note that for the `sum` function syntax to work correctly, we need to statically import it into our `CloudBalancingConstraintProvider` class. Hence, we need to add this import to our class definition:

~~~java
import static org.optaplanner.core.api.score.stream.ConstraintCollectors.sum;
~~~



1. Add the static import and the `requiredCpuPowerTotal` method to the `CloudBalancingConstraintProvider` class.

2. In the `defineConstraints` method, add a call to the `requiredCpuPowerTotal` method. Return the `Constraint` in a `Constraint` array, like so:

~~~java
@Override
public Constraint[] defineConstraints(ConstraintFactory constraintFactory) {
		return new Constraint[]{
						requiredCpuPowerTotal(constraintFactory)
		};
}
~~~

2. Open your `cloudBalancingSolverConfig.xml` file. In order to use the _Constraint Streams Score Calculator_, we need to configure the `ConstraintProvider` class. Configure the `scoreDirectorFactory` as follow and save the file (note that we've commented out the `easyScoreCalculatorClass` and the `cloudBalancingScoreRules.drl`):

~~~xml
<scoreDirectorFactory>
	<!--
	<easyScoreCalculatorClass>org.optaplanner.examples.cloudbalancing.optional.score.CloudBalancingEasyScoreCalculator</easyScoreCalculatorClass>
  <scoreDrl>org/optaplanner/examples/cloudbalancing/solver/cloudBalancingScoreRules.drl</scoreDrl>
	-->
	<constraintProviderClass>org.optaplanner.examples.cloudbalancing.optional.score.CloudBalancingConstraintProvider</constraintProviderClass>
</scoreDirectorFactory>
~~~

3. Run the `CloudBalancingSolverTest` by running a Maven Build. The output should show the test being executed.

~~~
13:06:51.365 [main] INFO org.optaplanner.core.impl.solver.DefaultSolver - Solving ended: time spent (5000), best score (0hard/0soft), score calculation speed (11471/sec), phase total (2), environment mode (REPRODUCIBLE).
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.781 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.769 s
[INFO] Finished at: 2020-02-19T13:06:51+01:00
[INFO] ------------------------------------------------------------------------
~~~

We can see that the test runs successfully. We can also see that we have `0hard/0soft` score. The reason for this is that we've not yet implemented all our constraints, and with the current solution that OptaPlanner finds, the hard constraint we've implemented is not broken.

We can now implement the other 2 hard constraints, the one for `memory` and `networkBandwidth` in the exact same way

4. Implement the other 2 hard constrains in the same way.


    **SOLUTION BELOW** ... Try not to cheat!


5. The full solution of the _hard constraint_ implementation looks like this:

~~~java
@Override
public Constraint[] defineConstraints(ConstraintFactory constraintFactory) {
		return new Constraint[]{
						requiredCpuPowerTotal(constraintFactory),
						requiredMemoryTotal(constraintFactory),
						requiredNetworkBandwidthTotal(constraintFactory)
		};
}

private Constraint requiredCpuPowerTotal(ConstraintFactory constraintFactory) {
		return constraintFactory.from(CloudProcess.class)
						.groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredCpuPower))
						.filter((computer, requiredCpuPower) -> requiredCpuPower > computer.getCpuPower())
						.penalize("requiredCpuPowerTotal",
										HardSoftScore.ONE_HARD,
										(computer, requiredCpuPower) -> requiredCpuPower - computer.getCpuPower());
}

private Constraint requiredMemoryTotal(ConstraintFactory constraintFactory) {
		return constraintFactory.from(CloudProcess.class)
						.groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredMemory))
						.filter((computer, requiredMemory) -> requiredMemory > computer.getMemory())
						.penalize("requiredMemoryTotal",
										HardSoftScore.ONE_HARD,
										(computer, requiredMemory) -> requiredMemory - computer.getMemory());
}

private Constraint requiredNetworkBandwidthTotal(ConstraintFactory constraintFactory) {
		return constraintFactory.from(CloudProcess.class)
						.groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredNetworkBandwidth))
						.filter((computer, requiredNetworkBandwidth) -> requiredNetworkBandwidth > computer.getNetworkBandwidth())
						.penalize("requiredNetworkBandwidthTotal",
										HardSoftScore.ONE_HARD,
										(computer, requiredNetworkBandwidth) -> requiredNetworkBandwidth - computer.getNetworkBandwidth());
}
~~~

With our hard constraints implemented, we can now look at our _soft constraints_. As in the previous labs, the _soft constraints_ are the constraints we want to optimize on. In this case we will only implement one: _the costs_ of our _computers_. (as said before, another possible _soft constraint_ of our use-case could be to reach a 80% resource utilization of our _computers_ to make sure we can safely accomadate for peaks).

We have to _minimize_ the _cost_ of our solution. We have to pay for a `CloudComputer` when at least one `CloudProcess` has been assigned to it.

1. Implement the _soft constraint_ of our planning problem.


    **SOLUTION BELOW** ... Try not to cheat!


2. The soft constraint can be implemented as shown below. Note that the `groupBy(CloudProcess::getComputer)` makes sure that every computer to which one or multiple processes have been assigned is only present once in the stream. This is to make sure that we only penaliza cost of a computer once, even if there have been multiple processes assigned to it:

~~~java
private Constraint computerCost(ConstraintFactory constraintFactory) {
		return constraintFactory.from(CloudProcess.class)
						.groupBy(CloudProcess::getComputer)
						.penalize("computerCost",
										HardSoftScore.ONE_SOFT,
										CloudComputer::getCost);
}
~~~

Your final `CloudBalancingConstraintProvider.java` class should look like this:

~~~java
package org.optaplanner.examples.cloudbalancing.optional.score;

import org.optaplanner.core.api.score.buildin.hardsoft.HardSoftScore;
import org.optaplanner.core.api.score.stream.Constraint;
import org.optaplanner.core.api.score.stream.ConstraintFactory;
import org.optaplanner.core.api.score.stream.ConstraintProvider;
import org.optaplanner.examples.cloudbalancing.domain.CloudComputer;
import org.optaplanner.examples.cloudbalancing.domain.CloudProcess;

import static org.optaplanner.core.api.score.stream.ConstraintCollectors.sum;

public class CloudBalancingConstraintProvider implements ConstraintProvider {

    @Override
    public Constraint[] defineConstraints(ConstraintFactory constraintFactory) {
        return new Constraint[]{
                requiredCpuPowerTotal(constraintFactory),
                requiredMemoryTotal(constraintFactory),
                requiredNetworkBandwidthTotal(constraintFactory),
                computerCost(constraintFactory)
        };
    }

    // ************************************************************************
    // Hard constraints
    // ************************************************************************

    private Constraint requiredCpuPowerTotal(ConstraintFactory constraintFactory) {
        return constraintFactory.from(CloudProcess.class)
                .groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredCpuPower))
                .filter((computer, requiredCpuPower) -> requiredCpuPower > computer.getCpuPower())
                .penalize("requiredCpuPowerTotal",
                        HardSoftScore.ONE_HARD,
                        (computer, requiredCpuPower) -> requiredCpuPower - computer.getCpuPower());
    }

    private Constraint requiredMemoryTotal(ConstraintFactory constraintFactory) {
        return constraintFactory.from(CloudProcess.class)
                .groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredMemory))
                .filter((computer, requiredMemory) -> requiredMemory > computer.getMemory())
                .penalize("requiredMemoryTotal",
                        HardSoftScore.ONE_HARD,
                        (computer, requiredMemory) -> requiredMemory - computer.getMemory());
    }

    private Constraint requiredNetworkBandwidthTotal(ConstraintFactory constraintFactory) {
        return constraintFactory.from(CloudProcess.class)
                .groupBy(CloudProcess::getComputer, sum(CloudProcess::getRequiredNetworkBandwidth))
                .filter((computer, requiredNetworkBandwidth) -> requiredNetworkBandwidth > computer.getNetworkBandwidth())
                .penalize("requiredNetworkBandwidthTotal",
                        HardSoftScore.ONE_HARD,
                        (computer, requiredNetworkBandwidth) -> requiredNetworkBandwidth - computer.getNetworkBandwidth());
    }

    // ************************************************************************
    // Soft constraints
    // ************************************************************************

    private Constraint computerCost(ConstraintFactory constraintFactory) {
        return constraintFactory.from(CloudProcess.class)
                .groupBy(CloudProcess::getComputer)
                .penalize("computerCost",
                        HardSoftScore.ONE_SOFT,
                        CloudComputer::getCost);
    }

}
~~~

We can now test our solution:

1. With the constraint rules fully implemented, run another Maven build that will execute the tests.

2. If the test executes correctly, you should see that the Solver will end after 5 seconds (5000 milliseconds), with (in this example run) a best score of `(0hard/-133980soft)`:

~~~~
15:00:43.179 [main] INFO org.optaplanner.core.impl.solver.DefaultSolver - Solving ended: time spent (5000), best score (0hard/-133980soft), score calculation speed (6653/sec), phase total (2), environment mode (REPRODUCIBLE).
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 5.741 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0

.......
.......

[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  8.774 s
[INFO] Finished at: 2020-02-19T15:00:43+01:00
[INFO] ------------------------------------------------------------------------
~~~

You've successfully implemented your _hard constraints_ and _soft constraints_ in Constraint Stream. In the next module of this workshop we will look at the OptaPlanner _Benchmark_. This component allows us to benchmark various `ScoreCalculator` and heuristic algorithm combinations, as well as how they perform against different data-sets. This allows us to both validate our problem space/size, our implementation and the optimal configuration of algorithms for the given planning problem.
