# Spatial data

The page is currently being updated, check back later.

## Introduction 
In this chapter, we will take a glimpse into spatial analysis using R. There are an overwhelming number of R packages for analyzing and visualizing spatial data. In broad terms, spatial visualizations require a merging of non-spatial and spatial information. For example, if you wish to create a choropleth map of the murder rate by county in New York State, you need county level data on murder rates, and you also need geographic data for drawing the county boundaries, stored in what are called shape files. 

A rough divide exists between packages that don’t require you deal with shape files and those that do. The former work by taking care of the geographic data under the hood: you supply the data with a column for the location and the package takes care of figuring out how to draw those locations. 


## Packages

### `choroplethr`

Choropleth maps use color to indicate the value of a variable within a defined region, generally political boundaries. `choroplethr` is capable of drawing state and county level maps without using shape files.

Consider the following example using `state.x77` showing the percentage of illiterate in each states. Note the process of data transformation, you need to exactly have a column named 'region' with the state names and a column named 'value'.


```r
library(dplyr)
library(tibble)
library(ggplot2)
library(choroplethr)

# data frame must contain "region" and "value" columns

df_illiteracy <- state.x77 %>% as.data.frame() %>% 
  rownames_to_column("state") %>% 
  transmute(region = tolower(`state`), value = Illiteracy)

state_choropleth(df_illiteracy,
                 title = "State Illiteracy Rates, 1977",
                 legend = "Percent Illiterate")
```

<img src="spatial_files/figure-html/unnamed-chunk-1-1.png" width="672" style="display: block; margin: auto;" />

Plotting at county level is also possible. In the following example we show the population of counties in New York state in 2012.


```r
library(choroplethrMaps)
data(county.regions)
data(df_pop_county) 

ny_county_fips = county.regions %>%
  filter(state.name == "new york") %>%
  select(region)

county_choropleth(df_pop_county, 
                  title = "Population of Counties in New York State in 2012",
                  legend = "Population",
                  county_zoom = ny_county_fips$region)
```

<img src="spatial_files/figure-html/unnamed-chunk-2-1.png" width="672" style="display: block; margin: auto;" />

We use `county_choropleth` to create a U.S map at county level and zoom into New York state. Note that `county_zoom` takes in fips codes of counties. For more reference, visit the [R documentation page](https://www.rdocumentation.org/packages/choroplethr/versions/3.7.0).


### `ggmap`

`ggmap` is a great package to generate real-world maps. Moreover, it is compatible with `ggplot2` allowing you to easily add layers on top of the base map. There are two options in generating maps and we demonstrate one using stamenmap.


```r
library(ggmap)

ny_map <- get_stamenmap(bbox = c(left = -74.2591, bottom = 40.4774, 
                                 right = -73.7002, top = 40.9162),
                        zoom = 10, maptype = "toner-lite")

ggmap(ny_map) +
  geom_point(aes(x=-73.9857,y=40.7484),size=0.8,color='blue') +
  annotate(geom="text",label="Empire State Building",x=-73.9, y=40.755,color='blue',size=3)
```

<img src="spatial_files/figure-html/unnamed-chunk-3-1.png" width="672" style="display: block; margin: auto;" />

A map of New York City is created, and we added the point representing Empire State Building. You can very simply add points using `geom_point` as long as you have the coordinates for the points. However, one clear drawback using stamenmap is that the map resolution is not satisfying and you will eventually find that the smaller zoom value creates blurry maps.

For this reason, we encourage readers to try the option using Google maps. You will need to set up a Google Cloud account (requires credit card). After enabling the Google map APIs, register you API keys with `ggmap` using `register_google(key = "[your key]")` and you are all set. Now take a look at the an example using Google map API

```
ggmap(get_googlemap(center = c(lon = -74.006, lat = 40.7128),zoom = 14))
```

<center>
![](images/ggmap_google.png){width=70%}
</center>

You will see the map is at great resolution regardless of how you zoom in. For demonstration purpose, we provided a image of the map derived by running the code. For more details regarding the package usage, you can refer at [ggmap GitHub page](https://github.com/dkahle/ggmap).


