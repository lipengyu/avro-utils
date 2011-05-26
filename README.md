# Avro Utils

Util library to be able to use Avro files as input and output of Hadoop Map/Reduce Jobs
or use Avro files as input of Hadoop Streaming.

## Avro I/O in Map/Reduce Jobs

### Avro Input

Use `com.tomslabs.grid.avro.AvroFileInputFormat` to use Avro files as the *input* of Map/Reduce jobs.  

'map()' will be called with a key of type Avro's `GenericRecord` and a `NullWritable` value.

### Avro Output

Use `com.tomslabs.grid.avro.AvroFileOuputFormat` to use Avro files as the *output* of Map/Reduce jobs.  

In `reduce()` method, the context must emit with a key of type Avro's `GenericRecord` and  a `NullWritable` value.
Please note that the _Avro Schema for the output data MUST be specified when setting up the Job_.
 
### Example 

`src/test/java/com/tomslabs/grid/avro/AvroWordCount.java` is an example showing how to use Avro files for both input and output of a Map/Reduce jobs.

This example can be run as unit test from `AvroWordCountTest.java`.


## Avro Input for Hadoop Streaming

To use Avro files as input for Hadoop Streaming, use the jar generated by the project and specify the correct input format:

    $ $HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
        -libjars ./avro-utils-<VERSION>.jar,avro-1.4.1.jar  \
        -inputformat com.tomslabs.grid.avro.AvroTextFileInputFormat \
        -input <Avro file or dir> \
        -output <output dir> \
        -mapper <map command> \
        -reducer <reducer command>
     
For example, to count the number of expression, something like:

    $ $HADOOP_HOME/bin/hadoop  jar $HADOOP_HOME/hadoop-streaming.jar \
        -libjars ./avro-utils-1.5.1-SNAPSHOT.jar,avro-1.4.1.jar  \
        -inputformat com.tomslabs.grid.avro.AvroTextFileInputFormat \
        -input /tmp/word-count.avro \
        -output /tmp/out \
        -mapper /bin/cat \
        -reducer /usr/bin/wc

The format of each line streamed through Avro looks like:

    <JSON representation of a Avro record>\t

(i.e. there is a trailing tabulation at the end of each line)

## Avro Input for Dumbo

To use Avro files as input for Dumbo, use the jar generated by the project and set correct input format to
`com.tomslabs.grid.avro.AvroAsTextTypedBytesInputFormat`:

    $ dumbo start <PYTHON_SCRIPT> \
         -input /tmp/word-count.avro \
         -output /tmp/out \
         -libjar avro-1.4.1.jar \
         -libjar avro-utils-<VERSION> \
         -inputformat com.tomslabs.grid.avro.AvroAsTextTypedBytesInputFormat
         -hadoop <HADOOP_HOME>
         -python <PYTHON_HOME>
         -outputformat text

The Python script's mapper will get the Avro record as a JSON string in its `value` parameter (the `key` parameter is not used).

## Avro Output for Hadoop Streaming

You can use Avro files as the output for Hadoop Streaming.
This expects to receive in the reducer a Text key containing the JSON representation of a Avro record.
To store in Avro (binary) files instead of text file, you must specifiy the properties:

    -D avro.output.schema=$SCHEMA \
    -libjars avro-utils-<VERSION>.jar,avro-1.4.1.jar \
    -reducer com.tomslabs.grid.avro.JSONTextToAvroRecordReducer \
    -outputformat org.apache.avro.mapred.AvroOutputFormat 

where SCHEMA is a String containing the JSON representation of the Avro *schema* to use to create the Avro records.

## Links

* [http://avro.apache.org/](http://avro.apache.org/)
* [http://wiki.apache.org/hadoop/HadoopStreaming](http://wiki.apache.org/hadoop/HadoopStreaming)
* [https://github.com/klbostee/dumbo/wiki/](Dumbo)

