# Guide to Showing State-by-State Data on a Map

![Google image search of "by state"](data/google_image_search_by_state.png)

The above screenshot shows the result of Googling images "by state." As a data analyst, you too have probably dealt with data organized by state at some point. Visualizing such data on a map can help you spot regional trends and anomalies. It's also a more attractive way to present your analysis, as a bar chart with 50 bars is just not very pretty.

You may also know that creating such thematic map (technically known as a [choropleth](https://en.wikipedia.org/wiki/Choropleth_map)) is non-trivial. You either have to reach for specialized geospatial tools or dive deep into Javascript/D3 programming. Here I'll show you a much simpler alternative using [Vega-Lite](https://vega.github.io/vega-lite/), a visualization language where you specify your graph using JSON. At the end of this tutorial, you'll have created this population growth choropleth:

```{vl file=pop_growth_rate.vl.json}
```
*Note: Source code and data for this article, including all visualizations, are in this [Github repo](https://github.com/capta-journal/map). You are welcome to fork the repo, edit it, and republish your version within [capta.studio](https://www.capta.studio/publish).*

## Building the Choropleth
We'll show how we build the choropleth above in four steps. After the first two steps you'll have a recognizable choropleth. The third step tidy things up to make it more presentable. The last step shows a few advanced techniques to make the visualization more meaningful.

### Step 1: Show a Map
```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" },
  "projection": { "type": "albersUsa" }
}
```

The foundation of a choropleth is a map, and this is often where most people get stuck. Fortunately, [Vega-Lite](https://vega.github.io/vega-lite/) provides great support for geospatial visualization. The United States map above is rendered by just a few lines:

```json
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" },
  "projection": { "type": "albersUsa" }
}
```

Besides Vega-Lite's concise grammar for geospatial visualization, the other part of the secret sauce is in the file `cb_2018_us_state_5m.json`, referenced in the `data.url` property above. This file, in a geospatial file format called [TopoJSON](https://github.com/topojson/topojson), contains data in lat/lon for the shape and location of all 50 states. We created it based on digital data from the Census Bureau. The code and the gory details to generate this file are documented elsewhere. We're open-sourcing the resulting data so you can reuse it as a "black box."

Before continuing, let's understand the JSON spec above in a bit more detail. The `$schema` property simply says this JSON file is a Vega-Lite specification. The `data` property points to our map file. Vega-Lite needs to know that this file is in TopoJSON `format.type`. A TopoJSON file can encode multiple `feature`s of a map to show different boundaries. For example, a map of the US may show states, counties, zip codes, etc. The `cb_2018_us_state_5m.json` file includes only the `states` feature.

The `mark` property tells Vega-Lite what type of visual "mark" to display our `data`. For a choropleth this will always be `geoshape`. Here we also specify colors for drawing those shapes.

The `projection` property is used to lay out, or "project," the geospatial shapes into something visually "meaningful." What's meaningful will depend on context. The `albersUsa` projection is a commonly used projection for showing the US map. It centers the map around the continental United States. It also intentionally puts Alaska and Hawaii in the "wrong" places, but the overall result is more meaningful and concise for most readers. 

### Step 2: Add Color / Data
So we got a map. To make it a choropleth let's bring in some data to color the states. The example data we'll use is [population estimates from the U.S. Census Bureau](https://www.census.gov/data/tables/time-series/demo/popest/2010s-state-total.html)^[The full US census is taken once every ten years, with the last one in 2010. Our data set is an _estimate_ of population changes since then, up to 2018.]. The Census Bureau website provides functions for viewing and downloading the data but doesn't provide a permanent link it, so I've included the [CSV file](data/PEP_2018_PEPTCOMP.csv) as `PEP_2018_PEPTCOMP.csv` in [this document's repo](https://github.com/capta-journal/map). Let's look at the first four columns of a few rows of this dataset:

```csv
GEO.id,GEO.id2,GEO.display-label,popchg4201072018
0400000US01,01,Alabama,107733
0400000US02,02,Alaska,27189
0400000US04,04,Arizona,779358
0400000US05,05,Arkansas,97797
0400000US06,06,California,2302522
```

The column names are given by the Census Bureau, and `popchg4201072018` represents the net population change between April 2010 and July 2018. The [CSV file](data/PEP_2018_PEPTCOMP.csv) has other columns breaking down the changes due to births, deaths, and migrations (domestic and international). I'll leave it as an exercise for the interested reader to explore those data.^[The sharp-eyed reader will notice there are extra rows in `PEP_2018_PEPTCOMP.csv` for areas that are not states, such as "Northeast Region," "Midwest Region," and "Puerto Rico." The default behavior in the lookup transform is to ignore those data, which works well for us here. On the other hand, if you have _missing_ data, the default behavior is to drop those areas on the map, which is rarely what you want. See the [world choropleth](../../../world/capta.md) example on how to gray out areas with missing data rather than leave them blank.]

Coloring areas on a choropleth is basically like filling out the color-by-number drawings for kids. To make it work, the coding on the map (`cb_2018_us_state_5m.json`) must be the same as the coding in our dataset (`PEP_2018_PEPTCOMP.csv`). In database parlance this is finding the join keys. Fortunately, unlike the color-by-number drawings for kids, our map supports more than one coding. The state map (`cb_2018_us_state_5m.json`) has the 2-digit [U.S. Federal Information Processing Standard](https://en.wikipedia.org/wiki/Federal_Information_Processing_Standard_state_code) (FIPS) code as the default `id` key. (California is '06'.) It also has the uppercase 2-letter state code as the `STUSPS` key ("CA"), and the full name as the `NAME` key ("California").

Looking back at our CSV file (`PEP_2018_PEPTCOMP.csv`), the first three columns are redundant coding for state. The `GEO.id` column has a coding that our map file doesn't have, so we couldn't use it. The `GEO.id2` column matches the `id` key in our state map, so we could use that pair for matching areas. The `GEO.display-label` column also matches the `NAME` key in our map, so that pair would also work. Given this choice, I prefer the numeric keys (`id`/`GEO.id2`) over the text ones, as it's less likely to make matching errors due to misspellings or case sensitivity.

With that background, let's extend our Vega-Lite spec from a bland map to a choropleth:

```json
{
  "$schema": ...,
  "data": ...,
  "mark": ...,
  "projection": ...,
  "transform": [{
    "lookup": "id",
    "from": {
      "data": { "url": "data/PEP_2018_PEPTCOMP.csv" },
      "key": "GEO\\.id2",
      "fields": ["popchg4201072018"]
    }
  }],
  "encoding": {
    "color": {
      "field": "popchg4201072018",
      "type": "quantitative"
    }
  }
}
```

Note the use of `\\.` in specifying `key`.

```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" },
  "projection": { "type": "albersUsa" },
  "transform": [{
    "lookup": "id",
    "from": {
      "data": { "url": "data/PEP_2018_PEPTCOMP.csv" },
      "key": "GEO\\.id2",
      "fields": ["popchg4201072018"]
    }
  }],
  "encoding": {
    "color": {
      "field": "popchg4201072018",
      "type": "quantitative"
    }
  }
}
```

### Step 3: Customize Legend and Tooltip
The population information is now encoded on the map. Let's go ahead and fix a few visual loose ends.

The default legend is quite poor in this case. It assumes the title from our data's field name ("popchg4201072018"), which will just be cryptic to our readers. The labels on the legend ("1,000,000", "2,000,000", and "3,000,000") are so long that they're squished together. Let's customize the title and reformat the labels.

If you mouse over the states (or tap on a mobile device), you'll see a tooltip showing details of the underlying data. The default tooltip in this case has the similar problems as the legend: the title is again the cryptic "popchg4201072018" and the labels can be better formatted. Here the labels don't need to be shortened, but we want to add commas or periods to make them more readable. In addition, we want to help the reader identify states by showing their names in the tooltip. Fortunately, the ready-to-use topojson file includes not only the data to render a map, but it also has other useful information in its `properties` object. State name is encoded in `properties.NAME` and we'll show that in our tooltip.

The added code is shown below. Of note is the `titleLimit` for legend. It's the limit (in pixels) of how long the title can be. [The default is 180](https://vega.github.io/vega-lite/docs/legend.html#title) which is too short for our title. As mentioned above, we change how the numbers are formatted using the `format` property. Explanation for the code is given in the [(Vega-Lite/D3) documentation](https://github.com/d3/d3-format#locale_format).

```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "projection": { "type": "albersUsa" },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" },
  "transform": [{
    "lookup": "id",
    "from": {
      "data": { "url": "data/PEP_2018_PEPTCOMP.csv" },
      "key": "GEO\\.id2",
      "fields": ["popchg4201072018"]
    }
  }],
  "encoding": {
    "color": {
      "field": "popchg4201072018",
      "type": "quantitative",
      "legend": {
        "title": "Total Population Change (2010-2018)",
        "titleLimit": 300,
        "format": "~s"
      }
    },
    "tooltip": [
      {
        "field": "properties.NAME",
        "type": "nominal",
        "axis": { "title": "State" }
      },{
        "field": "popchg4201072018",
        "type": "quantitative",
        "format": ",",
        "axis": { "title": "Population Change" }
      }
    ]
  }
}
```


### Step 4: Bonus - Calculate Other Values
The previous three steps are sufficient for making a usable choropleth. For making your own choropleth, that may be all the code you'll need. However, this minimal attempt may not always bring out the necessary insights from data, which can be context-specific. For our population change visualization, an analyst may want to bring out different points:
1. The color scale is linear, while population change is not. The 3.5M additional residents in Texas is more than the total population of some states. It's overwhelming and (misleadingly) makes the rest of the country look stable.
2. Depending on the context, the relative population change may be more important than the absolute population change. For example, if I'm a business looking for additional customers, I may care only about the absolute number. On the other hand, if I'm a sociologist investigating population trends, the relative number would be more useful.
3. The color scale can be locked onto special reference values. For example, population change can, and does, have negative values. It could be useful to have a color scale that locks onto zero and differentiates between positive and negative growth. As an alternative, one may "zero-indexed" on the *national growth rate* instead, so the differentiation is between those growing faster than average and those that are slower. This would be helpful in understand where the population gravity is shifting.

For this exercise I'll change the choropleth to show percentage population growth and the color scale indexed around the national growth rate. For your own choropleth you may have a different set of customizations (or none at all). I've chosen this set of changes mostly because it's a realistic requirement while also illustrates a few techniques.



```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "projection": { "type": "albersUsa" },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" },
  "transform": [{
    "lookup": "id",
    "from": {
      "data": { "url": "data/PEP_2018_PEPTCOMP.csv" },
      "key": "GEO\\.id2",
      "fields": ["popchg4201072018"]
    }
  },{
    "lookup": "id",
    "from": {
      "data": { "url": "data/PEP_2018_PEPANNRES.csv" },
      "key": "GEO\\.id2",
      "fields": ["rescen42010"]
    }
  },{
    "calculate": "datum.popchg4201072018 / datum.rescen42010",
    "as": "growth"
  }],
  "encoding": {
    "color": {
      "field": "growth",
      "type": "quantitative",
      "scale": {
        "scheme": "purplegreen"
      },
      "legend": {
        "title": "Population Growth Rate (2010-2018)",
        "titleLimit": 300,
        "format": ".1%"
      }
    },
    "tooltip": [
      {
        "field": "properties.NAME",
        "type": "nominal",
        "axis": { "title": "State" }
      },{
        "field": "popchg4201072018",
        "type": "quantitative",
        "format": ",",
        "axis": { "title": "Total Population Change" }
      },{
        "field": "growth",
        "type": "quantitative",
        "format": ".1%",
        "axis": { "title": "Growth Rate" }
      }
    ]
  }
}
```

In addition to converting our choropleth to show growth rate, we also changed the color scheme to a "divergent" one, meaning the color intensity goes up on both ends of the scale. For our data, the default center of the scale has no particular meaning, so we should change it to something more interpretable. An obvious choice is to center at zero. To do so, we could specify a scale from -15% to +15%, which would cover all possible values while ensuring the middle is at zero. (We chose 15% because from the graph above we know Utah is the fastest growing state at 14.4%.)

We'll leave it to you to make that change. Here we'll do something a little fancier. From looking at the original data, we know the total U.S. population increase was 18,409,329, based on a 2010 population of 308,745,538. That is, the U.S. population has grown 6%. Let's use a color range to help see states that are growing faster than this national average versus the slower growing states. With a little bit of arithmetic, we decide to use a scale from -3% to 15%, to ensure centering at 6%.

```{vl file=pop_growth_rate.vl.json}
```

#### Extra
We also have pre-made map files for you to build other choropleths ([countries of the world](../../../world/capta.md), counties in US, etc.) And if you don’t see one that fits your need, you can create an issue on Github to ask the community for help. Chances are some other people will have the same need and will be happy to help create it.