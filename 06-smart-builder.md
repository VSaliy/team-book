# Multi-Module Build Performance Improvements with the Takari Smart Builder

## Overview

The Takari Smart Builder, available at https://github.com/takari/takari-local-repository, is 
an alternative scheduler for build multi-module Maven projects. The primary difference between 
the standard multi-threaded scheduler in Maven and the Takari 
Smart Builder scheduler is illustrated below.

TBD

The standard multi-threaded scheduler using a rather naive and simple approach of using 
dependency-depth information in the project. Tt builds everything at a given 
dependency-depth before continuing to the next level.

The Takari Smart Builder scheduler is using a more advanced approach of dependency-path information.
Projects are aggressively built along a dependenty-path in topological order as upstream dependencies 
have been satisfied. 

In addition to the more aggressive build processing the Takari Smart Builder can optionally record 
project build times to determine your build's critical path. If possible, the Takari Smart Builder w
always attempt to schedule projects on the critical path first.

**NOTE: You need to have Maven 3.2.1+ to use this extension.** 

## Installation

To use the Takari Smart Builder you must install it in Maven's `lib/ext` folder:

```
curl -O http://repo1.maven.org/maven2/io/takari/maven/takari-smart-builder/0.3.0/takari-smart-builder-0.3.0.jar

cp takari-smart-builder-0.3.0.jar $M2_HOME/lib/ext
```

## Using Smart Builder

To take advantage of the Smart Builder you need to use multiple threads in order for the Smart Builder 
scheduling capabilities to take affect. To use the Smart Builder invoke Maven by specifying the number 
of threads with or the number of threads per core:

```
mvn clean install --builder smart -T8
```

or

```
mvn clean install --builder smart -T1.0C
```

Also note that if you are running builds in parallel that projects download their dependencies just 
prior to building the project. This means if you have two projects that are built simultaneously
and require the same dependency it is likely your local Maven repository will be corrupted. 
In order to avoid this problem we recommend using the Takari Local Repository implementation 
which provides thread/process safe access to the local Maven repository. 

## Using Critical Path Scheduling

If you want to try the critical path scheduling you need to create an `.mvn` directory at the 
root of your project for the `timing.properties` to be persisted. If there is no timing information 
available the critical path is estimated as the path with the greatest number of segments. On 
subsequent runs the timing information is used to calculate the critical path and an attempt is 
made to schedule that first. Where possible we try to schedule project builds such that your build 
should take no longer than the critical path, though this is not always possible.

## Reference and History

The original implementation of the Smart Builder was created byTravis Downs and Brian Toal at 
Salesforce. It was abased on ideas from the paper "Static vs. Dynamic List-Scheduling Performance Comparison". 
Takari subsequently added the metrics persistence and critical path scheduling.