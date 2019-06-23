```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "width": 800,
  "height": 500,
  "data": {
    "url": "https://vega.github.io/vega-lite/data/income.json"
  },
  "transform": [
    {
      "lookup": "id",
      "from": {
        "data": {
          "url": "https://vega.github.io/vega-lite/data/us-10m.json",
          "format": {
            "type": "topojson",
            "feature": "states"
          }
        },
        "key": "id"
      },
      "as": "geo"
    }
  ],
  "projection": {"type": "albersUsa"},
  "mark": { "type": "geoshape", "stroke": "white" },
  "encoding": {
    "shape": {"field": "geo","type": "geojson"},
    "color": {"field": "pct","type": "quantitative"}
  }
}
```

```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "width": 800,
  "height": 500,
  "projection": {
    "type": "albersUsa"
  },
  "data": {
    "url": "https://vega.github.io/vega-lite/data/us-10m.json",
    "format": {
      "type": "topojson",
      "feature": "states"
    }
  },
  "mark": {
    "type": "geoshape",
    "fill": "lightgray",
    "stroke": "white"
  }
}
```

```{vl}
{
  "$schema": "https://vega.github.io/schema/vega-lite/v3.json",
  "width": 800,
  "height": 500,
  "data": {
    "url": "https://vega.github.io/vega-lite/data/us-10m.json",
    "format": {
      "type": "topojson",
      "feature": "counties"
    }
  },
  "transform": [{
    "lookup": "id",
    "from": {
      "data": {
        "url": "https://vega.github.io/vega-lite/data/unemployment.tsv"
      },
      "key": "id",
      "fields": ["rate"]
    }
  }],
  "projection": {
    "type": "albersUsa"
  },
  "mark": "geoshape",
  "encoding": {
    "color": {
      "field": "rate",
      "type": "quantitative"
    }
  }
}
```


- [Command-Line Cartography, Part 1](https://medium.com/@mbostock/command-line-cartography-part-1-897aa8f8ca2c)
- [Choropleth Map: US States](https://d3-geomap.github.io/map/choropleth/us-states/)
- [Modular US State Choropleth](https://bl.ocks.org/wboykinm/dbbe50d1023f90d4e241712395c27fb3)
- [U.S. Daily Cigarette Smoking Rate, 1996-2012](http://bl.ocks.org/dougdowson/9832019)