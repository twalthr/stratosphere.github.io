--- 
layout: inner_docs_v04
title: Java Programming Guide
sublinks:
  - {anchor: "basic", title: "Basic Programs"}
  - {anchor: "advanced", title: "Advanced Programs"}
  - {anchor: "pact_record", title: "PACT Records"}
  - {anchor: "io_formats", title: "Input and Output Formats"}
  - {anchor: "testing", title: "Testing"}
  - {anchor: "package", title: "Packaging"}
---

<section id="basic">
Writing Stratosphere Programs in Java
=====================================

This guide explains how to develop Stratosphere programs with the Java programming interface.
It assumes you are familiar with the general concepts of Stratosphere's [Programming Model](pmodel.html "Programming Model").
We recommend to learn about the basic concepts first, before continuing with the Java or the [Scala](scala.html "Scala Programming Guide") programming guide.
The guide starts with a step-by-step instruction for a simple WordCount example job and delves into more details later.

Implement a PACT Program
------------------------

The implementation of PACT programs is done in Java. This section
describes the required components, the interfaces and abstract classes
to implement and extend, and how to assemble them to a valid PACT
program.   
 A set of example PACT programs with links to their source code can be
found in the
[Examples](pactexamples.html "pactexamples")
section.

### Code Dependencies

PACT programs depend on the *pact-common.jar* and transitively on its
dependencies. The easiest way to resolve all dependencies is to include
the public Stratosphere Maven repository into your *pom.xml* file and
declare the *pact-common* artifact as a dependency.   
 The following code snippet shows the required sections in the *pom.xml*
of your Maven project:

    <project>
      ...
      <repositories>
        <repository>
          <id>stratosphere.eu</id>
          <url>http://www.stratosphere.eu/maven2/</url>
        </repository>
        ...
      </repositories>
      ...
      <dependencies>
        <dependency>
          <groupId>eu.stratosphere</groupId>
          <artifactId>pact-common</artifactId>
          <version>0.2</version>
          <type>jar</type>
          <scope>compile</scope>
        </dependency>
        ...
      </dependencies>
    ...
    </project>

### Basic Components and Structure of a PACT Program

A PACT program basically consist of four components:

