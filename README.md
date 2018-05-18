About
======

DoubleBuffering is a low-level library for efficient usge of GPU resources and rendering data on screen as fast as possible. For example, our 3D-engine with dynamic lighting, as well as a small game based on raycasting are realised with this library. And they give out acceptable FPS values:

![](http://i.imgur.com/YgL9fCo.png?1)

![](http://i.imgur.com/yHEwiNo.png?1)

The main concept of library is very simple: there are 6 tables stored in the RAM and containing information about pixels on the screen. The first 3 tables stores what is displayed at the moment, and the other 3 tables - what user is drawing to screen buffer in this moment. After performing all the drawing operations, user calls the buffer.**drawChanges** () function: library will automatically detect changed pixels, groups them into an temporary buffer by color values to reduce GPU calls, and finnaly will display result on screen.

Compared with standard linear rendering, the rendering with this library time is reduced by hundreds and thousands of times. The picture below shows this:

![](https://i.imgur.com/k9wEQTO.png)

The price of such speed is an increased consumption of RAM. To minimize it, library uses the one-dimensional structure of the pixel data tables instead of the three-dimensional one. To obtain pixel data, special methods are used that convert screen coordinates to screen buffer indices and vice versa, more about this is written below in **Auxiliary methods** section.

In addition, this library doesn't access any Lua table directly, replacing them with constants aliases and avoiding calculation of hashes: for example, instead of instead of gpu.**setBackground**(...) library uses **GPUSetBackground**(...). With a competent implementation, this extremeley increases perfomance without loading Lua GC.

| Contents |
| ----- |
| [Installation](#installation) |
| [Main methods:](#main-methods) |
| [   buffer.getResolution](#buffergetresolution-int-width-int-height) |
| [   buffer.setResolution](#buffersetresolution-width-height-) |
| [   buffer.bindScreen](#bufferbindscreen-address-) |
| [   buffer.bindGPU](#bufferbindgpu-address-) |
| [Rendering methods:](#rendering-methods) |
| [   buffer.drawChanges](#bufferdrawchanges-force-) |
| [   buffer.setDrawLimit](#buffersetdrawlimit-x1-y1-x2-y2-) |
| [   buffer.getDrawLimit](#buffergetdrawlimit--int-x1-int-y1-int-x2-int-y2) |
| [   buffer.copy](#buffercopy-x-y-width-height--table-pixeldata) |
| [   buffer.paste](#bufferpaste-x-y-pixeldata-) |
| [   buffer.set](#bufferpaste-x-y-pixeldata-) |
| [   buffer.get](#bufferpaste-x-y-pixeldata-) |
| [   buffer.drawRectangle](#bufferdrawrectangle-x-y-width-height-background-foreground-symbol-transparency-) |
| [   buffer.clear](#bufferclear-color-transparency-) |
| [   buffer.drawText](#bufferdrawtext-x-y-color-text-transparency-) |
| [   buffer.drawImage](#bufferdrawimage-x-y-picture-) |
| [   buffer.drawLine](#bufferdrawline-x1-y1-x2-y2-background-foreground-symbol-) |
| [   buffer.drawEllipse](#bufferdrawellipse-centerx-centery-radiusx-radiusy-background-foreground-symbol-) |
| [Semi-pixel rendering methods:](#semipixel-rendering-methods) |
| [   buffer.semiPixelSet](#buffersemipixelset-x-y-color-) |
| [   buffer.drawSemiPixelSquare](#bufferdrawsemipixelrectangle-x-y-width-height-color-) |
| [   buffer.drawSemiPixelLine](#bufferdrawsemipixelline-x1-y1-x2-y2-color-) |
| [   buffer.drawSemiPixelEllipse](#bufferdrawSemiPixelEllipse-centerX-centerY-radiusX-radiusY-color-) |
| [   buffer.drawSemiPixelCurve](#bufferdrawsemipixelcurve-points-color-accuracy-) |
| [Auxiliary methods:](#auxiliary-methods) |
| [   buffer.flush](#bufferflush-width-height-) |
| [   buffer.getIndexByCoordinates](#buffergetindexbycoordinates-x-y--int-index) |
| [   buffer.getCoordinatesByIndex](#buffergetcoordinatesbyindex-index--int-x-int-y) |
| [   buffer.rawSet](#bufferrawset-index-background-foreground-symbol-) |
| [   buffer.rawGet](#bufferrawget-index--int-background-int-foreground-string-symbol) |
| [   buffer.getCurrentFrameTables](#buffergetcurrentframetables-table-currentframebackgrounds-table-currentframeforegrounds-table-currentframesymbols) |
| [   buffer.getNewFrameTables](#buffergetnewframetables-table-newframebackgrounds-table-newframeforegrounds-table-newframesymbols) |

Installation
======
The easiest way is to use automatic [installer](https://pastebin.com/vTM8nbSZ) that will download all necessary files for you via single command:

    pastebin run vTM8nbSZ
    
However, you can download dependencies manually, if required. They are listed in the following table:

| Dependency | Description | Documentation |
| ------ | ------ | ------ |
| *[doubleBuffering](https://github.com/IgorTimofeev/DoubleBuffering/blob/master/DoubleBuffering.lua)* | This library | - | 
| *[advancedLua](https://github.com/IgorTimofeev/AdvancedLua/blob/master/AdvancedLua.lua)* | Addition to standard Lua libraries with a variety of functions: fast table serialization, text wrapping, binary data processing, etc. | [https://github.com/Igor...](https://github.com/IgorTimofeev/AdvancedLua/blob/master/README.md) | 
| *[color](https://github.com/IgorTimofeev/Color/blob/master/Color.lua)* | Extrusion and packaging of color channels, conversion of the RGB color model to HSB and vice versa, the implementation of alpha blending, the generation of color transitions and the conversion of color into an 8-bit format for the OpenComputers palette | [https://github.com/Igor...](https://github.com/IgorTimofeev/Color/blob/master/README.md) | 
| *[image](https://github.com/IgorTimofeev/Image/blob/master/Image.lua)* | Implementation of the image standard for OpenComputers and basic methods of their processing: transpose, crop, rotate, reflection, etc. | - | 
| *[OCIF](https://github.com/IgorTimofeev/Image/blob/master/OCIF.lua)* | The OpenComputers Image Format module for the image library, written with regard to the features of the mod and realizing effective compression of pixel data | - | 

Main methods
======

buffer.**getResolution**(): *int* width, *int* height
-----------------------------------------------------------
Get screen buffer resolution. There's also buffer.**getWidth**() and buffer.**getHeight**() methods for your comfort.

buffer.**setResolution**( width, height )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | width | Screen buffer width |
| *int* | height | Screen buffer height |

Set screen buffer and GPU resolution. Content of buffer will be cleared with black pixels and whitespace symbol.

buffer.**bindScreen**( address )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | address | Screen component address |

Bind the GPU used by library to the specified screen component address. Content of buffer will be cleared with black pixels and whitespace symbol.

buffer.**bindGPU**( address )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *string* | address | GPU component address |

Set the GPU component address is used by library. Content of buffer will be cleared with black pixels and whitespace symbol.

buffer.**getGPUProxy**( ): *table* GPUProxy
-----------------------------------------------------------
Get a pointer to currently bound GPU component proxy.

Rendering methods
======

buffer.**drawChanges**( [force] )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| [*boolean* | force] | Force content drawing |

Draw contents of screen buffer on screen. If optional argument **force** is specified, then the contents of screen buffer will be drawn completely and regardless of the changed pixels.

buffer.**setDrawLimit**( x1, y1, x2, y2 )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x1 | First point coordinate of draw limit by x-axis |
| *int* | y1 | First point coordinate of draw limit by y-axis |
| *int* | x2 | Second point coordinate of draw limit by x-axis |
| *int* | y2 | Second point coordinate of draw limit by y-axis |

Set buffer draw limit to the specified values. In this case, any operations that go beyond the limits will be ignored. By default, the buffer always has a drawing limit in the ranges **x ∈ [1; buffer.width]** and **y ∈ [1; buffer.height]** 

buffer.**getDrawLimit**( ): *int* x1, *int* y1, *int* x2, *int* y2
-----------------------------------------------------------
Get currently set draw limit

buffer.**copy**( x, y, width, height ): *table* pixelData
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Copied area coordinate by x-axis |
| *int* | y | Copied area coordinate by y-axis |
| *int* | width | Copied area width |
| *int* | height | Copied area height |

Copy content of specified area from screen buffer and return it as a table. Later it can be used with buffer.**paste**(...).

buffer.**paste**( x, y, pixelData )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Paste coordinate by x-axis |
| *int* | y | Paste coordinate by y-axis |
| *table* | pixelData | Table with copied screen buffer data |

Paste the copied contents of screen buffer to the specified coordinates.

buffer.**set**( x, y, background, foreground, symbol )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Screen coordinate by x-axis |
| *int* | y | Screen coordinate by x-axis |
| *int* | background | Background color |
| *int* | foreground | Text color |
| *string* | symbol | Symbol |

Set value of specified pixel on screen.

buffer.**get**( x, y ): *int* background, *int* foreground, *string* symbol
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Screen coordinate by x-axis |
| *int* | y | Screen coordinate by x-axis |

Get value of specified pixel on screen.

buffer.**drawRectangle**( x, y, width, height, background, foreground, symbol, transparency )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Rectangle coordinate by x-axis |
| *int* | y | Rectangle coordinate by x-axis |
| *int* | width | Rectangle width |
| *int* | height | Rectangle height |
| *int* | background | Rectangle background color |
| *int* | foreground | Rectangle text color |
| *string* | symbol | The symbol that will fill the rectangle |
| [*float* [0.0; 1.0] | transparency] | Optional background transparency |

Fill the rectangular area with the specified pixel data. If optional transparency parameter is specified, the rectangle will "cover" existing pixel data, like a glass.

buffer.**clear**( [color, transparency] )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| [*int* | background] | Optional background color |
| [*float* [0.0; 1.0] | transparency] | Optional background transparency |

It works like buffer.**square**(...), but it applies immediately to all the pixels in the buffer. If arguments are not passed, then the buffer is filled with the standard black color and the whitespace symbol.

buffer.**drawText**( x, y, color, text, transparency )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Text coordinate by x-axis |
| *int* | y | Text coordinate by y-axis|
| *int* | foreground | Text color |
| *string* | text | Text |
| [*float* [0.0; 1.0] | transparency] | Optional text transparency |

Draw the text of the specified color. The background color under text will remain the same. It is also possible to set the transparency of the text.

buffer.**drawImage**( x, y, picture )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Image coordinate by x-axis |
| *int* | y | Image coordinate by y-axis|
| *table* | picture | Loaded image |

Draw image that was loaded earlier via image.**load**(...) method. The alpha channel of image is also supported.

buffer.**drawLine**( x1, y1, x2, y2, background, foreground, symbol )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x1 | First point coordinate by x-axis |
| *int* | y1 | First point coordinate by y-axis |
| *int* | x2 | Second point coordinate by x-axis |
| *int* | y2 | Second point coordinate by y-axis |
| *int* | background | Line background color |
| *int* | foreground | Line background color |
| *string* | symbol | Line symbol |

Draw a line with specified pixel data from first point to second.

buffer.**drawEllipse**( centerX, centerY, radiusX, radiusY, background, foreground, symbol )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | centerX | Ellipse middle point by x-axis |
| *int* | centerY | Ellipse middle point by x-axis |
| *int* | radiusX | Ellipse radius by x-axis |
| *int* | radiusY | Ellipse radius by x-axis |
| *int* | background | Ellipse background color |
| *int* | foreground | Ellipse background color |
| *string* | symbol | Ellipse symbol |

Draw ellipse with specified pixel data.

Semi-pixel rendering methods
======

Allsemi-pixel methods allow to avoid the effect of doubling pixel height of the console pseudographics using special symbols like "▄". In this case, the transmitted coordinates along the **Y** axis must belong to the interval **[0; buffer.height * 2]**.

buffer.**semiPixelSet**( x, y, color )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x1 | Coordinate by x-axis |
| *int* | y1 | Coordinate by y-axis |
| *int* | color | Pixel color |

Set semi-pixel value in specified coordinates.

buffer.**drawSemiPixelRectangle**( x, y, width, height, color )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Rectangle coordinate by x-axis |
| *int* | y | Rectangle coordinate by y-axis |
| *int* | width | Rectangle width |
| *int* | height | Rectangle height |
| *int* | color | Rectangle color |

Draw semi-pixel rectangle.

buffer.**drawSemiPixelLine**( x1, y1, x2, y2, color )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x1 | First point coordinate by x-axis |
| *int* | y1 | First point coordinate by y-axis |
| *int* | x2 | Second point coordinate by x-axis |
| *int* | y2 | Second point coordinate by y-axis |
| *int* | color | Line color |

Rasterize a semi-pixel line witch specified color.

buffer.**drawSemiPixelEllipse**( centerX, centerY, radiusX, radiusY, color )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | centerX | Ellipse middle point by x-axis |
| *int* | centerY | Ellipse middle point by x-axis |
| *int* | radiusX | Ellipse radius by x-axis |
| *int* | radiusY | Ellipse radius by x-axis |
| *int* | color | Ellipse color |

Draw semi-pixel ellipse with specified color.

buffer.**drawSemiPixelCurve**( points, color, accuracy )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *table* | points | Table with structure ```{{x = 32, y = 2}, {x = 2, y = 2}, {x = 2, y = 98}}``` that contains a set of points for drawing curve |
| *int* | color | Curve color |
| *float* | accuracy | Curve accuracy. Less = more accurate |

Draw the [Bezier Curve](https://en.wikipedia.org/wiki/B%C3%A9zier_curve) with specified color

Auxiliary methods
======

The following methods are used by the library itself or by applications that require maximum performance and calculate the pixel data of the buffer manually. In most cases, they do not come in handy, but they are listed *just in case*.

buffer.**flush**( [width, height] )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | width | New screen buffer width |
| *int* | height | New screen buffer width |

Set screen buffer resolution to the specified one and fill it with black pixels and whitespace stringacter. Unlike buffer.**setResolution**() it does not change the current resolution of the GPU. If optional arguments are **not specified**, then the buffer size becomes equivalent to the current GPU resolution.

buffer.**getIndex*( x, y ): **int** index
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | x | Screen coordinate by x-axis |
| *int* | y | Screen coordinate by y-axis |

Convert screen coordinates to the screen buffer index. For example, a **2x1** pixel has a buffer index equals **4**, and a pixel of **3x1** has a buffer index equals **7**.

buffer.**getCoordinates**( index ): **int** x, **int** y
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | index | Screen buffer index |

Convert screen buffer index to it's screen coordinates.

buffer.**rawSet**( index, background, foreground, symbol )
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | index | Screen buffer index |
| *int* | background | Background color |
| *int* | foreground | Text color |
| *string* | symbol | Symbol |

Set specified data values to pixel with specified index.

buffer.**rawGet**( index ): **int** background, **int** foreground, **string** symbol
-----------------------------------------------------------
| Type | Parameter | Description |
| ------ | ------ | ------ |
| *int* | index | Screen buffer index |

Get data values of pixel with specified index.

buffer.**getCurrentFrameTables**(): **table** currentFrameBackgrounds, **table** currentFrameForegrounds, **table** currentFrameSymbols
-----------------------------------------------------------

Get current screen buffer frames (that is displayed on screen) that contains pixel data. This method is used in rare cases where maximum performance and manual buffer changing pixel-by-pixel is required.

buffer.**getNewFrameTables**(): **table** newFrameBackgrounds, **table** newFrameForegrounds, **table** newFrameSymbols
-----------------------------------------------------------

Works like buffer.**getCurrentFrameTables**(), but returns frames that user is changing in realtime (before calling buffer.**drawChanges**())

Practical example
======

```lua
-- Import required libraries
local buffer = require("doubleBuffering")
local image = require("image")

--------------------------------------------------------------------------------

-- Load image from file and draw it to screen buffer
buffer.drawImage(1, 1, image.load("/MineOS/Pictures/Raspberry.pic"))
-- Fill buffer with black color and transparency set to 0.6 to make image "darker"
buffer.clear(0x0, 0.6)

-- Draw 10 rectangles filled with random color
local x, y, xStep, yStep = 2, 2, 4, 2
for i = 1, 10 do
	buffer.drawRectangle(x, y, 6, 3, math.random(0x0, 0xFFFFFF), 0x0, " ")
	x, y = x + xStep, y + yStep
end

-- Draw yellow semi-pixel ellipse
buffer.drawSemiPixelEllipse(22, 22, 10, 10, 0xFFDB40)
-- Draw yellow semi-pixel line
buffer.drawSemiPixelLine(2, 36, 35, 3, 0xFFFFFF)
-- Draw green bezier curve with accuracy set to 0.01
buffer.drawSemiPixelCurve(
	{
		{ x = 2, y = 63},
		{ x = 63, y = 63},
		{ x = 63, y = 2}
	},
	0x44FF44,
	0.01
)

-- Draw changed pixels on screen
buffer.drawChanges()
```

Result: 

![](http://i.imgur.com/wvu0jeh.png?1)
