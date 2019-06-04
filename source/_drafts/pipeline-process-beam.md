---
title: 'pipeline process: beam'
toc: true
date: 2019-01-30 17:02:49
tags:
	- bigdata
	- distributed processing
---

# What's beam

[beam](https://beam.apache.org/get-started/beam-overview/) is a open-source, unified model for defining both batched & streaming data-parallel processing pipelines.

* open-source (apache v2 license)
* to define data-parallel processing pipelines
* an unified model to define pipelines. The real processing is run by the underlying runner (eg. spark, apache apex, etc.). [all available runners](https://beam.apache.org/get-started/beam-overview/)
* can process both batched  (bounded datasets) & streaming (unbounded datasets) datasets

# Use it

See the [wordcount examples](https://beam.apache.org/get-started/beam-overview/), [wordcount src](https://github.com/apache/beam/blob/master/examples/java/src/main/java/org/apache/beam/examples/MinimalWordCount.java)

Now we define a simple pipeline and run it.

`Transform`, `Count` are all built-in atom operations to define the pipeline scripts.

```java
package org.apache.beam.examples;

import java.util.Arrays;
import org.apache.beam.sdk.Pipeline;
import org.apache.beam.sdk.io.TextIO;
import org.apache.beam.sdk.options.PipelineOptions;
import org.apache.beam.sdk.options.PipelineOptionsFactory;
import org.apache.beam.sdk.transforms.Count;
import org.apache.beam.sdk.transforms.Filter;
import org.apache.beam.sdk.transforms.FlatMapElements;
import org.apache.beam.sdk.transforms.MapElements;
import org.apache.beam.sdk.values.KV;
import org.apache.beam.sdk.values.TypeDescriptors;

public class MinimalWordCount {

  public static void main(String[] args) {

    // Create a PipelineOptions object. This object lets us set various execution
    // options for our pipeline, such as the runner you wish to use.
    PipelineOptions options = PipelineOptionsFactory.create();

    // Create the Pipeline object with the options we defined above
    Pipeline p = Pipeline.create(options);

    // Concept #1: Apply a root transform to the pipeline; in this case, TextIO.Read to read a set
    p.apply(TextIO.read().from("gs://apache-beam-samples/shakespeare/*"))

        // Concept #2: Apply a FlatMapElements transform the PCollection of text lines.
        .apply(
            FlatMapElements.into(TypeDescriptors.strings())
                .via((String word) -> Arrays.asList(word.split("[^\\p{L}]+"))))
        .apply(Filter.by((String word) -> !word.isEmpty()))
        // Concept #3: Apply the Count transform to our PCollection of individual words. 
        .apply(Count.perElement())
        .apply(
            MapElements.into(TypeDescriptors.strings())
                .via(
                    (KV<String, Long> wordCount) ->
                        wordCount.getKey() + ": " + wordCount.getValue()))
        // Concept #4: Apply a write transform, TextIO.Write, at the end of the pipeline.
        .apply(TextIO.write().to("wordcounts"));

    p.run().waitUntilFinish();
  }
}
```

# Some conceptions

## I/O (data source/target)

Beam can process both batched  (bounded datasets) & streaming (unbounded datasets) datasets. [built-in io transforms](https://beam.apache.org/documentation/io/built-in/)

Take reading as example, you specify the file location (the location must be accessable for the runner), and then the reader pull from datasource. You may also define the trigger to collect input window. When trigger is satisfied, window elements are emitted.

For unbounded datasets, they are split into windows. And each window is again a bounded datasets. In each window, there're some elements. You can define how the elements are grouped as a window and when to emit the window elements for processing. [window concept](https://beam.apache.org/documentation/programming-guide/#windowing)

## Runner

Beam is an unified model. It abstracts the conception to define and run a pipeline. The real execution is conducted by the underlying runners.

[all available runners](https://beam.apache.org/get-started/beam-overview/)

For unbounded datasets, the underlying runner must support stream processing.