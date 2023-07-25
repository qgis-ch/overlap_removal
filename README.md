# overlap_removal
QGIS process to remove overlaps of geometries in the same layer and assign the overlap to the larger neighboring polygon

Surprisingly, QGIS doesn't provide an overlap removal tool "out of the box" in processing. So we came up with this solution of a combination of algorithms.
Suppose you have a dirty data set where neighboring polygons partially overlap. You want to assign the overlapping parts to the neighboring polygon with the largest area - and keep the original attribute information.

Here is a sample data set illustrating some - for illustration purpose - very large overlaps:

![grafik](https://github.com/qgis-ch/overlap_removal/assets/884476/333a74cb-7bb0-40bc-ab2c-2e13ddc3d624)

The sequence of algorithms is as follows:

1. Calculate the "original_area" of your input polygons (with some precision, e.g. 3 decimal places). This area is used as a kind of unique identifier. It is very unlikely that you will have two polygons with the exact same area down to a couple of places behind the decimal point.
2. Union of the input layer: Overlapping polygons will appear twice or multiple times in the union output.
3. Another area calculation to later allow the aggregation of the overlapping part
4. Aggregate geometries (eliminate the double or multiple polygons), while keeping the largest area of the neighbouring polygon
5. Dissolve while using the "original_area" as the attribute to keep. The result is the correct merging with the largest neighbouring polygons, but without the original attributes.
6. Join by location to get back the original attributes

