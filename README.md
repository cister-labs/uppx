# Uppx

Allows extending an UPPAAL model with __annotated blocks__ and __XML blocks__, e.g.:
```xml
<nta>
    <declaration>
        ...
        // @myAnnotation
        const int v1 = 1;
        const int v2 = 2;
        
        ...
    </declaration>
    ...
    <queries>
        ...
    </queries>
    ...
</nta>
```
And reads a companion MS Excel file (with the same base name) with tables that describe how to adapt the block following an annotation command, until the next empty line.
For example, with the expression and table below in a sheet called `@myAnnotation`, the values 1 and 2 will become 10 and 20.

 `const int $var = $number;`

| var | num |
| --- | --- |
| v1 | 10 |
| v2 | 20 |

It is also possible to replace the content of the `<queries>` block by introducing a sheet named `<queries>` to our spreadsheet with a table like the one below:

`<query> <formula>$Formula</formula> <comment>$Comment</comment> </query>`

|Formula | Comment|
| ------ | -------|
|`A[]!deadlock` | No deadlocks|
|`A[] W.Idle` | The worker is always Idle|

This concrete table will replace the content of the `<queries>` block by two `<query>` blocks containing the corresponding formulas and comments from the table.


## Supporting multiple configurations

Often we want to experiment with different combinations of values and queries.

Uppx supports the specification of a list of configurations, each producing variations of the values and XML blocks, following principles from Software Product Line Engineering.
Uppx generates, for each of the provided configurations, a different instance of the original UPPAAL model and verifies all properties of all instances.
We will first see how to define configuration by selecting desired "features", and then how to enrich the annotations with these "features".

### Specifying configurations

A special sheet named `@Configurations` lists and specifies the configuration in a table similar to the one below:

|Configuration | _Feature1_ | _Feature2_ | ...  |
| :----------- | :------: | :-------: | ---- |
| _Main_ |  |  |  |    
| _Conf2_ | x | | |
| _Conf3_ |  | x | |

The names below __Configuration__ identify the set of desired configurations, and every non-empty cell at the right of these selects the feature name at the top of the corresponding column.

In this example we have 3 configurations: _Main_, _Conf2_, and _Conf3_; configuration _Conf2_ selects _Feature1_, _Conf3_ selects _Feature2_, and _Main_ does not select any feature.
Each feature selection in a configuration will modify the annotations, explained below, yielding a different instance of the UPPAAL model.

### Enriching annotations with features

Recall that each annotation is described by a table ...

## Build the (fat) jar

Dependencies:

 - SBT [(https://www.scala-sbt.org)](https://www.scala-sbt.org)
 - JVM (>=1.8)
 - UPPAAL (optional - https://uppaal.org)

Use the sbt task to build the self-contained jar-file running at the root:
```bash
sbt assembly
```

## Run the jar

The compiled jar-file can be found in `target/scala-3.0.2/uppx.jar`.
You can copy it to your working folder, with the UPPAAL and Excel files. To list the possible options run in the command line:
```bash
java -jar uppx.jar --help
```

For example, to apply the default configuration in a file `myfile.xlsx` to an UPPAAL model `myfile.xml`, you can run the command:

```bash
java -jar uppx.jar myfile.xlsx
```

The file names of the configuration and UPPAAL files must match.
You will be presented with a list of changes applied to the `myfile.xml`, this file will be updated, and a copy of the original file will be placed in a `backups` folder, which will be created if it does not exist.

To check all properties using UPPAAL of all configurations, you can run the command:

```bash
java -jar uppx.jar --runAll myfile.xlsx
```

This requires the command `verifyta` to be available in the `PATH`, which can be found in the binaries included by UPPAAL.
Extra options, such as a timeout value, can be defined here, requiring the command `timeout` to be available in the `PATH`.

## Examples

You can find a simple example in folder [examples/simple](./examples/simple) to illustrate the usage of annotations. 
It includes a minimalistic Uppaal file, a simple Excel file, and a script `runuppaal.command` to call Uppx and Uppaal.
This example includes both __annotation__ and __XML__ blocks; the former are in sheets whose name starts in `@`, and the latter are in sheets whose name is surrounded by angular brackets `<>`.

To run the simple example, first build the fat jar (`sbt assembly`).
Then, using the command line in the folder of the example, type `sh runuppaal.command`.

---

A variation of this example can be found in folder [examples/simple-with-conf](./examples/simple-with-conf) that further illustrates how to apply __multiple configurations__, using the special sheet named `@Configurations`.