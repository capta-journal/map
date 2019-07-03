# State Map of US

## Show a Map

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

## Add Color
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
      "legend": { "title": "Total Population Change" }
    }
  }
}
```
