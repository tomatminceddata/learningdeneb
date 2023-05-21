# learningdeneb
Over the next couple of months I will use this repo to share my learning about using deneb for creating data visualizations inside Power BI. If you are not familiar with deneb you should start here: https://deneb-viz.github.io/

I consider deneb being a very powerful custom data visual available for Power BI. deneb does not provide a certain data visualization to solve a data visualization task instead it provides a framework that allows us creating almost any data visualization we need to communicate insights with the users of our Power BI reports. During the last years I encountered various requirements for specific data visualizations that have not been met by the default visuals or the custom visuals available. Sometimes these requirements could have been described as minor additions to the way information is presented, sometimes these requirements only could be met by creating R or Python script visauls. Sometimes I considered creating a new custom visual. All of these custom approaches have their downsides, no matter how powerful packages like ggplot2 or seaborn are, the data visuals created using this are lacking interactivity the users of Power BI are used to. Creating tailor made custom visuals requires additional tools like Visual Studio Code for developing the custom visual. From my point of view the amazing freedom that comes with creating my own custom visuat that fits the needs exactly comes with a price tag, maintenance and the integration of additional functionality, meaning new features. Next to that, I do not know that many business users that are familiar with the required developent skills.

I consider deneb different, because it provides a framework where I, the report author can focus on the data visualization task without the need for additional tools. When I say deneb provides a framework where almost any data visualization can be created, this of course also states that there are certain tasks that can not be solved. This is true for two reasons. The first reason of course is simple, it's simply because of my lack of knowledge, but this is what this repo is all about sharing my learning experience. The second reason is a more complex one. At the current moment deneb embeds two javascript libraries used for data visualization, vega-lite and vega. There are certain data visualization task that can not be solved with the above mentioned libraries alone, e.g. the mapping of data to spatial aesthetics. But I'm happy about these kind of limitations as there certain capabilities of a custom visual require communicating with third party services, this does not necessarily mean that data is not safe, but this prevents the visual from becoming a certified. From my personal perspective I consider being certified more important than being able to talk with third party services.

deneb allows to describe how the data at hand should be visualized by providing a specifiaction, this specifiaction is using json notification. Then this specifiaction is used for rendering a data visual, that out the box supports cross filter and tooltip capabilities (thanks to the inventer of the deneb custom visual). If you think this is simple, you are correct. Unfortunately, the vega-lite and the vega library provide a lot of means to manipulate the available data, create new data using transforms and expressions. Because, it is required to provide the "formula" in a syntax I'm not familiar with, I'm struggling. I thing what Alberto Ferrari and Marco Russo once have said about DAX is also valid for using vega-lite and vega: Using vega-lite or vega is simple, but not easy.

As I'm convinced that deneb will play a great part when a custom visual is required, I will learn how use it.

Throughout, the next months I will use this repo to share my findings, my projects - the data visualization tasks I'm currently working on.

The next sections will desribe the topology of a specifiation, its sections like mark, encoding, layers, and all the details that have not been from the beginning. This is not about how to use deneb inside Power BI, this is described in very great detail on the official site.

# The specification
# The transform
More than one year has passed between publishing this repository and writing this section, but I learned some new things, forgot some of them, and re-learned them. Now is the time to write these lines, as I do not want to spend time on things I already solved once too often. I realize that I'm still struggling with the syntax of vega/vega-lite. I assume this is because of the time I'm able to spend with deneb, and not because of the complexity.
A vega/vega-lite specification might or might not contain an array of data transformations. Remember the specification of a data visualization for a single view, here a bar chart is depicted:
{
"data": ... ,
"transform": [
{ ... }
],
"mark": {"type": "bar", ... },
"encoding": { ... }
}
To better understand what this article is about. The following data

```"data": {
    "values": [
      {"group": "I", "a": "C", "b": 2000000, "t": 22500000},
      {"group": "I", "a": "C", "b": 7000000, "t": 22500000},
      {"group": "I", "a": "C", "b": 4000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 1000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 2000000, "t": 22500000},
      {"group": "I", "a": "D", "b": 6000000, "t": 22500000},
      {"group": "II", "a": "E", "b": 8000000, "t": 18500000},
      {"group": "II", "a": "E", "b": 4000000, "t": 18500000},
      {"group": "II", "a": "E", "b": 7000000, "t": 18500000}
    ]
```

will be transformed and visualized. The next image shows the data visualization:

<img width="295" alt="transform - the visualization to achieve" src="https://user-images.githubusercontent.com/29025119/236114636-3140b7ce-6a6d-4692-a5f4-dc7a3b3f5441.png">

A quick count of the data points reveals that the data viz is only visualizing 7 data points, whereas the table contains 9 rows. When we are looking closer, we realize that the data labels of the "this" view represent the sum of the segment values of the stacked bar chart. This finding might trigger a re-count of the data point and we will end somewhere between 


