# State Map of US

### Step 1: Show a Map

```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "data": {
    "url": "topojson/cb_2018_us_state_5m.json",
    "format": { "type": "topojson", "feature": "states" }
  },
  "projection": { "type": "albersUsa" },
  "mark": { "type": "geoshape", "fill": "lightgray", "stroke": "gray" }
}
```

### Step 2: Add Color / Data
https://www.census.gov/data/tables/time-series/demo/popest/2010s-state-total.html

Note the use of `\\.` in specifying `key`.


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
      "legend": { "title": "Total Population Change (2010-2018)" }
    }
  }
}
```

### Step 3: Optional - Customize Tooltip
If you mouse over the states in the above choropleth (or tap on a mobile device), you'll see a tooltip showing the underlying data. By default it repeats the encoded information in a generic format. We show you how to customize it a little bit.

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
      "legend": { "title": "Total Population Change (2010-2018)" }
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
The previous three steps are sufficient for making a choropleth. However, this minimal attempt may not always bring out the necessary insights from data, which can be context-specific. For our population change visualization, an analyst may want to bring out different points:
1. The color scale is linear, while population change is not. The 3.5M additional residents in Texas is more than the total population of some states. It's overwhelming and (misleadingly) makes the rest of the country look stable.
2. Depending on the context, the relative population change may be more important than the absolute population change. For example, if I'm a business looking for additional customers, I may care only about the absolute number. On the other hand, if I'm a sociologist investigating population trends, the relative number would be more useful.
3. The color scale could be locked onto special reference values. For example, population change can, and does, have negative values. It could be useful to have a color scale that locks onto zero and differentiates between positive and negative growth. As an alternative, one may "zero-indexed" on the *national growth rate* instead, so the differentiation is between those growing faster than average and those that are slower. This would be helpful in understand where the population gravity is shifting.

For this exercise I'll change the choropleth to show percentage population growth and the color scale indexed around the national growth rate. For your own choropleth you may have a different set of customizations (or none at all). I've chosen this set of changes mostly because it's a realistic requirement while also illustrates a few techniques.

