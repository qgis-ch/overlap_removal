# overlap_removal
QGIS process to remove overlaps of geometries in the same layer and assign the overlap to the larger neighboring polygon

Surprisingly, QGIS doesn't provide an overlap removal tool "out of the box" in processing. Apparently, ArcGIS has such a tool "out of the box". So we came up with one possible solution of a combination of algorithms.
Suppose you have a dirty data set where neighboring polygons partially overlap. You want to assign the overlapping parts to the neighboring polygon with the largest area - and keep the original attribute information.

Here is a sample data set illustrating some - for illustration purpose - very large overlaps:

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/333a74cb-7bb0-40bc-ab2c-2e13ddc3d624)

## Sequence of Algorithms

1. Calculate the "original_area" of your input polygons using the **Refactor fields** algorithm (with some precision, e.g. 3 decimal places). This area is used as a kind of unique identifier. It is very unlikely that you will have two polygons with the exact same area down to a couple of places behind the decimal point.
2. **Union** of the input layer: Overlapping polygons will appear twice or multiple times in the union output.
3. Another area calculation with **Refactor fields** to later allow the aggregation of the overlapping part
4. **Aggregate"** geometries (eliminate the double or multiple polygons), while keeping the largest area of the neighbouring polygon
5. **Dissolve** while using the "original_area" as the attribute to keep. The result is the correct merging with the largest neighbouring polygons, but without the original attributes.
6. **Join attributes by location** to get back the original attributes

## Graphical model
![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/05fe6533-18a1-4843-b816-3591910ffef1)


## Detailed parameters and steps

### 1. Refactor fields and area calculation
![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/4dec8b19-802a-4764-8f27-feaf6ac4dfae)

The formula for calulating the original area with 3 decimal places is ```round(area($geometry),3)```.

### 2. Union of the input layer
This creates separate polygons for the non-overlapping parts and the overlapping parts, however, when two polygons are overlapping, you will get the overlapping parts duplicated - these need to be eliminated later.

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/67472ed4-1f53-4758-bd88-b1e5133dcde1)
(graphics from the QGIS manual)

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/65e67aa6-6733-4dc7-ab2f-eea77789639e)

We don't need an overlay layer if we just check against overlaps *within* the layer.

### 3. Calculating areas with "Refactor fields"
Same as in step 1. You will notice in the attribute table of the intermediate results that features are duplicated and appear several times with the same area, but different "original_area" attributes:

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/53670772-c962-4fda-9bfe-bd5a8711e804)

### 4. Aggregate algorihtm to eliminate multiple overlaps

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/13c4fe6c-88ba-4961-994b-fd6e15f92662)

We group by attribute *area* but the keep the maximum of the *original_area* so we can later dissolve with the neighbour polygon with the largest area.

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/e77ebca4-0827-49f3-8ad7-eb37b78f8f8d)

### 5. Join attributes by location
to get back the original attributes of the input layer.

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/bede103a-e17d-4ccd-9748-eb1630d4315f)

Note the "Join type" "Take attributes from the feature with the largest overlap only".

## Result without overlaps and original attributes

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/1bc2b96a-9be7-4382-a232-1bb0cdd7d4ed)
