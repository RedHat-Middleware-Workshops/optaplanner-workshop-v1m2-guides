
# Drools Score Calculator

In this section you will learn

1. How to integrate OptaPlanner with storage solutions.
2. The OptaPlanner SolutionFileIO interface to load planning problems from persistent storage.

OptaPlanner is written in Java, based on open standards, and the domain models of planning problems are, as we've seen in the previous lab, based on Java POJOs. As such, OptaPlanner does not enforce any type of storage or persistence framework. OptaPlanner can integrate with any form of storage type and format, as long as there is a Java client or framework available. For examples

- Java Persistence API (JPA) can be used to interact with relational databases
- XStream, JABX2, etc. can be used to store to and read from XML files.
- Integration with Red Hat DataGrid can be accomplished via the DataGrid HotRod client.

Being able to import, and also export, planning problems and planning solutions in a proper way is extremely important when creating an OptaPlanner application/solution. Being able to show the business the output of a planning solution, and being able to properly visualize the solution, for example in an Excel file will give your business users insight in your solution, and enables them to give feedback about the planning solutions created by your OptaPlanner implementation. In other words, it enables your business users to be part of the development process and stay in control.


## SolutionFileIO

In this lab we will read our planning problem data from XML files. OptaPlanner provides an interface called `SolutionFileIO`, which provides methods to read and write planning solutions from and to persistent storage. This interface is, among other things, used by the OptaPlanner Benchmarker to load unsolved planning problems from storage and to optionally write the best solution back to disk. Although it is not mandatory to use this interface in your OptaPlanner application (i.e. your production application), it provides a convenient abstraction between your planning solution class and storage.

We will use XStream as our XML serialization and deserialization library. OptaPlanner provides and out-of-the-box `SolutionFileIO` implementation that uses the XStream library to serialize to and from XML, the `XStreamSolutionFileIO`. We will use this out-of-the-box implementation to read our planning problem  from XML files (these files will be provided).


1. Create a new class with the name `AbstractPersistable` in the same package of our domain model and give it the following implemenation:
```
package org.optaplanner.examples.cloudbalancing.domain;

import org.optaplanner.core.api.domain.lookup.PlanningId;

/**
 * AbstractPersistable
 */
public abstract class AbstractPersistable {

    private Long id;

    protected AbstractPersistable() {
    }

    protected AbstractPersistable(Long id) {
        this.id = id;
    }

    @PlanningId
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

}
```

2. Have our 3 domain model classes, `CloudBalance`, `CloudProcess` and `CloudComputer` extend from our `AbstractPersistable` class, e.g.:
```
public class CloudBalance extends AbstractPersistable {
```

3. Change the constructors of these classes to accept an id, and have them cal their abstract super class. I.e.:

```
public CloudBalance(Long id, List<CloudComputer> computerList, List<CloudProcess> processList) {
  super(id);
  this.computerList = computerList;
  this.processList = processList;
}
```

```
public CloudComputer(long id, int cpuPower, int memory, int networkBandwidth, int cost) {
  super(id);
  this.cpuPower = cpuPower;
  this.memory = memory;
  this.networkBandwidth = networkBandwidth;
  this.cost = cost;
}
```

```
public CloudProcess(long id, int requiredCpuPower, int requiredMemory, int requiredNetworkBandwidth) {
  super(id);
  this.requiredCpuPower = requiredCpuPower;
  this.requiredMemory = requiredMemory;
  this.requiredNetworkBandwidth = requiredNetworkBandwidth;
}
```

4. Add an `@XStreamAlias` annotation to our 3 domain model classes:

```
@PlanningSolution
@XStreamAlias("CloudBalance")
public class CloudBalance extends AbstractPersistable {
```

```
@XStreamAlias("CloudComputer")
public class CloudComputer extends AbstractPersistable {
```

```
@PlanningEntity
@XStreamAlias("CloudProcess")
public class CloudProcess extends AbstractPersistable {
```

5. Create a _Repository_ class that is responsible for loading the planning problem from the XML files. Give the class the name `CloudBalanceRepository` and store it in the package `org.optaplanner.examples.cloudbalancing.persistence`. Give it the following implementation:

```
package org.optaplanner.examples.cloudbalancing.persistence;

import java.io.File;

import org.optaplanner.examples.cloudbalancing.domain.CloudBalance;
import org.optaplanner.examples.cloudbalancing.domain.CloudComputer;
import org.optaplanner.examples.cloudbalancing.domain.CloudProcess;
import org.optaplanner.persistence.xstream.impl.domain.solution.XStreamSolutionFileIO;

public class CloudBalanceRepository {

    public static CloudBalance load(File inputSolutionFile) {
        XStreamSolutionFileIO<CloudBalance> solutionFileIO = new XStreamSolutionFileIO<>(CloudBalance.class,
                CloudProcess.class, CloudComputer.class);
        return solutionFileIO.read(inputSolutionFile);
    }
}
```

