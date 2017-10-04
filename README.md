# Welcome!

* [Diff showing extra annotations](https://github.com/mapbox/mapbox-gl-js/commit/0ab948b27712a178c297d13ea0d27cff1aab8df2)
* [mapbox-gl-js directory guide](./mbgl_directory_guide.txt)
* [Walkthrough](./walkthrough.md)
    * [...with answers](./walkthrough_with_answers.md)
* [Worksheet](./worksheet.md)

## Helpful hints

* You’ll see .`emplaceBack` in a lot of the bucket types — think of this as being the same as an array `.push`.
* GLSL is especially type-strict, so doing operations with different number types (e.g. adding a `float` and an `int`) is not allowed. Most numbers in these shaders are `float`s, so usually if you want to e.g. divide a number by 2, it’ll need to be `myNum / 2.0`.
* If you’re curious about any of the GLSL functions in the shaders (like `mix`, `smoothstep`, etc), [here](http://www.shaderific.com/glsl-functions/) is a helpful reference.
* There’s lots of syntactic sugar in GLSL for vector component access called [“swizzling”](https://www.khronos.org/opengl/wiki/Data_Type_(GLSL)#Swizzling) that make vector access faster, where you can use the shorthands `xyzw`, `rgba`, or `stpq` — so constructing `[color[2], color[0]]` is as simple as `color.gr`, though this could also be written `color.zx` or `vec2(color.g, color.r)`, etc. Modifying a single channel of `color` can be done by changing, say, `color.r = 0.3`.
* Here are a few things you’ll see across various bucket types. You don’t need to worry about modifying them, but if you’re curious as to what they do:
* `prepareSegment` is sort of a bookkeeping and utility function used to make sure the vertex array and index array we’re going to use isn’t already too full; these arrays have a maximum capacity of 65535, so if adding this feature would push them over the limit, prepareSegment takes care of creating new arrays.
* `holeIndices` is present in fill_bucket and fill_extrusion_bucket; it’s just doing bookkeeping necessary for running the `earcut` algorithm.
