# Explore materials in Reality Composer Pro

Learn how Reality Composer Pro can help you alter the appearance of your 3D objects using RealityKit materials. We’ll introduce you to MaterialX and physically-based (PBR) shaders, show you how to design dynamic materials using the shader graph editor, and explore adding custom inputs to a material so that you can control it in your visionOS app.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10202", purpose: link, label: "Watch Video (20 min)")

   @Contributors {
      @GitHubUser(stevenpaulhoward)
      @GitHubUser(multitudes)
   }
}



# Chapters
[0:00 - Introduction](https://developer.apple.com/videos/play/wwdc2023/10202/?time=0)  
[0:55 - Materials in xrOS](https://developer.apple.com/videos/play/wwdc2023/10202/?time=55)  
[3:38 - Material editing](https://developer.apple.com/videos/play/wwdc2023/10202/?time=218)  
[10:26 - Node graphs](https://developer.apple.com/videos/play/wwdc2023/10202/?time=626)  
[13:16 - Geometry modifiers](https://developer.apple.com/videos/play/wwdc2023/10202/?time=796)  
[19:14 - Wrap-up](https://developer.apple.com/videos/play/wwdc2023/10202/?time=1154)  

<!-- The timestamp was redundant because already set above...-->

# Materials in visionOS

![What Are Materials][whatAreMaterials]

[whatAreMaterials]: WWDC23-10202-whatAreMaterials

Materials are what define the appearance of the objects in 3D scenes. Materials can be simple, just a single color, or they can use images. We might apply a wood texture to the model of a chair or map an image of bricks to a wall. Materials can also be quite sophisticated. They might use animation to look like rippling water or change appearance based on viewing angle. Like iridescent sparkle of mother of pearl. Materials can even modify the geometry of the objects they are applied to.

## Physically Based Rendering
Materials in visionOS use Physically Based Rendering (PBR) to represent physical properties of real world objects, for example, how metallic or how rough an object looks.  

![Physically Based Rendering example][pbr]

[pbr]: WWDC23-10202-pbr

Materials consist of one or more **shaders**. These are programs that do the actual work of computing the appearance of their material.

![shaders][shaders]

[shaders]: WWDC23-10202-shaders

With RealityKit 2 for iOS and iPadOS, Apple introduced CustomMaterial. Shaders in CustomMaterial are hand coded in Metal.

![Shaders in CustomMaterial are hand coded in Metal][metal]

[metal]: WWDC23-10202-metal

**Surface shaders** operate on the PBR attributes of models. **Geometry modifiers** operate on the geometry of objects.

## ShaderGraphMaterial

In xrOS, a new type of material is been introduced: **ShaderGraphMaterial**.

![shaderGraphMaterial][shaderGraphMaterial]

[shaderGraphMaterial]: WWDC23-10202-shaderGraphMaterial

**This is the exclusive way of creating custom materials for visionOS**. Based on the open standard [MaterialX](https://materialx.org).

Supports 2 main types of shaders: **Physically Based** and **Custom**.

![shaderGraphMaterial][shaderGraphMaterial2]

[shaderGraphMaterial2]: WWDC23-10202-shaderGraphMaterial2

- _Physically based shader is intended for simpler use cases (e.g. constant, non-changing values)_
- _Custom is intended for precise and custom control over 3D objects (e.g. animation, geometry modifiers, special effects, etc._)

ShaderGraphMaterial supports two main types of shaders, which we call Physically Based and Custom. Physically Based is a basic PBR shader. we configure this shader by providing constant, nonchanging values, like colors or images, for each property. Custom shaders, on the other hand, give us precise and custom control over the appearance of 3D objects.  
Custom shaders can incorporate animation, adjust object's geometry, and create special effects on the surface of the object, like a sparkly paint look.
we can build ShaderGraphMaterials using the Shader Graph editor right inside Reality Composer Pro.

![shaderGraphMaterial][shaderGraphMaterial3]

[shaderGraphMaterial3]: WWDC23-10202-shaderGraphMaterial3

<!-- The timestamp was redundant because already set above...-->
# Material editing

Materials in xrOS can be created using Reality Composer Pro

![Reality Composer Pro][realityComposerPro]

[realityComposerPro]: WWDC23-10202-realityComposerPro

For an introduction to the Reality composer pro app before starting with materials, please watch:
[Meet Reality Composer Pro](https://developer.apple.com/videos/play/wwdc2023/10083)

The session will use the Yosemite Valley model and apply a topographical map appearance to it. All the objects we don't need for this session have been hidden using the Deactivate command.

![the Yosemite Valley model in Reality Composer Pro][yosemiteModel]

[yosemiteModel]: WWDC23-10202-yosemiteModel

## Topography map
Let's add a topography feature. We'll add a material showing the topography of our model by displaying contour lines along the slope of the terrain. It'll look similar to the topographical map that we might have come across when planning a hike. 

![the Yosemite Valley model finished version in Reality Composer Pro][yosemiteModelTopographic]

[yosemiteModelTopographic]: WWDC23-10202-yosemiteModelTopographic

Let's create a custom material containing a shader that will draw contour lines on our terrain. We want to draw lines on our terrain through all the areas that have the same elevation, like in this diagram. This blue line, for example, shows all points on the terrain at 1000 meters of elevation. Seen from above, it will look something like the diagram on the right.

![the Yosemite Valley model in Reality Composer Pro][yosemiteModel2]

[yosemiteModel2]: WWDC23-10202-yosemiteModel2

Add a material by clicking the `+` icon inside the Project Browser. Under Materials, notice our two shader types: Physically Based and Custom. Choose Custom.

![the Yosemite Valley model in Reality Composer Pro][yosemiteModel3]

[yosemiteModel3]: WWDC23-10202-yosemiteModel3

This reveals the Shader Graph of our editor.

![the Yosemite Valley model in Reality Composer Pro][yosemiteModel4]

[yosemiteModel4]: WWDC23-10202-yosemiteModel4

New custom shaders start with two nodes, the surface node in purple and the outputs node in blue. the material's active surface is the one connected to the custom surface input of the outputs node. The inputs on the surface are how we set our shader's physically based, or PBR, parameters. Base color, for example.

Let's give the material a descriptive name. TopographyMaterial sounds good. An important step is to assign our new material to our Yosemite Valley model. Select it in the project hierarchy. In the inspector, under Material Bindings, choose TopographyMaterial from the Binding menu.

![the Yosemite Valley model][yosemiteModel5]

[yosemiteModel5]: WWDC23-10202-yosemiteModel5

Notice the model changes from color to a simple gray. Our material is working, but it doesn't do anything interesting yet. Select TopographyMaterial from the hierarchy to return to the Shader Graph editor. We'll connect some nodes to our surface inputs to draw stripes on our model in the right places.

We'll illustrate how our material will work with this example terrain. We need our material to decide where to draw topographical lines based on its location on our model. So first we'll add a position node to our material. This node returns our rendering position in 3D space. We're only interested in the height of position, so we'll also add a node called separate to extract just the position's Y-coordinate. Separate returns the Y-coordinate position, which increases with terrain height.

![the Yosemite Valley model][yosemiteModel6]

[yosemiteModel6]: WWDC23-10202-yosemiteModel6

Let's add these first two nodes to our material in Reality Composer Pro. Double-click on the background of the editor to add nodes. This brings up the New Node picker. we can browse through the list of all the available nodes, or search for nodes by name or keyword. Let's type "position" and select the Position node from the list to insert it into our shader. Position outputs the location in 3D space where the material is being rendered.

![the Yosemite Valley model][yosemiteModel7]

[yosemiteModel7]: WWDC23-10202-yosemiteModel7

Our material varies with height, so let's add a separate node to extract the Y component of position. Double-click the background to bring up the New Node picker again and add a Separate 3 node. To make connections in the editor, just click and drag from node outputs to node inputs like this. 

![the Yosemite Valley model][yosemiteModel8]

[yosemiteModel8]: WWDC23-10202-yosemiteModel8

These two nodes combined give us the height of the terrain. Next, we'll take the output of the separate node and pass it to a modulo node. Modulo gives the remainder of dividing two values. We'll use modulo to divide height by our desired topographical line spacing. The result looks like this. we can see height has been divided into bands. The height values inside each range start at 0 and increase to that range's height. This will be important for our next step.

![the Yosemite Valley model][yosemiteModel9]

[yosemiteModel9]: WWDC23-10202-yosemiteModel9

Let's add the modulo node to our material in Reality Composer Pro. Instead of double-clicking to add a node and then connecting it, we can drag a new connection to an empty space to create a node that's already connected in one step. In the New Node picker, type "modulo" and click to insert a Modulo node. Modulo has two inputs. The first is the dividend and the second is the divisor. we can conveniently set constant values on inputs in the inspector instead of connecting nodes. In the inspector, change the second argument to 0.1. This is the divisor and sets the width of our height ranges. 

![the Yosemite Valley model][yosemiteModel10]

[yosemiteModel10]: WWDC23-10202-yosemiteModel10

We have just one more step to go before we can see our output. We'll use an ifgreater node to determine where our repeating values fall in narrow height bands on our terrain. The ifgreater node will return a value representing one of the two band colors we see on screen, depending on the result of a comparison. When the height is greater than our shader topographical line width, we'll choose a background color. And where the height falls within our desired line width, we'll choose our topography line color.

![the Yosemite Valley model][yosemiteModel11]

[yosemiteModel11]: WWDC23-10202-yosemiteModel11

Let's add the ifgreater node to our material and see the results. We want to compare the result of the modulo node to a value we choose for our topographical line width. So we'll add an ifgreater node to do that comparison. Ifgreater compares its two inputs and returns one value when its first input is greater than its second input, and a different value when the first input is less than its second input. This ifgreater node is set to output floating point values, but we want to choose between two colors, one for our terrain and one for our topographical lines. In the inspector, under Type, change this node to output RGB. Next, let's pick our two colors.
 
![the Yosemite Valley model][yosemiteModel12]

[yosemiteModel12]: WWDC23-10202-yosemiteModel12
 
In the inspector, we'll click on the color picker next to True Result and set our terrain color. Let's use white.  
Let's leave the False Result, which is our topographical line color, set to black.

This will give us a lot of contrast with our white terrain. 0.002 is a good value for line width, so in the inspector, we'll set the comparison value to 0.002. we'll connect the ifgreater node's output to our surface's Diffuse Color input. Awesome, now our material is setting the color at every point on our terrain and we've made a topographical lines material.

![the Yosemite Valley model][yosemiteModel13]

[yosemiteModel13]: WWDC23-10202-yosemiteModel13

This section jumps right into a walk-through tutorial on applying topography lines to a geographical diorama. Starts at [4:31](https://developer.apple.com/videos/play/wwdc2023/10202?time=271)

**Important!** make sure we assign our new material to our model in the hierarchy. we do this by selecting custom material from the model's Material Binding dropdown in the inspector [5:48](https://developer.apple.com/videos/play/wwdc2023/10202?time=349).

<!-- The timestamp was redundant because already set above...-->
# Node graphs 
> _Node graphs help simplify complex materials and let we create our own nodes to reuse parts of graphs._

We'll use node graphs to give a material a real topographical map look. Let's add a second set of lines between the current lines of our material.  

Here's what our material looks like so far. 

![the Yosemite Valley model][nodeGraphs]

[nodeGraphs]: WWDC23-10202-nodeGraphs

And here's what it will look like when we add our subdivisions.

![the Yosemite Valley model][nodeGraphs2]

[nodeGraphs2]: WWDC23-10202-nodeGraphs2

We'll start with the four nodes in our material that draw our topographical lines. 

![the Yosemite Valley model][nodeGraphs3]

[nodeGraphs3]: WWDC23-10202-nodeGraphs3

Then we'll compose a node graph from them. This will combine our four nodes into a single node. Finally, we'll create an instance of our node graph to reuse its functionality. One node graph will draw our original set of lines, and the instance of it will draw our second set of lines. 

![the Yosemite Valley model][nodeGraphs4]

[nodeGraphs4]: WWDC23-10202-nodeGraphs4

Let's go back into Reality Composer Pro and build it.  

Here is our topographical lines shader again. Drag to select these four nodes that compute the colors for our lines and where to draw them. 

![the Yosemite Valley model][nodeGraphs5]

[nodeGraphs5]: WWDC23-10202-nodeGraphs5

After selecting them, right-click and choose Compose Node Graph. Now the nodes appear as a single node that I can use inside other graphs. Let's assign a descriptive name to our new node. Let's call it Lines.

![the Yosemite Valley model][nodeGraphs6]

[nodeGraphs6]: WWDC23-10202-nodeGraphs6

We'll create a copy of this node graph to draw our second set of lines. For this, we'll create an instance using the Create Instance command in the hierarchy.

![the Yosemite Valley model][nodeGraphs7]

[nodeGraphs7]: WWDC23-10202-nodeGraphs7

Instances are live copies that adopt any changes made to the original node graph. We'll return to our material by selecting it. Let's call our new instance SecondaryLines.

![the Yosemite Valley model][nodeGraphs8]

[nodeGraphs8]: WWDC23-10202-nodeGraphs8

We want our node graph and its instance to draw lines with different spacings and colors, so we'll add two inputs, called Spacing and Color, to our original node graph to control these properties.
Let's edit our original node graph by double-clicking it. 

we add inputs and outputs to our node graph in the inspector. First let's add an input called Spacing and set its type to Float. 

![the Yosemite Valley model][nodeGraphs9]

[nodeGraphs9]: WWDC23-10202-nodeGraphs9

We'll also add an input called Color to control our topography line color. Set the type of the input to Color3.

![the Yosemite Valley model][nodeGraphs10]

[nodeGraphs10]: WWDC23-10202-nodeGraphs10

I'll connect these to the right places in our graph.

![the Yosemite Valley model][nodeGraphs11]

[nodeGraphs11]: WWDC23-10202-nodeGraphs11

Let's go back to our material by selecting it in the project hierarchy. Notice that our node graph now has the two inputs we created, and that our instance inherited these two new inputs. On the original Lines node graph, set spacing to the values we chose before. And let's choose a smaller spacing and a lighter color for our instance node graph.  
The last step is to combine the outputs from our original and instance node graphs. I've conveniently chosen grayscale colors for both node graphs, so we can combine their colors using a multiply node.

![the Yosemite Valley model][nodeGraphs12]

[nodeGraphs12]: WWDC23-10202-nodeGraphs12

**Quick Tips**
- Multi-select any set of nodes and right-click > Compose Node Graph to make reusable node components [11:21](https://developer.apple.com/videos/play/wwdc2023/10202?time=681).
- In the project hierarchy, right-click > Create Instance to create an instance of our node graph [11:35](https://developer.apple.com/videos/play/wwdc2023/10202?time=695).
- Add inputs and outputs to our node graph in the inspector [12:10](https://developer.apple.com/videos/play/wwdc2023/10202?time=730).

**Instances** of node graphs are live copies that **adopt any changes made to the original node graph**. They inherit inputs and outputs added to the original node graph it points to.

<!-- The timestamp was redundant because already set above...-->
# Geometry modifiers 
> _These are a feature of custom materials we can use to modify our models in real time._ 

We'll replace our static terrain model and recreate it using geometry modifiers and height data. 
Then we'll extend our geometry modifier to dynamically animate between two different terrains: Yosemite Valley and Catalina Island, California. When we're done, we'll have a dynamic terrain material that can animate between our two different locations. Let's see how this is done. All the shaders we've looked at so far are surface shaders. These are shaders that set the physically based, or PBR, attributes for each pixel of our model as it's rendered. Geometry modifiers are similar to surface shaders, but operate on the geometry of our objects instead. In fact, we build them right in the same editor in Reality Composer Pro.

**Reminder** _Surface shaders operate on the PBR attributes while Geometry modifiers operate on the geometry of objects._

This section jumps right into a walk-through tutorial on applying heightmaps to generate geometry, and animating between those heightmaps. Starts at [13:46](https://developer.apple.com/videos/play/wwdc2023/10202?time=826).

Here is an overview of the geometry modifier we're going to build that will create a terrain of Yosemite Valley using its height map data. We'll start with a flat disk model, which contains some plain geometry. This will be our base surface. Next we'll use our terrain height data, which is 2D data about the height at each location of our model, and use a geometry modifier to raise the terrain by the correct amount. This will result in the terrain we want.
Once shown the basic version, we'll demonstrate a version that takes two sets of terrain heights and uses those to animate between our two locations of interest, Yosemite Valley and Catalina Island.

![Geometry modifiers][geometryModifiers]

[geometryModifiers]: WWDC23-10202-geometryModifiers

Here's another view. We'll start with a flat model, and our geometry modifier will move its vertices vertically using data from a 2D height map like this.

![Geometry modifiers][geometryModifiers2]

[geometryModifiers2]: WWDC23-10202-geometryModifiers2

The first thing we'll do is use the Deactivate command to hide the prebuilt Yosemite Valley model.  

![Geometry modifiers][geometryModifiers3]

[geometryModifiers3]: WWDC23-10202-geometryModifiers3

We'll generate the same Yosemite Valley model, but this time using our geometry modifier. I'll start with a flat disk model in my Project Browser. 
We can bring it in by dragging it into the Root entity in the project hierarchy.

![Geometry modifiers][geometryModifiers4]

[geometryModifiers4]: WWDC23-10202-geometryModifiers4

Here we created a new material named DynamicTerrainMaterial and assigned it to the disk. Let's get to work on our geometry modifier. In the Shader Graph editor, we'll need a geometry modifier surface. We'll add this to our material alongside our PBR surface. We drag a connection from the Custom Geometry Modifier input of our output node and choose Geometry Modifier from the New Node UI. 

![Geometry modifiers][geometryModifiers5]

[geometryModifiers5]: WWDC23-10202-geometryModifiers5

Let's first apply our Yosemite Valley image to our surface. To read image data, we'll use an image node. 

![Geometry modifiers][geometryModifiers6]

[geometryModifiers6]: WWDC23-10202-geometryModifiers6

In the inspector, let's assign our Yosemite Valley image to the image node's Filename input. 

![Geometry modifiers][geometryModifiers7]

[geometryModifiers7]: WWDC23-10202-geometryModifiers7

Things look a bit funny since the terrain is flat, but we're going to fix that now. 

![Geometry modifiers][geometryModifiers8]

[geometryModifiers8]: WWDC23-10202-geometryModifiers8

We'll need the height data for our valley. It's in an image file which contains height values instead of color data. Since this data is in an image, we'll read it by adding another image node. Now assign our Yosemite Valley EXR image containing our height data to this new image node's Filename input. 

![Geometry modifiers][geometryModifiers9]

[geometryModifiers9]: WWDC23-10202-geometryModifiers9

Geometry modifiers can move the vertices of our model in any direction, but we're only interested in moving them vertically, so let's insert a Combine 3 node to create a 3D vector with only the Y component set. Now connect this to the GeometryModifier surface's Model Position Offset input, and boom, our flat model has been transformed into Yosemite Valley. 

![Geometry modifiers][geometryModifiers10]

[geometryModifiers10]: WWDC23-10202-geometryModifiers10

There's one more step we need to take. When we move our vertices, we'll also need to set the surface normal vectors of our model to match our new terrain shape. There are ways to compute these from the height data, but today we're going to use an image that contains precomputed normals for this geometry. Let's create another image node to read the normals.

![Geometry modifiers][geometryModifiers11]

[geometryModifiers11]: WWDC23-10202-geometryModifiers11

Since these are precomputed, we'll connect them directly to our surface shader's normal input for better accuracy. Our surface expects normals to have values between -1 and 1, but the normals in our image are between 0 and 1. I've remapped the values from the image using a remap node.

![Geometry modifiers][geometryModifiers12]

[geometryModifiers12]: WWDC23-10202-geometryModifiers12

Now we've created the terrain from a flat geometry using height data. Next, let's make our diorama dynamic and add the ability to morph from one terrain to another. In this case, we'll add an animated transition to change Yosemite Valley to Catalina Island. To achieve this, we'll first add another set of image nodes to our existing geometry shader. The nodes contain the heights, colors, and normals for our Catalina Island terrain. Then we'll add mix nodes to blend between these two sets of heights, normals, and colors. Finally, we'll connect a value to our mix nodes to control blending between our two sets of data. 

![Geometry modifiers][geometryModifiers13]

[geometryModifiers13]: WWDC23-10202-geometryModifiers13

Let's build this in Reality Composer Pro now. OK, here's our material so far.

![Geometry modifiers][geometryModifiers14]

[geometryModifiers14]: WWDC23-10202-geometryModifiers14

Let's add another set of image nodes containing the same data -- heights, colors, and normals -- for Catalina Island instead of Yosemite Valley.

![Geometry modifiers][geometryModifiers15]

[geometryModifiers15]: WWDC23-10202-geometryModifiers15

Next we'll add some mix nodes to blend between our two colors, heights, and normals. 

![Geometry modifiers][geometryModifiers16]

[geometryModifiers16]: WWDC23-10202-geometryModifiers16

Finally let's connect a 0 to 1 constant to our mix nodes, which will control which terrain is shown. we'll notice when I set my mixing constant to 1, the terrain shows Catalina Island. When I set the mixing constant to 0, we see Yosemite Valley.

![Geometry modifiers][geometryModifiers17]

[geometryModifiers17]: WWDC23-10202-geometryModifiers17

Now that we have a material that can transition between our two different terrains, let's make this transition progress value changeable from our Swift code. We'll use the Promote command to convert our progress value into an input on our material. 

![Geometry modifiers][geometryModifiers18]

[geometryModifiers18]: WWDC23-10202-geometryModifiers18

Inputs on materials become properties of the material that can be accessed from the Swift code. Now our material is ready to be used in our Dioramas Swift app. Here's the final version, which combines our topography lines and our dynamic terrain. This version has some additional refinements, like anti-aliased topography lines and ambient occlusion maps, all added using Shader Graph. 

![Geometry modifiers][geometryModifiers20]

[geometryModifiers20]: WWDC23-10202-geometryModifiers20

![Geometry modifiers][geometryModifiers19]

[geometryModifiers19]: WWDC23-10202-geometryModifiers19

Check those out in the sample for this session.


## Resources
[Diorama - Code Sample](https://developer.apple.com/documentation/visionOS/diorama)  
[Have a question? Ask with tag wwdc2023-10202](https://developer.apple.com/forums/create/question?&tag1=760030&tag2=796030)  
[Search the forums for tag wwdc2023-10202](https://developer.apple.com/forums/tags/wwdc2023-10202)

## Related Videos

[Build great games for spatial computing](https://developer.apple.com/videos/play/wwdc2023/10096)  
[Explore rendering for spatial computing](https://developer.apple.com/videos/play/wwdc2023/10095)  
[Explore the USD ecosystem](https://developer.apple.com/videos/play/wwdc2023/10086)  
[Meet Reality Composer Pro](https://developer.apple.com/videos/play/wwdc2023/10083)  
[Optimize app power and performance for spatial computing](https://developer.apple.com/videos/play/wwdc2023/10100)  
[Work with Reality Composer Pro content in Xcode](https://developer.apple.com/videos/play/wwdc2023/10273)
