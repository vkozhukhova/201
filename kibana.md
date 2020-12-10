# Kibana visualizations

## Lens

**This functionality is in beta.**

To create visualizations, you drag and drop your data fields onto the visualization builder pane, and Lens automatically generates a visualization that best displays your data.

With Lens, you can:

* Use the automatically generated visualization suggestions to change the visualization type.
* Create visualizations with multiple layers and indices.
* Add your visualizations to dashboards and Canvas workpads.

To get started with Lens, you need to select a field in the data panel, then drag and drop the field on a highlighted area.

## Line, area, and bar charts

Compares different series in X/Y charts.

## Pie chart

Displays each source contribution to a total.

## Data table

Flattens aggregations into table format.

## Metric

Displays a single number.

## Goal and gauge

Displays a number with progress indicators.

## Tag cloud

Displays words in a cloud, where the size of the word corresponds to its importance.

## TSVB

Visualizes time series data using pipeline aggregations.

TSVB comes with these types of visualizations:

### Time Series

A histogram visualization that supports area, line, bar, and steps along with multiple y-axis.

### Metric

A metric that displays the latest number in a data series.

### Top N

A horizontal bar chart where the y-axis is based on a series of metrics, and the x-axis is the latest value in the series.

### Gauge

A single value gauge visualization based on the latest value in a series.

### Markdown

Edit the data using using Markdown text and Mustache template syntax.

### Table

Display data from multiple time series by defining the field group to show in the rows, and the columns of data to display.

## Timelion

Computes and combine data from multiple time series data sets. Timelion is a time series data visualizer that enables you to combine totally independent data sources within a single visualization. It’s driven by a simple expression language you use to retrieve time series data, perform calculations to tease out the answers to complex questions, and visualize the results.

## Maps

Enables you to parse through your geographical data at scale, with speed, and in real time. With features like multiple layers and indices in a map, plotting of raw documents, dynamic client-side styling, and global search across multiple layers, you can understand and monitor your data with ease.

With Maps, you can:

* Create maps with multiple layers and indices.
* Upload GeoJSON files into Elasticsearch.
* Embed your map in dashboards.
* Symbolize features using data values.
* Focus in on just the data you want.

## Heat map

Display graphical representations of data where the individual values are represented by colors. Use heat maps when your data set includes categorical data. For example, use a heat map to see the flights of origin countries compared to destination countries using the sample flight data.

## Dashboard tools

Visualize comes with controls and Markdown tools that you can add to dashboards for an interactive experience. **This functionality is experimental.** The controls tool enables you to add interactive inputs on a dashboard.

You can add two types of interactive inputs:

* Options list — Filters content based on one or more specified options. The dropdown menu is dynamically populated with the results of a terms aggregation. For example, use the options list on the sample flight dashboard when you want to filter the data by origin city and destination city.
* Range slider — Filters data within a specified range of numbers. The minimum and maximum values are dynamically populated with the results of a min and max aggregation. For example, use the range slider when you want to filter the sample flight dashboard by a specific average ticket price.

## Vega

**This functionality is experimental.** Vega and Vega-Lite are both declarative formats to create visualizations using JSON. Both use a different syntax for declaring visualizations, and are not fully interchangeable.

Vega and Vega-Lite are capable of building most of the visualizations that Kibana provides, but with higher complexity. The most common reason to use Vega in Kibana is that Kibana is missing support for the query or visualization, for example:

* Aggregations using the nested or parent/child mapping
* Aggregations without a Kibana index pattern
* Queries using custom time filters
* Complex calculations
* Extracting data from `_source` instead of aggregation
* Scatter charts
* Sankey charts
* Custom maps
* Using a visual theme that Kibana does not provide

