# Java-Call-Graph-Utilities

A suite of programs for generating static and dynamic call graphs in Java.

* javacg-static: Reads classes from a jar file, walks down the method bodies and
   prints a table of caller-caller relationships.
* javacg-dynamic: Runs as a [Java agent](http://download.oracle.com/javase/6/docs/api/index.html?java/lang/instrument/package-summary.html) and instruments
  the methods of a user-defined set of classes in order to track their invocations.
  At JVM exit, prints a table of caller-callee relationships, along with a number
  of calls

#### Compile

The java-callgraph package is build with maven. Install maven and do:

```
mvn install
```

This will produce a `target` directory with the following three jars:
- javacg-0.1-SNAPSHOT.jar: This is the standard maven packaged jar with static and dynamic call graph generator classes
- `javacg-0.1-SNAPSHOT-static.jar`: This is an executable jar which includes the static call graph generator
- `javacg-0.1-SNAPSHOT-dycg-agent.jar`: This is an executable jar which includes the dynamic call graph generator

#### Run

Instructions for running the callgraph generators

##### Static

`javacg-static` accepts as arguments the jars to analyze.

```
java -jar javacg-0.1-SNAPSHOT-static.jar lib1.jar lib2.jar...
```

`javacg-static` produces combined output in the following format:

###### For methods

```
  M:class1:<method1>(arg_types) (typeofcall)class2:<method2>(arg_types)
```

The line means that `method1` of `class1` called `method2` of `class2`.
The type of call can have one of the following values (refer to
the [JVM specification](http://java.sun.com/docs/books/jvms/second_edition/html/Instructions2.doc6.html)
for the meaning of the calls):

 * `M` for `invokevirtual` calls
 * `I` for `invokeinterface` calls
 * `O` for `invokespecial` calls
 * `S` for `invokestatic` calls
 * `D` for `invokedynamic` calls

For `invokedynamic` calls, it is not possible to infer the argument types.

###### For classes

```
  C:class1 class2
```

This means that some method(s) in `class1` called some method(s) in `class2`.

##### Dynamic

`javacg-dynamic` uses
[javassist](http://www.csg.is.titech.ac.jp/~chiba/javassist/) to insert probes
at method entry and exit points. To analyze a class `javassist` must
resolve all dependent classes at instrumentation time. It reads
classes from the JVM's boot classloader. JVM sets the boot
classpath by Java's default classpath implementation (`rt.jar` on
Win/Linux, `classes.jar` on the Mac). The boot classpath can be extended using
the `-Xbootclasspath` option, which works the same as the traditional
`-classpath` option. It is advisable for `javacg-dynamic` to work as expected,
to set the boot classpath to the same, or an appropriate subset, entries as the
normal application classpath.

Moreover, since instrumenting all methods will produce huge callgraphs which
are not necessarily helpful (e.g. it will include Java's default classpath
entries), `javacg-dynamic` includes support for restricting the set of classes
to be instrumented through include and exclude statements. The options are
appended to the `-javaagent` argument and has the following format