-   **Data types**: Primitive types (Integer, String, Double, etc.) are
    provided in
    *[eu.stratosphere.pact.common.type.base](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/base "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/base")*.
      
     More complex types must implement the interfaces
    *[eu.stratosphere.pact.common.api.common.type.Key](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Key.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Key.java")*
    or
    *[eu.stratosphere.pact.common.api.common.type.Value](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Value.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Value.java")*.
      
       
     **Important:** All data types must implement a default constructor
    (no arguments expected)!   

    *[compareTo()](http://download.oracle.com/javase/6/docs/api/java/lang/Comparable.html#compareTo%28T%29 "http://download.oracle.com/javase/6/docs/api/java/lang/Comparable.html#compareTo%28T%29")*,
    *[equals()](http://download.oracle.com/javase/6/docs/api/java/lang/Object.html#equals%28java.lang.Object%29 "http://download.oracle.com/javase/6/docs/api/java/lang/Object.html#equals%28java.lang.Object%29")*,
    and
    *[hashCode()](http://download.oracle.com/javase/6/docs/api/java/lang/Object.html#hashCode%28%29 "http://download.oracle.com/javase/6/docs/api/java/lang/Object.html#hashCode%28%29")*.
    Jobs that use key types with incorrect implementations of these
    functions might behave unexpectedly and return indeterministic
    results!

-   **Data formats**: Data formats generate the key-value pairs
    processed by the Pact Program and consume the result records,
    typically by reading them from files and writing them back to files.
    They represent the user-defined functionality of DataSources and
    DataSinks. For reading and writing pairs, two corresponding classes
    have to be implemented that extend
    *[eu.stratosphere.pact.common.generic.io.InputFormat](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/InputFormat.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/InputFormat.java")*
    and
    *[eu.stratosphere.pact.common.generic.io.OutputFormat](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/OutputFormat.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/OutputFormat.java")*.
      
       
     For the common case of reading text records from files and writing
    text records back, some convenience formats are provided:
    *[eu.stratosphere.pact.common.io.TextInputFormat](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/TextInputFormat.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/TextInputFormat.java")*
    and
    *[eu.stratosphere.pact.common.io.TextOutputFormat](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/TextOutputFormat.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/TextOutputFormat.java")*.
    The text input and output format work with a customizable delimiter
    character (default is '\\n').   
       
     For more details on the Input- and Output Formats, such as non-file
    data sources, binary formats, or custom partitions, please refer to
    the detail section on [Input And Output
    Formats](dataformats.html "dataformats").

\* **Stub implementations**: Your job logic goes into PACT stub
implementations. Stubs are templates for first-order user functions that
are executed in parallel. Stub implementations extend the stubs found in
*[eu.stratosphere.pact.common.stub](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/stub "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/stub")*.
You need to extend the stub that corresponds to the [Input
Contract](pactpm#input_contracts "pactpm")
for that specific user function. For example, if your function should
use a `Match` contract, than you need to extend the `MatchStub`.

\* **Plan construction**: A class that implements the interface
*[eu.stratosphere.pact.common.plan.PlanAssembler](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/plan/PlanAssembler.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/plan/PlanAssembler.java")*
provides PACT job plans. The method *getPlan(String …)* constructs the
plan of a [PACT
Job](pactpm.html "pactpm").
The plan consists of connected [Input
Contracts](pactpm#input_contracts "pactpm")
(found in
*[eu.stratosphere.pact.common.contract](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract")*).
Each [Input
Contract](pactpm#input_contracts "pactpm")
has an own contract class. A contract is initialized with the
corresponding PACT stub implementation. Contracts are assembled to a
contract graph with their *setInput()*
(*setFirstInput()**/setSecondInput()*) methods. A plan
(*[eu.stratosphere.pact.common.plan.Plan](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/plan/Plan.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/plan/Plan.java")*)
is generated and initialized with all DataSinks of the assembled
contract graph. Finally, the plan is returned by the *getPlan(String …)*
method. By default, all PACTs are executed single-threaded (only one
instance is started). To run a PACT in parallel, you must specify a
degree of parallelization at the PACT contract using the method
*setDegreeOfParallelsm(int dop);*.   

### Reading Data Sources

Nephele reads its data from file systems, in the distributed setup
typically the [Hadoop Distributed Filesystem
(HDFS)](http://hadoop.apache.org/hdfs "http://hadoop.apache.org/hdfs"),
in the local setup typically the local file system. Data is fed into a
PACT program with DataSourceContracts. A DataSourceContract is
instantiated during plan construction with a path (HDFS path or local
path) to the data it will provide. The path may point to a single file
or to a directory.   
 If the data source is a directory, all (non-recursively) contained
files are read and fed into the PACT program.

### Writing Result Data

Nephele writes result data to a file system. A PACT program emits result
data with a DataSinkContract. During plan construction, a
DataSinkContract is instantiated with a HDFS path (or local path) to
which the result data is written. Result data can be written either as a
single file or as a collection of files. If the data shall be written to
a single file, the provided path must point to file.   
 **Attention**: In case the file already exists, it will be overwritten!
Result data will be written to a single file, if the DataSink task runs
with a degree-of-parallelism of one. In that case, only a single
parallel instance of the DataSink is created!   
 If the output of the result should be done in parallel, the output path
must be an existing directory or a directory is created. This is not
possible if the target path is an existing file. Each parallel instance
of the DataSink task will write a file into the output directory. The
name of the file is the number of the parallel instance of the DataSink
subtask.

### Reduce Combiners

The PACT Programming Model does also support optional combiners for
Reduce stubs.   
 Combiners are implemented by overriding the `combine()` method of the
corresponding Reduce stub
(*[eu.stratosphere.pact.common.stub.ReduceStub](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/stub/ReduceStub.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/stub/ReduceStub.java")*).
To notify the optimizer of the *combine()* implementation the Reduce
stub must be annotated with the *Combinable* annotation
(*[eu.stratosphere.pact.common.contract.ReduceContract.Combinable](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract/ReduceContract.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract/ReduceContract.java")*).
  
 **Attention:** The combiner will not be used, if the annotation is
missing. Annotating a Reduce stub although *combine()* was not
implemented may cause incorrect results, unless the combine function is
identical to the reduce function!

### User Code Annotations

[Stub
Annotations](pactpm#user_code_annotations "pactpm")
are realized as Java annotations and defined in
*[eu.stratosphere.pact.common.stubs.StubAnnotation](https://github.com/stratosphere/stratosphere/tree/master/pact-common/src/main/java/eu/stratosphere/pact/common/stubs/StubAnnotation.java "https://github.com/stratosphere/stratosphere/tree/master/pact-common/src/main/java/eu/stratosphere/pact/common/stubs/StubAnnotation.java")*.
  

There are two ways to attach the annotations.

1.  Attach the annotation as a class annotation directly at the stub
    class.
2.  Set the annotation at the contract object using the method
    *setStubAnnotation(…)*.

DataSources can also be annotated. Because annotations of DataSources
will mostly depend on the input data, they cannot be attached to
DataSource stub classes, i.e. *InputFormat*s. Instead, they can only be
attached to DataSource contracts which include the definition of the
input location using the *setStubAnnotation(…)* method.

### Configuration of Stubs

To improve the re-usability of stub implementations, stubs can be
parametrized, e.g. to have configurable filter values instead of
hard-coding them into the stub implementation.   

Stubs can be configured with custom parameters by overriding the method
`configure(Configuration parameters)` in your stub implementation. The
PACT framework calls *configure()* with a *Configuration* object that
holds all parameters which were passed to the stub implementation during
plan construction. To set stub parameters, use the methods
*setStubParameter(String key, String value)*, *setStubParameter(String
key, int value)*, and *setStubParameter(String key, boolean value)* of
the contract object that contains the stub class to parametrize.   
 See the `FilterO` Map stub of the [TPCHQuery3 Example
Program](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-examples/src/main/java/eu/stratosphere/pact/example/relational/TPCHQuery3.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-examples/src/main/java/eu/stratosphere/pact/example/relational/TPCHQuery3.java")
to learn how to configure Stubs.

### Compiler Hints

During optimization, the compiler estimates costs for all generated
execution plans. The costs of a plan are derived from size estimations
of intermediate results. Due to the nature of black-box stub code, the
optimizer can only make rough conservative estimations. Later stages in
the plan often have no estimates at all. In this case the compiler
chooses robust default strategies that behave well for large result
sizes. The accuracy of these estimates can be significantly improved by
supplying compiler hints to the optimizer.   

Compiler hints can be given to the optimizer by setting them at the
contracts during plan construction. This is done by first fetching the
contract's
[eu.stratosphere.pact.common.contract.CompilerHints](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract/CompilerHints.java "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/contract/CompilerHints.java")
object:

    Contract.getCompilerHints()

Compiler hints can then be set on the CompilerHints object. The
following compiler hints are currently supported:

-   **Distinct Count**: Specifies the number of distinct values for a
    set of fields. Set by:

<!-- -->

    CompilerHints.setDistinctCount(FieldSet fields, long card)

Cardinality must be \> 0.

-   **Average Number of Records per Distinct Fields**: Specifies the
    average number of records that share the same values for the
    specified set of fields. Set by:

<!-- -->

    CompilerHints.setAvgNumRecordsPerDistinctFields(FieldSet fields, float avgNumRecords)

Average number of records must be \> 0.

-   **Unique Fields**: Specifies that a set of fields has unique values.
    Set by:

<!-- -->

    CompilerHints.setUniqueField(FieldSet uniqueFieldSet)

-   **Average Bytes per Record**: Specifies the average width of emitted
    records in bytes. The width is the sum of the sizes of all
    serialized fields. Set by:

<!-- -->

    CompilerHints.setAvgBytesPerRecord(float avgBytes)

Average bytes per record must be \>= 0.

-   **Average Number of Records Emitted Per Stub Call**: Specifies the
    average number of emitted records per stub call. If not set, “1.0f”
    is used as default value. Set by:

<!-- -->

    CompilerHints.setAvgRecordsEmittedPerStubCall(float avgRecordsEmittedPerStubCall)

Average records emitted per stub call must be \>= 0.

### Accumulators and Counters
Accumulators are simple constructs with an add operation, that can be called from your stub code, and a final (accumulated) result, which is available after the job ended. The most straightforward accumulator is a counter: You can increment it, using the ```Accumulator.add(V value)``` method, and at the end of the job Stratosphere will sum up (merge) all partial results and send the result to the client. Since accumulators are very easy to use, they can be useful during debugging or if you quickly want to find out more about your data.

Stratosphere currently has the following built-in accumulators. Each of them implements the [Accumulator](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/Accumulator.java) interface.

- [__IntCounter__](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/IntCounter.java), [__LongCounter__](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/LongCounter.java) and [__DoubleCounter__](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/DoubleCounter.java): See below for an example using a counter.
- [__Histogram__](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/Histogram.java): A histogram implementation for a discrete number of bins. Internally it is just a map from Integer to Integer. You can use this to compute distributions of values, e.g. the distribution of words-per-line for a word count program.

__How to use accumulators:__

First you have to create an accumulator object (here a counter) in the stub where you want to use it.

    private IntCounter numLines = new IntCounter();

Second you have to register the accumulator object, typically in the ```open()``` method of the stub. Here you also define the name.

    getRuntimeContext().addAccumulator("num-lines", this.numLines);

You can now use the accumulator anywhere in the stub, including in the ```open()``` and ```close()``` methods.

    this.numLines.add(1);

The overall result will be stored in the ```JobExecutionResult``` object which is returned when running a job using the Java API (currently this only works if the execution waits for the completion of the job).

    myJobExecutionResult.getAccumulatorResult("num-lines")

All accumulators share a single namespace per job. Thus you can use the same accumulator in different stubs of your job. Stratosphere will internally merge all accumulators with the same name.

Please look at the [WordCountAccumulator example](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-examples/src/main/java/eu/stratosphere/pact/example/wordcount/WordCountAccumulators.java) for a complete example.

A note on accumulators and iterations: Currently the result of accumulators is only available after the overall job ended. We plan to make the result of the previous iteration available in the next iteration.

__Custom accumulators:__

To implement your own accumulator you simply have to write your implementation of the Accumulator interface. Please look at the [WordCountAccumulator example](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-examples/src/main/java/eu/stratosphere/pact/example/wordcount/WordCountAccumulators.java) for an example. Feel free to create a pull request if you think your custom accumulator should be shipped with Stratosphere.

You have the choice to implement either [Accumulator](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/Accumulator.java) or [SimpleAccumulator](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/services/accumulators/SimpleAccumulator.java). ```Accumulator<V,R>``` is most flexible: It defines a type ```V``` for the value to add, and a result type ```R``` for the final result. E.g. for a histogram, ```V``` is a number and ```R``` is a histogram. ```SimpleAccumulator``` is for the cases where both types are the same, e.g. for counters.

Package a PACT Program
----------------------

To run a PACT program on a Nephele system all required Java classes must
be available to all participating Nephele
[TaskManagers](taskmanager.html "taskmanager").
Therefore, all Java classes that are part of the PACT program (stub
implementations, data types, data formats, external classes) must be
packaged into a jar file. To make the jar file an executable PACT
program, it must also contain a class that implements the
*PlanAssembler* interface. In addition, you must register in the jar
file's manifest using the attribute *Pact-Assembler-Class*. See
[Executing Pact
Programs](executepactprogram.html "executepactprogram")
for details.

</section>

<section id="advanced">
Best Practices of Pact Programming
==================================

This page provides a brief overview of insights and strategies that we
learned during programming with PACTs to achieve better performance.

Consider the Number of Stub Calls
---------------------------------

-   You should consider that your stub methods usually are called
    extremely often. A *map(…)* function is called once per processed
    record.
-   As a (very) general guideline, implement your methods with the least
    possible overhead you can think of. Move any invariant code out of
    the function. The *open(Configuration)* and *close()* methods are
    places for initialization and tear-down logic.

Object Instantiations and Modifications
---------------------------------------

-   Avoid instantiation of objects as much as possible. Recall that many
    method calls happen in the course of a program. Even though the
    object instantiation itself is pretty cheap in Java, it will
    eventually cause many short-lived objects to be created. The tracing
    of a plethora of dead objects as well as the eventual reclamation of
    their memory space by the garbage collector has a cost that is often
    underestimated. Careless object instantiation is a major cause for
    bad performance.
-   Reuse object instances. Once an data type object and a record are
    emitted to the Collector, their instances are safe to be reused,
    because the data will have been serialized at this point. The
    following code illustrates that at the example of the line
    tokenizing mapper in the WordCount example. The map function reuses
    all object instances. The PactString containing the next token is
    reused after it has been emitted to the Collector.

        public static class TokenizeLine extends MapStub
        {
            // initialize reusable mutable objects
            private final PactRecord outputRecord = new PactRecord();
            private final PactString word = new PactString();
            private final PactInteger one = new PactInteger(1);
         
            private final AsciiUtils.WhitespaceTokenizer tokenizer = new AsciiUtils.WhitespaceTokenizer();
         
            @Override
            public void map(PactRecord record, Collector<PactRecord> collector){
                // get the first field (as type PactString) from the record
                PactString line = record.getField(0, PactString.class);
         
                // normalize the line
                AsciiUtils.replaceNonWordChars(line, ' ');
                AsciiUtils.toLowerCase(line);
         
                // tokenize the line
                this.tokenizer.setStringToTokenize(line);
                while (tokenizer.next(this.word)) {
                    // we emit a (word, 1) pair 
                    this.outputRecord.setField(0, this.word);
                    this.outputRecord.setField(1, this.one);
                    collector.collect(this.outputRecord);
                }
            }
        }

Implement your own efficient Data Types
---------------------------------------

-   Use Java's primitive types (int, double, boolean, char, …) as much
    as possible when implementing your own data types and try to avoid
    using objects.
-   The provided data types are fine for rapid prototyping, but you
    should avoid composing them to more complex data types and rather
    use primitives, if you aim for performance.
-   The previous statement holds especially for the provided collection
    types, such as *PactList* and *PactMap*.
-   Pay attention to efficient serialization and deserialization
    methods. Serialization and deserialization occur very often,
    whenever network is involved, data is sorted or hashed, or
    checkpoints are created.
-   If your data type acts as a key, consider also implementing the
    [NormalizableKey](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/NormalizableKey.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/NormalizableKey.java")
    interface. That allows the system to operate more efficiently during
    sorting.

Implement State-Free Stub Methods
---------------------------------

-   Stub methods are executed in parallel. On the same physical node (or
    Virtual Machine), all stubs run in the same JVM in concurrent
    threads.
-   Ideally, stub methods should be implemented state-free, following
    the principle of functional programming. All invocations of the
    method stub should be completely independent from one another and no
    shared variables are used. If state needs to be held, use instance
    variables. The same single stub object is used throughout the life
    of a single parallel thread, so the state within a thread can be
    maintained through instance variables.
-   Avoid class (*static*) variables. Their scope is within the JVM of
    the physical instance (or VM). Even when specifying a default
    intra-node parallelism of one, more parallel instances of your stub
    may be scheduled to the same JVM, because TaskManagers on large
    machines may be assigned multiple smaller virtual instances. Even if
    the statically referenced objects are thread-safe, the performed
    synchronization may cause significant overhead through lock
    congestion.
-   Using *static* objects, can cause unpredictable behavior, if the
    statically referenced objects (or any methods of those objects) are
    not thread-safe. In the worst case, that can lead to exceptions and
    cause the job to fail repeatedly.
-   Using state-free or thread-safe processing objects (such as
    Formatter, Parser, and so on) in your stub methods is fine. These
    objects should be created during instantiation (within the
    constructor) or using *configure()* method of the stub.

Use User Code Annotations and Compiler Hints
--------------------------------------------

-   *User Code Annotations* give the optimizer information about the
    behavior of your stub methods. The optimizer can frequently exploit
    that information to generate execution plans that avoid expensive
    operations such as data shipping and sorting. Be sure to attach only
    valid annotations to your function. An incorrect annotation that the
    function violates may cause incorrect results. Refer to the [list of
    annotations](pactpm#user_code_annotations "pactpm")
    for an overview of available annotations, and to the [programming
    tutorial](writepactprogram.html "writepactprogram")
    to learn how to attach them. Our
    [examples](pactexamples.html "pactexamples")
    also show how to correctly use them. We are currenlty prototyping
    static code analysis methods to infer those annotations
    automatically.
-   Give the optimizer as much information as possible.
    Average-records-per-stub-call, average-record-width,
    key-cardinality, and values-per-key are important numbers for the
    optimizer that help generating better estimates consequently saving
    costs. See the [Compiler
    Hints](writepactprogram#compiler_hints "writepactprogram")
    section for the supported compiler hints and how to set them.


</section>

<section id="pact_record">
Pact Record
===========

The
[PactRecord](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/PactRecord.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/PactRecord.java")
is the basic data type that flows in the PACT data flow programs. It
replaces the Key/Value pair in MapReduce and is more generic and
flexible.

The Pact Record can be thought of as a schema free tuple that may
contain arbitrary fields. The fields may be any data type that
implements the
[Value](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Value.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/type/Value.java")
interface, which describes how the type is serialized and deserialized.

The interpretation of a field is up to the user code. Whenever a field
in the record is accessed, the user code must specify the position of
the field to access and the data type as which to read it. A simple
example of a filtering map that accesses a single integer field is given
below:

    @Override
    public void map(PactRecord record, Collector<PactRecord> out) throws Exception {
     
        PactInteger theInt = record.getField(1, PactInteger.class);
        if (theInt.getValue() > RANKFILTER) {
            out.collect(record);
        }
    }

Lazy DeSerialization
--------------------

When the record is read from a stream, all fields are typically in a
binary representation. Fields are lazily deserialized when accessed and
then cached. Any modifications to the record are also cached and lazily
serialized when needed (for example for network communication). It may
happen that the record never serializes data, for example when it is
piped to a successive task which filters it.

In the above example of the filtering map, the functions parameter
record is re-emitted. Because the record is unmodified, it can use its
original binary signature and will not invoke any data-types
serialization code. This is rather efficient. For records where only a
subset of the fields are changed, the unchanged ones are kept in their
original binary representation, avoiding serialization and
deserialization effort.

Mutable Objects
---------------

**Warning**: It is important to treat PactRecords as mutable objects.
They were specifically designed for reusing objects and perform quite a
bit of caching and lazy work to increase performance. That makes them
rather heavy-weight objects which put a lot of pressure on the garbage
collector, if they are not reused.

PactRecords reuse internal data structures for performance reasons. You
should refrain from creating instances in the user function. If the
record is unchanged (or only modified at some positions) it is best to
emit the instance that was supplied in the function call, as in the
above example of the filtering map function. If you emit records whose
contents is very much unrelated to the received records, create a single
instance outside the function and reuse that one. It is save to reuse
any PactRecord object once it has been handed to the Collector. The code
below (from the WordCount example) illustrates that:

    public static class TokenizeLine extends MapStub
    {
        // initialize reusable mutable objects
        private final PactRecord outputRecord = new PactRecord();
        private final PactString word = new PactString();
        private final PactInteger one = new PactInteger(1);
     
        private final AsciiUtils.WhitespaceTokenizer tokenizer = new AsciiUtils.WhitespaceTokenizer();
     
        @Override
        public void map(PactRecord record, Collector<PactRecord> collector){
            // get the first field (as type PactString) from the record
            PactString line = record.getField(0, PactString.class);
     
            // normalize the line
            AsciiUtils.replaceNonWordChars(line, ' ');
            AsciiUtils.toLowerCase(line);
     
            // tokenize the line
            this.tokenizer.setStringToTokenize(line);
            while (tokenizer.next(this.word)) {
                // we emit a (word, 1) pair 
                this.outputRecord.setField(0, this.word);
                this.outputRecord.setField(1, this.one);
                collector.collect(this.outputRecord);
            }
        }
    }

Pact Records are also reused by the runtime. Whenever a function is
invoked multiple times, it is typically supplied with the same
PactRecord Object, with different contents each time. This means that
you cannot keep references to the object across function invocations,
because the object's contents will be changed. If you need to keep a
copy of the entire record (which is typically an indication for bad
code), create a copy of the record via *PactRecord\#createCopy()*. In
general, it is better to keep a copy of some individual fields,
preferably as primitive types.

The following code will cause the list to contain multiple times the
same reference:

    public void reduce(Iterator<PactRecord> records, Collector<PactRecord> out) {
        List<PactRecord> list = new ArrayList<PactRecord>();
        while (records.hasNext()) {
            list.add(records.next());
        }
    }

A correct and also more efficient version would be the version below.
The code saves any unnecessary copying and, depending on a clever
implementation of the *IntList*, code might run without ever
instantiating an object inside the reduce function.

    private final IntList list = new IntList();
     
    @Override
    public void reduce(Iterator<PactRecord> records, Collector<PactRecord> out) {
        list.clear();
        while (records.hasNext()) {
            PactInteger i = records.next().getField(1, PactInteger.class);
            list.add(i.getValue);
        }
     
        // do something with the list
    }

Cached Fields
-------------

To save de-serialization effort, the Pact Record caches deserialized
fields. That means a successive access to the fields (possibly in a
succeeding function) may be a lot cheaper. The implication of that
caching mechanism is that you must be careful modifying instances that
you obtained from the record. In any case, any object may be reused
across different function calls or records obtained from an iterator.
The following example shows a possible error source:

    public void map(PactRecord record, Collector<PactRecord> out) {
        PactInteger i = record.getField(0, PactInteger.class); // deserialized the value and cached the object instance
        System.out.println(i); // prints the original value
        i.setValue(42); // sets the value to the object that is also cached

        PactInteger k = record.getField(1, PactInteger.class); // returns the cached object
        System.out.println(k); // prints the 42
    }

Sparsity
--------

The record may have null fields between fields that are actually set. If
a certain field is set, all yet unset fields before that field are
considered *null*. IN the following example, fields 2 and 5 are set,
while fields 1, 3, 4 are *null*.

    PactRecord rec = new PactRecord();
    rec.setField(2, new PactInteger(43290));
    rec.setField(5, new PactString("some text...");

Null fields are sparsely encoded with bit masks and occupy rather little
space. They allow to write code that moves little data between the
fields.

</section>

<section id="io_formats">
Input and Output Formats
========================

Input- and Output Formats are the sources and sinks of the records in
your Pact Program dataflow and are hence the user defined functionality
of the DataSources and DataSinks in your Pact Program.

Input Formats
=============

Input formats are a generic description of how your records enter the
dataflow in a parallel fashion. All input formats implement the
interface
*[eu.stratosphere.pact.common.generic.io.InputFormat](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/InputFormat.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/generic/io/InputFormat.java")*.
The input format describes two aspects:

1.  How to partition an input into *splits* that are processed in
    parallel.
2.  How to generate records from a split.

The input formats base classes are very generic to enable a generic type
runtime. However, all subclasses that programmers typically extend are
typed to PactRecord, so we will use the typed signatures in the
following.

### Creating Splits

The logic that generates splits is invoked on the Nephele JobManager.
The InputFormat is instantiated (with its null-ary constructor) and
configured via the **configure(Configuration)** method. Inside the
configure method, parameters that define the behavior of the input
format are read from the passed configuration object. For example, the
file input format reads the path to the file from the configuration.

The actual split generation happens in the method *InputSplit[]
createInputSplits(int minNumSplits)*. The method creates an array of
*[eu.stratosphere.nephele.template.InputSplit](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/template/InputSplit.java "https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/template/InputSplit.java")*s
(or a subclass thereof) that describe the partitions of the data. The
input split interface itself does not provide more information than the
number of the split, and input formats with sophisticated partitioning
typically implement more complex subclasses of InputSplit. The file
input format uses for example the more specialized
*[eu.stratosphere.nephele.fs.FileInputSplit](https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/fs/FileInputSplit.java "https://github.com/stratosphere/stratosphere/blob/master/nephele/nephele-common/src/main/java/eu/stratosphere/nephele/fs/FileInputSplit.java")*,
which holds information about the file path, the start position in the
file's byte stream, the partition length, and the number of hosts where
the split is locally accessible. The internal semantics of the split are
not interpreted by the JobManager, but only by the input format itself,
when it is assigned an input split.

The *InputSplit[] createInputSplits(int minNumSplits)* method is given a
hint how many splits to create at least. It may create fewer, but that
might result in some parallel instances not getting a split assigned.

### Creating Records

In the parallel deployment, the input format class is instantiated once
for each parallel instance. As before, it is configured via the
**configure(Configuration)** method after instantiation. Splits are
assigned to the parallel instances in a lazy fashion - The first one to
be available gets the next split.

Once an input format is assigned a split, its **open(InputSplit)**
method is invoked. This method must set up the internal state such that
the record generation can start. As an example, the file input format
reads the path of the file, opens a file input stream and seeks that
stream to the position that marks the beginning of the split. The input
format for delimited records
*[eu.stratosphere.pact.common.io.DelimitedInputFormat](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/DelimitedInputFormat.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/DelimitedInputFormat.java")*
searches the file stream for the occurrence of the next delimiter.

During the actual record generation process, the method **boolean
nextRecord(PactRecord record)** is invoked as long as a preceding call
to **boolean reachedEnd()** returned false. The method is always given
an instance of the PactRecord, into which the contents of the record is
to be put. The return value indicates whether the record is valid. The
system skips records with a return value of *false* - that way,
individual erroneous records can be skipped.

Because Stratosphere uses mutable objects (for performance reasons), the
given PactRecord object is typically the same each time. The input
format must not use the object to hold whatever state across function
invocations.

Finally, when no more records are read from the split, the **close()**
method is called.

Note that the same instance of the input format is used for multiple
splits, if a parallel instance is assigned more than one input split. In
that case, a closed input format is opened again with another split, so
it must make sure that closing opening cleans and resets internal state
accordingly.

* * * * *

### Input Formats Examples

**Whole File Split Input Format**

A simple change to the file input format is to create only one split per
file. That is useful, if for example the data is encoded in a fashion
that does not permit a read to start in the middle of the file. To
enforce such split generation, simply override the split creation method
as shown below:

    public FileInputSplit[] createInputSplits(int minNumSplits) throws IOException {
        final Path path = this.filePath;
        final FileSystem fs = path.getFileSystem();
     
        final FileStatus pathFile = fs.getFileStatus(path);
     
        if (pathFile.isDir()) {
            // input is directory. list all contained files
            List<FileInputSplit> splits = new ArrayList<FileInputSplit>();
            final FileStatus[] dir = fs.listStatus(path);
            for (int i = 0; i < dir.length; i++) {
                final BlockLocation[] blocks = fs.getFileBlockLocations(dir[i], 0, dir[i].getLen());
                splits.add(new FileInputSplit(i, dir[i].getPath(), 0, dir[i].getLen(), blocks[0].getHosts()));
            }
            return (FileInputSplit[]) splits.toArray(new FileInputSplit[splits.size()]);
        } else { 
            // analogous for one file
        }
    }

* * * * *

**A Data Generating Input Format**

You can easily create an input format that runs a data generator rather
than reading the data with the help of the
*[eu.stratosphere.pact.common.io.GenericInputFormat](https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/GenericInputFormat.java "https://github.com/stratosphere/stratosphere/blob/master/pact/pact-common/src/main/java/eu/stratosphere/pact/common/io/GenericInputFormat.java")*.
It uses a generic input split type that holds no information except its
partition number. The example below shows a minimal example:

    public class GeneratorFormat extends GenericInputFormat
    {
        private Random random;
        private int numRecordsRemaining;
     
        private final PactInteger theInteger = new PactInteger();
        private final PactString theInteger = new PactString();
     
        @Override
        public void configure(Configuration parameters) {
            super.configure(parameters);
            this.numRecordsRemaining = parameters.getInteger("generator.numRecords", 100000000);
        }
     
        @Override
        public void open(GenericInputSplit split) throws IOException {
            this.random = new Random(split.getSplitNumber());
        }
     
        @Override
        public boolean reachedEnd() throws IOException {
            return this.numRecordsRemaining > 0;
        }
     
        @Override
        public boolean nextRecord(PactRecord record) throws IOException {
            this.theInteger.setValue(this.random.nextInt());
            generateRandomString(this.theString, this.random);
     
            record.setField(0, theInteger);
            record.setField(1, theString);
            this.numRecordsRemaining--;
            return true;
        }
     
        private static void generateRandomString(PactString target, Random rnd) {
            ...
        }
    }

Note that the above code example works with mutable objects and reuses
object instances across function calls. In general, user code should
always try to do this, as reference tracing and object garbage
collection can become a serious computational bottleneck. See [Advanced
Pact
Programming](advancedpactprogramming.html "advancedpactprogramming")

</section>

<section id="testing">
Testing PACTs
=============

Stratosphere incorporates a test harness for unit tests of PACT stub
implementations and PACT programs. Similar to JUnit test cases, one or a
pipeline of stub implementations can be tested individually to guarantee
certain behavior for some use and edge cases. The tests do not require
any specific installation and do not perform any network communication.
Thus, a test usually completes within seconds.

The test harness is located in the `pact-client` Maven module in the
package
[eu.stratosphere.pact.testing](https://github.com/stratosphere/stratosphere/tree/master/pact/pact-clients/src/main/java/eu/stratosphere/pact/testing "https://github.com/stratosphere/stratosphere/tree/master/pact/pact-clients/src/main/java/eu/stratosphere/pact/testing").

### Simple Case

Consider the following minimalistic map that simply echos all input
values.

    @SameKey
    public class IdentityMap extends MapStub {
        @Override
        public void map(final PactRecord record, final Collector<PactRecord> out) {
            out.collect(record);
        }
    }

In order to test that the map performs its expected behavior (i.e.
forwarding all input records), the following test case verifies the
correct behavior for one input/output combination.

    public class IdentityMapTest {
     
        @Test
        public void shouldEchoOutput() {
            final MapContract map = MapContract.builder(IdentityMap.class).name("Map").build();
            TestPlan testPlan = new TestPlan(map);
            testPlan.getInput().
                add(new PactInteger(1), new PactString("test1")).
                add(new PactInteger(2), new PactString("test2"));
            testPlan.getExpectedOutput(PactInteger.class, PactString.class).
                add(new PactInteger(1), new PactString("test1")).
                add(new PactInteger(2), new PactString("test2"));
            testPlan.run();
        }
    }

The test initializes a contract with the identity map stub and creates a
TestPlan with it. The following lines initialize the input tuples of the
test and the expected output tuples. Please note, that you have to
specify the expected types.

Finally, the plan is executed and the result is compared with the
expected output. In case of unequal output, the JUnit test will fail due
to an AssertionError. Additionally, if the PACT had an runtime error,
the test would fail with an AssertionError, too.

### Multiple PACTs

Testing a single PACT stub implementation is not sufficient in many
cases. If the collective behavior of a whole PACT pipeline needs to be
tested, the PACTs need to be chained similarly to a normal PACT program.

    final MapContract map1 = MapContract.builder(IdentityMap.class).name("Map").build();
    final MapContract map2 = MapContract.builder(IdentityMap.class).name("Map").input(map1).build();
    TestPlan testPlan = new TestPlan(map2);
    testPlan.getInput().
        add(new PactInteger(1), new PactString("test1")).
        add(new PactInteger(2), new PactString("test2"));
    testPlan.getExpectedOutput(PactInteger.class, PactString.class).
        add(new PactInteger(1), new PactString("test1")).
        add(new PactInteger(2), new PactString("test2"));
    testPlan.run();

In this simple example, the concatenated identity maps should still echo
the input.

### Explicit Data Sources/Sinks

The framework automatically adds data sources and data sinks for each
unconnected input and output respectively. However, there are many cases
where the user needs to exert more control. For example, if he
specifically wants to test a new file format.

    final FileDataSource source = new FileDataSource(IntegerInFormat.class, "TestPlan/test.txt");

    final MapContract map = MapContract.builder(IdentityMap.class).name("Map").input(read).build();

    FileDataSink output = new FileDataSink(IntegerOutFormat.class, "output");
    output.setInput(map);

    TestPlan testPlan = new TestPlan(output);
    testPlan.run();

### Multiple Inputs

So far, we have only seen programs with one implicit data source and
sink. If multiple data sources are involved, there are two ways to
specify the input.

    CrossContract crossContract = CrossContract.builder(CartesianProduct.class).build();
     
    TestPlan testPlan = new TestPlan(crossContract);
    testPlan.getInput(0).
        add(new PactInteger(1), new PactString("test1")).
        add(new PactInteger(2), new PactString("test2"));
    testPlan.getInput(1).
        add(new PactInteger(3), new PactString("test3")).
        add(new PactInteger(4), new PactString("test4"));
     
    testPlan.getExpectedOutput(PactInteger.class, PactInteger.class, PactString.class, PactString.class).
        add(new PactInteger(1), new PactInteger(3), new PactString("test1"), new PactString("test3")).
        add(new PactInteger(1), new PactInteger(4), new PactString("test1"), new PactString("test4")).
        add(new PactInteger(2), new PactInteger(3), new PactString("test2"), new PactString("test3")).
        add(new PactInteger(2), new PactInteger(4), new PactString("test2"), new PactString("test4"));
    testPlan.run();

Additionally, when the sources have been explicitly defined, it is
possible to identify the input with the source.

    CrossContract crossContract = CrossContract.builder(CartesianProduct.class).build();
     
    TestPlan testPlan = new TestPlan(crossContract);
    // first and second input are added in TestPlan
    testPlan.getInput((GenericDataSource<?>) crossContract.getFirstInputs().get(0)).
        add(new PactInteger(1), new PactString("test1")).
        add(new PactInteger(2), new PactString("test2"));
    testPlan.getInput((GenericDataSource<?>) crossContract.getSecondInputs().get(0)).
        add(new PactInteger(3), new PactString("test3")).
        add(new PactInteger(4), new PactString("test4"));
     
    testPlan.getExpectedOutput(PactInteger.class, PactInteger.class, PactString.class, PactString.class).
        add(new PactInteger(1), new PactInteger(3), new PactString("test1"), new PactString("test3")).
        add(new PactInteger(1), new PactInteger(4), new PactString("test1"), new PactString("test4")).
        add(new PactInteger(2), new PactInteger(3), new PactString("test2"), new PactString("test3")).
        add(new PactInteger(2), new PactInteger(4), new PactString("test2"), new PactString("test4"));
    testPlan.run();

Use Test Data from Files
------------------------

It is also possible to either specify data sources/sinks explicitly or
load the input and/or output tuples from a file.

    final MapContract map = MapContract.builder(IdentityMap.class).name("Map").build();
    TestPlan testPlan = new TestPlan(map);
    testPlan.getInput().fromFile(IntegerInFormat.class, "test.txt");
    testPlan.getExpectedOutput(output, PactInteger.class, PactString.class).fromFile(IntegerInFormat.class, "test.txt");
    testPlan.run();

How it works
------------

The TestPlan transparently adds data sources and sinks to the PACT
program if they are not specified. Expected and actual output tuples are
sorted before comparison since the actual order is usually not of
interest.

All network communications are mocked and the test cases are executed
with low memory settings for the MemoryManager. All channels are
replaced with InMemoryChannels to enable fast execution.

However, all PACTs are still executed in their own threads to maintain
the consumer-producer notions. It is also possible to set the degree of
parallelism for all PACTs or the complete test.

Since only little code of the PACT execution is mocked, passed PACT
tests are likely to work in the same way on a complete cluster. However,
the partitioning of the data is not emulated. Therefore, the PACT tests
are not a replacement for integration tests on a real cluster.

---
---

Unit Tests with Mockito and Powermock
-------------------------------------

Readings:

-   [http://mockito.org](http://mockito.org "http://mockito.org")
-   [http://code.google.com/p/powermock/](http://code.google.com/p/powermock/ "http://code.google.com/p/powermock/")
-   [http://code.google.com/p/hamcrest/](http://code.google.com/p/hamcrest/ "http://code.google.com/p/hamcrest/")

Mockito is a Test Framework extending JUnit. It provides easy mechanism
to mock objects in a unit test. Powermock extends Mockito with some
features to test “untestable” code (e.g. static methods, local variables
…).

Example: test the PactCompiler.compile(.) method
------------------------------------------------

signature of the method:

    public OptimizedPlan compile(Plan pactPlan) throws CompilerException

first we have to mock the Plan object pactPlan:

    @Mock Plan myTestPlan;

To give the mocked object a desired behaviour we use the API of Mockito.
E.g. we want, that the call to Plan.getMaxNumberMachines() returns 1:

    //return a valid maximum number of machines
    when(myTestPlan.getMaxNumberMachines()).thenReturn(1);

Finally, if all dependent objects are mocked, we want to test the
method:

    PactCompiler toTest = new PactCompiler();
     
    // now the compile schould succeed
    toTest.compile(myTestPlan);
    verify(myTestPlan, times(1)).getMaxNumberMachines();

the verify-call checks, whether the method getMaxNumberMachines() was
called exactly once. The times(1) is called a VerificationMode in
Mockito. There are more VerificationModes available. See the Mockito
Documentation for more modes and details.

Note: To run the Testcase you should use the
@RunWith(MockitoJUnitRunner.class) Annotation for the test class.

local variables in PactCompiler.compile(.)
------------------------------------------

A closer look to the compile()-method shows some “untestable” code.
There are local variables instantiated, which can not be mocked with
Mockito, since the instantiation is done in the method. To mock this
local variables, we use Powermock:

first we have to use the PowermockRunner as Testrunner:

    @RunWith(PowerMockRunner.class)
    @PrepareForTest(PactCompiler.class)
    public class TestPactCompiler {

The @PrepareForTest annotation tells Powermock, that the class to test
(PactCompiler) has some local variables that we want to mock (same
applies to static methods).

Inside the test method we tell Powermock, that the instantiation of a
class of type OptimizedPlan shall return a mocked object:

    whenNew(OptimizedPlan.class).withArguments(Matchers.any(Collection.class), Matchers.any(Collection.class),
                    Matchers.any(Collection.class), Matchers.any(String.class)).thenReturn(
                    this.mockedOP);

The code means, that a call to the constructor of OptimizedPlan with
argument types (Collection,Collection,Collection,String) with any
arguments (Matchers.any(.)) shall return our mock object mockedOP. For a
more comprehensive view on the test class, see the attached file.

declarative Matchers with Hamcrest
----------------------------------

Since JUnit 4.x declarative matchers may be used to define assertions on
the outcome of tests. The core matchers of hamcrest are already part of
the JUnit library and can be extended by using the hamcrest lib. The
advantage of those matchers are the assertions being readable in a more
intuitive format and the ability to implement own matchers. The
following assertion almost reads like an english sentence (and thus
reduces the time to understand/the need to document the test):

*assertThat(matchCount, is(lessThan(2)));*

</section>

<section id="package">

Package your Program
====================

In order to execute a PACT program on a Nephele cluster it must be
packaged into a JAR file. That file needs to contain all necessary class
files for execution. The JAR typically has a Manifest file with a
special entry   
 ”*Pact-Assembler-Class: \<your-plan-assember-class\>*” which defines
the plan assembler class.   
 **Example manifest file:**

    Manifest-Version: 1.0
    Pact-Assembler-Class: eu.stratosphere.pact.example.wordcount.WordCount

If the plan assembler class is not described within the manifest, its
fully qualified name must be passed as a parameter to the client.

Specify Data Paths
------------------

The paths for data sources and data sinks must be specified as absolute
paths with filesystem prefix. Data can be read from single files or
directories. For more detail consult the Tutorial on [How to write a
Pact
Program](writepactprogram#reading_data_sources "writepactprogram").

-   HDFS Path hdfs://host:port/path/to/data/in/hdfs
-   Local Path (for local setup) file:///path/to/data/in/local/fs