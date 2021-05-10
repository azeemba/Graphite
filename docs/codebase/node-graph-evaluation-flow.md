# Node graph evaluation flow

1. Document Graph -> Asset Graph
	1. Values of non-exposed parameters are inlined
	2. Private subgraphs and code nodes are compiled to machine code and substituted within the graph
	3. History backreferences are flattened *(see [persistent data structure](https://en.wikipedia.org/wiki/Persistent_data_structure))*
2. Asset Graph is baked
	1. Caller provides values for exposed parameters, defaults are used for those not provided
	2. ??? *How do we use some parameters as uniforms passed to the shaders at render time versus others which are required in the baking process, for example as an input bitmap image used by a content aware fill node?* ??? - The content aware fill node internally has its own render node
	3. Graph is traversed and nodes generate and return output data
		* This is where computation happens for complex nodes on the CPU or GPU compute shaders
		* Raster data is a *sampleable* shader
		* ??? *But how does this allow raster input nodes to sample their surroundings, like blur or content aware fill? maybe a forward and backward pass?* ???
3. Baked Asset Graph is rendered
	* For 2D rendering: X, Y, and Quality are sampled inputs -> colored pixels and a quality buffer are outputs
	* For vector rendering: nothing is input -> vector data are outputs
	* For audio rendering: time or input stream is input -> audio stream is output
	* For 3D geometry rendering: nothing is input -> 3D geometry is output




Rendering should be a very small part of the process, everything else should be part of the nodes themselves. Consider a 2D document containing a rendered 3D model with a texture provided by another 2D document. That texture lookup should happen on the 2D texture document directly, without having to first choose a resolution to render the whole document and then sample from the final texture. (For AA, this also means samples aren't based on square pixels anymore, either.) In this example, the 2D <- 3D <- 2D flow should happen entirely within the node graph as part of a single 2D render just to get pixels into the viewport or onto the output file. In other words, the "render" step is nothing more than an output— the nodes need to handle the actual pixel fill calculations, because this is needed for the 2D texture sampling for the 3D model and also for the 3D rendering to 2D.


Key questions:
* When can we include the values for exposed parameters? If they change often? Live video data for example?
* How and when does sampling work? How do we make the "sampleable" concept work, with lensing, work with a fragment shader?
* When do we actually evaluate the node graph? It seems that rendering is something intrinsic to a data type and/or nodes that use a data type (like raster or 3D geometry), and there is just a concept of outputting (to the viewport, a file, a game that uses the library, etc.)