6. Run a Maven Build to verify that the project compiles correctly.


With the code in place to load our planning problem, we now need to verify whether we can correctly load a CloudBalance dataset from the filesystem. We will therefore write a unit-test. Because OptaPlanner is a Java library, and fully supports Maven and JUnit, we can use our standard Java unit-testing skills to write tests for our OptaPlanner project.

1. Create a new folder in the root of your project called `data/cloudbalancing/unsolved`.

2. In this folder, create a file called `4computers-12processes.xml`, and add the following content:

```
<CloudBalance id="1">
  <id>0</id>
  <computerList id="2">
    <CloudComputer id="3">
      <id>0</id>
      <cpuPower>24</cpuPower>
      <memory>96</memory>
      <networkBandwidth>16</networkBandwidth>
      <cost>4800</cost>
    </CloudComputer>
    <CloudComputer id="4">
      <id>1</id>
      <cpuPower>6</cpuPower>
      <memory>4</memory>
      <networkBandwidth>6</networkBandwidth>
      <cost>660</cost>
    </CloudComputer>
    <CloudComputer id="5">
      <id>2</id>
      <cpuPower>6</cpuPower>
      <memory>16</memory>
      <networkBandwidth>4</networkBandwidth>
      <cost>680</cost>
    </CloudComputer>
    <CloudComputer id="6">
      <id>3</id>
      <cpuPower>8</cpuPower>
      <memory>32</memory>
      <networkBandwidth>12</networkBandwidth>
      <cost>1270</cost>
    </CloudComputer>
  </computerList>
  <processList id="7">
    <CloudProcess id="8">
      <id>0</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="9">
      <id>1</id>
      <requiredCpuPower>3</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="10">
      <id>2</id>
      <requiredCpuPower>11</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="11">
      <id>3</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="12">
      <id>4</id>
      <requiredCpuPower>5</requiredCpuPower>
      <requiredMemory>4</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="13">
      <id>5</id>
      <requiredCpuPower>5</requiredCpuPower>
      <requiredMemory>2</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="14">
      <id>6</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>2</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="15">
      <id>7</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>7</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="16">
      <id>8</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>5</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="17">
      <id>9</id>
      <requiredCpuPower>7</requiredCpuPower>
      <requiredMemory>5</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="18">
      <id>10</id>
      <requiredCpuPower>3</requiredCpuPower>
      <requiredMemory>3</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
    <CloudProcess id="19">
      <id>11</id>
      <requiredCpuPower>1</requiredCpuPower>
      <requiredMemory>1</requiredMemory>
      <requiredNetworkBandwidth>1</requiredNetworkBandwidth>
    </CloudProcess>
  </processList>
</CloudBalance>
```

3. Add the following dependencies to the `pom.xml` file of your project to add JUnit functionality:
```
<dependency>
  <groupId>junit</groupId>
  <artifactId>junit</artifactId>
  <version>4.12</version>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.hamcrest</groupId>
  <artifactId>java-hamcrest</artifactId>
  <version>2.0.0.0</version>
  <scope>test</scope>
</dependency>
```

4. Create the following folder in the root of your project: `src/test/java`

5. In this new folder create the following package: `org.optaplanner.examples.cloudbalancing.persistence`

6. In this package, create a new Java class with the following name: `CloudBalanceRepositoryTest`. Implement the class as follows:
```
package org.optaplanner.examples.cloudbalancing.persistence;

import static org.hamcrest.MatcherAssert.assertThat;
import static org.hamcrest.CoreMatchers.*;

import java.io.File;

import org.junit.Test;
import org.optaplanner.examples.cloudbalancing.domain.CloudBalance;


/**
 * CloudBalanceRepositoryTest
 */
public class CloudBalanceRepositoryTest {

    @Test
    public void testLoadCloudBalance() {
        CloudBalanceRepository repository = new CloudBalanceRepository();
        File inputFile = new File("data/cloudbalancing/unsolved/4computers-12processes.xml");
        CloudBalance cloudBalance = repository.loadCloudBalance(inputFile);

        assertThat(4, is(cloudBalance.getComputerList().size()));
        assertThat(12, is(cloudBalance.getProcessList().size()));
    }

}
```

7. Run the Unit test by running a Maven Build. The output should show the test being execute:
```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.optaplanner.examples.cloudbalancing.persistence.CloudBalanceRepositoryTest
Tests run: 1, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 1.595 sec

Results :

Tests run: 1, Failures: 0, Errors: 0, Skipped: 0
```

Now that we have the ability to load planning problem data, we can go the next step of our project: configuring the solver!
