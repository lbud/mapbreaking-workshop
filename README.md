# Learning WebGL by breaking stuff
**A workshop for doing reckless things with maps**

### [Lauren Budorick](https://github.com/lbud) & [Anjana Vakil](https://github.com/vakila), [Mapbox](https://github.com/mapbox)
### [WebCamp Zagreb 2017](https://2017.webcampzg.org/workshops/learning-webgl-by-breaking-stuff/)

## Description

In this workshop we'll try to wrap our heads around some basic concepts of WebGL, an open-source library for 3D web graphics based on the OpenGL project. But instead of working through safe, boring tutorial examples, we'll live dangerously and do it the fun way: by breaking things! We'll get our hands dirty by cloning and exploring, then tweaking and breaking a mature WebGL-based project: Mapbox GL JS, an open source library for rendering 3D, interactive maps in the browser. By messing with its internals, we'll create some crazy maps that are very bad for getting around town, but very good for building intuitions about how WebGL works its magic.

## Setup instructions

* [Create a free Mapbox account](https://www.mapbox.com/signup/) if you don’t yet have one
* Follow the [mapbox-gl-js setup instructions](https://github.com/mapbox/mapbox-gl-js/blob/master/CONTRIBUTING.md#preparing-your-development-environment) specific to your operating system
* [Start up the debug server](https://github.com/mapbox/mapbox-gl-js/blob/master/CONTRIBUTING.md#serving-the-debug-page) using the [access token](https://www.mapbox.com/studio/account/tokens/) for your Mapbox account

## Links

* [Diff showing extra annotations](https://github.com/mapbox/mapbox-gl-js/commit/0ab948b27712a178c297d13ea0d27cff1aab8df2)
* [mapbox-gl-js directory guide :book:](./mbgl_directory_guide.txt)
* [Worksheet :hammer_and_pick:](./worksheet.md)

# Walkthrough examples

* In src/data/bucket/fill_bucket.js (use [http://localhost:9966/debug/index.html](http://localhost:9966/debug/index.html)), modify [this line](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/data/bucket/fill_bucket.js#L142) to add a random integer between 0 and 8192 instead of the x or y coordinate (or try both). Can we explain what this does? (Try zooming to the Croatian islands maybe, where there are lots of interesting polygons with good background contrast color). Zoom in and out; when and why does it change at intervals?

<details>
<summary>Answer:</summary>
    When we randomize the X coordinate, we’re moving just one vertex of a polygon by setting it to anywhere from the far left to the far right of the tile. Chances are this is going to be pretty far away from its intended X coordinate, producing the horizontal jagged effect. Same with the Y coordinate. If we randomize the coordinate, then the jaggedness goes all over the place, because one of the polygon’s vertices is anywhere on the tile, probably far from where it should be.
</details>

* In src/data/bucket/fill_extrusion_bucket.js (use [http://localhost:9966/debug/buildings.html](http://localhost:9966/debug/buildings.html)), comment out [this line](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/data/bucket/fill_extrusion_bucket.js#L165), and then change [this](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/data/bucket/fill_extrusion_bucket.js#L168) to add only one instead of two (this is where we keep track of how many triangles we should have in the index array — this is more or less just bookkeeping). Can we explain what’s happening? Why do all the rooftops look intact? How could we make those look just as jagged?

<details>
<summary>Answer:</summary>
    When we comment out the first line, we’re removing one out of every two triangles that make up the “walls” of a 3D extrusion (remember, we create a triangle by adding the indices of its vertices in the vertex array to the index array), so in our map, every wall will have only one lower triangle.
</details>

* In src/data/bucket/circle_bucket.js (use [http://localhost:9966/debug/circles.html](http://localhost:9966/debug/circles.html)), [here](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/data/bucket/circle_bucket.js#L145-L146) we add two triangles. What if we changed which indices we made the triangles out of? Change the first line to add triangles at index 0-2-3, then the second to add triangle 0-3-1. Can we explain this? Using [this diagram](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/data/bucket/circle_bucket.js#L131-L135) as a guide, how would you flip the pacman in the opposite direction?

<details>
<summary>Answer:</summary>
    We’re still using all the same vertices — which basically make up a square (in the fragment shader we shave off the corners to make it round) — but connecting them wrong. Instead of constructing a lower right and an upper left, we’re now constructing an upper left triangle and a lower left triangle, leaving a slice of the circle unaccounted for. If you wanted to flip the pacman, you could use triangles 0-1-2 and 1-2-3 (note those numbers don’t need to be in order).
</details>

* Text is rendered using a method called [“signed distance fields (SDFs),”](https://blog.mapbox.com/drawing-text-with-signed-distance-fields-in-mapbox-gl-b0933af6f817) which are essentially rasterized single-channel representations of glyphs, where the value of a given pixel within a glyph is higher the more “inside” the glyph it is, and lower the further outside the glyph it is. We use this value to determine the opacity of a pixel within a glyph “quad” (rectangle) when we sample the SDF raster [here](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/shaders/symbol_sdf.fragment.glsl#L420. If you invert the alpha value [here](https://github.com/mapbox/mapbox-gl-js/blob/78a685240f07c4af6ece224ebd46022e8e60ce1a/src/shaders/symbol_sdf.fragment.glsl#L47) (alpha range is [0, 1]), how does this affect rendering?

<details>
<summary>Answer:</summary>
    This will invert text rendering, creating a knockout effect. Note how throughout the fragment shader `color` is generally static: we’re actually filling in the entire quad with that same rgb value, and the visibility of any given pixel depends only on its opacity.
</details>

## Helpful hints

* You’ll see `.emplaceBack` in a lot of the bucket types — think of this as being the same as an array `.push`.
* GLSL is especially type-strict, so doing operations with different number types (e.g. adding a `float` and an `int`) is not allowed. Most numbers in these shaders are `float`s, so usually if you want to e.g. divide a number by 2, it’ll need to be `myNum / 2.0`.
* If you’re curious about any of the GLSL functions in the shaders (like `mix`, `smoothstep`, etc), [here](http://www.shaderific.com/glsl-functions/) is a helpful reference.
* There’s lots of syntactic sugar in GLSL for vector component access called [“swizzling”](https://www.khronos.org/opengl/wiki/Data_Type_(GLSL)#Swizzling) that make vector access faster, where you can use the shorthands `xyzw`, `rgba`, or `stpq` — so constructing `[color[2], color[0]]` is as simple as `color.gr`, though this could also be written `color.zx` or `vec2(color.g, color.r)`, etc. Modifying a single channel of `color` can be done by changing, say, `color.r = 0.3`.
* Here are a few things you’ll see across various bucket types. You don’t need to worry about modifying them, but if you’re curious as to what they do:
* `prepareSegment` is sort of a bookkeeping and utility function used to make sure the vertex array and index array we’re going to use isn’t already too full; these arrays have a maximum capacity of 65535, so if adding this feature would push them over the limit, prepareSegment takes care of creating new arrays.
* `holeIndices` is present in fill_bucket and fill_extrusion_bucket; it’s just doing bookkeeping necessary for running the `earcut` algorithm.