+ 5, the data points of bar chart are used to "calculate" the tabular data points, plus two data points for the tick mark visualization
+ 6, 3 datapoints to visualize stack I and 1 data point to visualize stack II, where the data point is used to twice, plus two data points for the data visualization
+ 7,  7 independent data points are used
No matter the result of the counting the data points. It is obvious that the data inside the table has to be transformed. The next sections describe specific transformations to create the above visualization. As I will use this writing as a how-to on transformation for some future work it will also contain transformations that are not necessary to obtain the data visualization.

## General thoughts about data transformations
Data transformations are required to shape the available data in a way that a given body of data (basically the dataset) can be used to create a data visualization that helps us telling our data story conveys insights in an efficient way.
If you are wondering if you should use DAX to "shape" your data before you pass the data to the deneb visual or use vega/vega-lite transforms. My answer is simple: it depends. If you are sure that the DAX statement will be used more than once and you are not familiar with vega/vega-lite then of course use DAX. My experience dictates using the vega/vega-lite transformations though. I can tell they are fast and do not add measures to a data model that are only used in combination with specific data visualizations. Another reason why I do data transformations inside of deneb is reusability. I realized that it is often more simple to adapt a "complex" deneb specification to a given dataset, than to bring more complex DAX statements into various Power BI datasets.
A transform is specified either in the main specification, but it can also be specified inside an individual view if there is a multi-view composition, no matter if this composition is a multilayered view using "layer": ... or a multi-view composition, e.g., using "hconcat:" ... 
Some things to know about data transformations:

+ The transform array can contain multiple transformations. 
+ Transformations are executed in the order of definition. 
+ The result of a data transformation is available in subsequent data transformations.

One of the transformations that are described in this chapter is "rowIndexByGroupAndSegment". This column in combination with the filter transform helps to remove duplicate rows. It is very likely that real duplicate rows will not be present in your dataset when you are working with Power BI, the reason for this is that Power BI automatically removes duplicate rows by grouping across all rows, but without using aggregation functions for numeric values. Nevertheless, I tend to use various transforms at the view level and not in the encoding channel. Maybe this is only because of my data engineering experience, but I think that defining all transforms in a single place is reducing the overall complexity of the specification.

## The mental model of the data visualization (I will find a better place for this section in the not so distant future)
Dear reader, please excuse this section. I know that you are expecting data transformation related aspects, but as I'm revisiting this repo after quite some time I consider this important, and I'm pretty sure that I will put this section somewhere else when I do some re-factoring of this repo. 
Whenever I start something new using a technology or programming language that I do not speak fluently, I start with creating a mental model to discover obstacles early.
To create the above data visualization some challenges/obstacles are obvious:

+ The final data viz is constructed by aligning different visualization types, a table like structure on the left, a stacked bar chart with data labels in the middle and a table like structure on the right. All three visualizations have to use the data from the column "group" as a common y-axis
+ Data has to be aggregated to create the values for the tabular like visualization on the left
+ Data has to be aggregated to create the data labels for the three segments ("C", "D", "E")

