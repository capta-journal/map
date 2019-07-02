# State Map of US

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