If you are not familiar with the ![vega-lite text mark](https://vega.github.io/vega-lite/docs/text.html), then one task that has to be tackled that does not appear immediately before your third eye -  the placement of the data label inside each bar at the base (or left) of each segment. If you are using Power BI as I do, then we have to admit that we are a little spoiled in regards to the placement of the data labels. We simply turn on the property, then we tweak the placement a little inside the given boundaries, nevertheless, often I'm able to achieve what I want. If not, I use the one that comes closest to what I want. Now, using vega-lite it's time to face the hard reality, showing data labels is not a simple property, it requires thinking and some maths. The next section will describe all the necessary transformations (more or less) to determine the position for the text placement at the base of each segment.

As this section will move to a better place, the gist that creates the above data viz will stay, here is the link: ![multi-view composition - stacked-bullet chart](https://vega.github.io/editor/#/gist/30bb2932e8924bce69ec7d0fd050831e/multi view composition - stacked bullet chart.json)

## the text mark (this section will move to where it belongs, guess what - in the not so distant future)
The next image shows all transforms including the "filter" transform that is removing duplicate rows.  The column "xSegmentBase" might be considered the final transform, this column contains the values necessary to place the text (that's what a data label is) at the base (the left border of a horizontal bar) of each segment (column: a) inside a stack (column: group):

<img width="700" alt="transform - data object - multiview and stacked bullet-chart" src="https://user-images.githubusercontent.com/29025119/236117075-3e158042-5f43-4275-9f5b-3b7e97a8f297.png">

I think it's obvious that I start with the result, this is also a habit of mine as it helps me to stay focused. Here it measn that I describe the calculate transform first.

## the calculate transform ![calculate transform](https://vega.github.io/vega-lite/docs/calculate.html)
The position of a segment's base is determined by a simple subtraction: [the sum of all preceeding segments] - [the value of the current segment]. To create a new column and calculate a value for each row the transform "calculate" can be used, the next code snippet shows the transform:

```
{
  "calculate": "datum['runningSumBySegment'] - datum['segmentSum']",
  "as": "xSegmentBase"
}
```
The formula to calculate the wanted value is provided between quotation marks. The name of the new column is defined by using the key-word "as".
Referencing a column inside a data object is done by using datum. Datum can be used with many notations styles like datum.columnname or as I do datum['columnname']. To be honest, I'm not expert enough to recommend one over the other.

## the join aggregate ![join aggregate](https://vega.github.io/vega-lite/docs/joinaggregate.html)
JoinAggregate is a very powerful transform as it creates new values based on aggregations across multiple rows while preserving all rows. This transform is very useful if  detailValue / groupValue calculations are needed for the data visualization. Preserving rows of course, means repeating  values. This must not be fogotton. I will remove all unnecessary rows by using the filter transform, but this is a later step.
The next two code snippets show how the columns segmentSum and groupSum are calculated:

```
{
  "joinaggregate": [
    {
     "field": ["b"], 
     "op": ["sum"],
     "as": ["segmentSum"]
    }
  ],
  "groupby": ["group", "a"] 
}
```

```
{
  "joinaggregate": [
    {
      "field": ["b"], 
      "op": ["sum"], 
      "as": ["groupSum"]
    }
  ],
  "groupby": ["group"]
}
```

Looking at the syntax, it becomes obvious that each joinaggregate transformation is part of a list (the outer curly brackets). This list contains multiple transformations, where each transformation is an array. 
The next image shows how the transform works to create the column "segmentSum":

<img width="343" alt="transform - joinaggregate" src="https://user-images.githubusercontent.com/29025119/236118505-3e4fed28-dd7e-4d1d-a11d-32ad0c0f2592.png">

It's necessary to note that the order of the columns that are passed to the "groupby" phrase does not influence the result.

## the window transform ![window transform](https://vega.github.io/vega-lite/docs/window.html)
WINDOW adding new columns based on calculations for a sorted partition of a data object (a sorted group of rows). This is similar to the windowing functions you might know from SQL and can also be compared to the new DAX windowing functions. The next two code snippets show how the columns "rowIndexByGroupAndSegment" and "runningSumBySegment" are calculated:

```
{
  "window": [
    {
      "op": ["row_number"], 
      "as": ["rowIndexByGroupAndSegment"]
    }
  ], 
  "groupby": ["group", "a"]
}
```

```
{
  "window": [{
    "op": "sum",
    "field": "segmentSum",
    "as": "runningSumBySegment"
  }],
  "groupby": ["group"],
  "sort": [{"field": "rowIndexByGroup", "order": "ascending"}]
}
```

Due to the nature of the operator ("op", the operator determines the window function) the data field can be omitted, e.g. row_number does not require a data field.
The transform that creates the column "runningSumBySegment" is special in many cases, first it is demonstrating that columns can be referenced that have been created in preceeding transforms, but it is also using the power of the window transform. Here the data object is partitioned by the field "group", then the rows inside each partition are sorted by the field "rowIndexByGroup". The partitions are determined by the columns specified by the "groupby" operator. But the real "magic" of the window transform happens when the aggregation function (specified by the "op" operator) is applied to each row of the ordered partition. But this application is not only considering the current row, instead the aggregation function is applied to the current row and all preceeding rows, the preceedings rows are determined by the value of the column that is defined by the "sort" operator.  I always imagine the application of a window transform (but not only when I'm using vega or vega-lite) as an iterator. This iterator loops across all the rows inside a partition applies the aggregation function to a stack of values. Of course this stack is can be a single value, e.g. when the aggregation is more simple like count or sum, or a real stack of many values when the aggregation function is more complex like rank. I assume this is the reason why I consider the application of window transforms as some kind of magic, I do not have to implement the logic by myself I only use operators and magic is happening. The next image hopefully explains some of this magic:

<img width="425" alt="transform -window iterating across rows inside a partition" src="https://user-images.githubusercontent.com/29025119/236118887-007321a1-0041-40ad-886c-33bd5111fa46.png">

## the filter transform ![filter transform](https://vega.github.io/vega-lite/docs/filter.html)
The filter transform simply filters the data object, the next code snippet shows a very simple filter:

```
{
  "filter": "datum['rowIndexByGroupAndSegment']===1"
}
```

I very often use the transform operation "joinaggregate", this operation repeats an aggregated value for a given partition. This of course requires removing these duplicate values at a later stage. This is exactly what the above snippet is doing.

# The mark
# The encoding
