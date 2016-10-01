#### _Implicit Closure of Filled Subpaths_
<a name="Implicit_Closure_of_Filled_Subpaths"></a>
When filling a path, any subpaths that do not end with a `CLOSE_PATH` segment command (_i.e_., that are terminated with a `MOVE_TO_ABS` or `MOVE_TO_REL` segment command, or that contain the final segment of the path) are implicitly closed, without affecting the position of any other vertices of the path or the \left( sx, sy\right), \left( px, py\right) or \left( ox, oy\right) variables. For example, consider the sequence of segment commands:

`MOVE_TO_ABS 0, 0`; `LINE_TO_ABS 10, 10`; `LINE_TO_ABS 10, 0`
`MOVE_TO_REL 10, 2`; `LINE_TO_ABS 30, 12`; `LINE_TO_ABS 30, 2`

If filled, this sequence will result in one filled triangle with vertices $(0, 0)$, $(10, 10)$, and $(10, 0)$ and another filled triangle with vertices $(20, 2)$, $(30, 12)$, and $(30, 2)$. Note that the implicit closure of the initial subpath prior to the `MOVE_TO_REL` segment command has no effect on the starting coordinate of the second triangle; it is computed by adding the relative offset $(10, 2)$ to the final coordinate of the previous segment $(10, 0)$ to obtain $(20, 2)$ and is not altered by the (virtual) insertion of the line connecting the first subpath’s final vertex $(10, 0)$ to its initial vertex $(0, 0)$). Figure 10 illustrates this process, with the resulting filled areas highlighted. When stroking a path, no implicit closure takes place, as shown in Figure 11. Implicit closure affects only the output when filling a path, and does not alter the path data in any way.
![figure10](figures/figure10.PNG)
_Figure 10: Implicit Closure of Filled Paths_
<a name="Figure10:Implicit_Closure_of_Filled_Paths"></a>

![figure11](figures/figure11.PNG)
_Figure 11: Stroked Paths Have No Implicit Closure_
<a name="Figure11:Stroked_Paths_Have_No_Implicit_Closure"></a>

### _8.7.2 Stroking Paths_
<a name="Stroking_Paths"></a>
Stroking a path consists of “widening” the edges of the path using a straight-line pen held perpendicularly to the path. At the start and end vertices of the path, an additional end-cap style is applied. At interior vertices of the path, a line join style is applied. At a cusp of a Bézier segment, the pen is rotated smoothly between the incoming and outgoing tangents.

Conceptually, stroking of a path is performed in two steps. First, the stroke parameters are applied in the user coordinate system to form a new shape representing the end result of dashing, widening the path, and applying the end cap and line join styles. Second, a path is created that defines the outline of this stroked shape. This path is transformed using the path-user-to-surface transformation (possibly involving shape distortions due to non-uniform scaling or shearing). Finally, the resulting path is filled with paint in exactly the same manner as when filling a user-defined path using the non-zero fill rule.

Stroking a path applies a single “layer” of paint, regardless of any intersections between portions of the thickened path. Figure 12 illustrates this principle. A single stroke (above) is drawn with a black color and an alpha value of 50%, compared with two separate strokes (below) drawn with the same color and alpha values. The single stroke produces a shape with a uniform color of 50% gray, as if a single layer of translucent paint has been applied, even where portions of the path overlap one another. By contrast, the separate strokes produce two applications of the translucent paint in the area of overlap, resulting in a darkened area.
![figure12](figures/figure12.PNG)
_Figure 12: Each Stroke Applies a Single Layer of Paint_
<a name="Figure12:_Each_Stroke_Applies_a_Single_Layer_of_Paint"></a>

### _8.7.3 Stroke Parameters_
<a name="Stroke_Parameters"></a>
Stroking a path involves the following parameters, set on a context:
* Line width in user coordinate system units
* End cap style – one of Butt, Round, or Square
* Line join style – one of Miter, Round, or Bevel
* Miter limit – if using Miter join style
* Dash pattern – array of dash on/off lengths in user units
* Dash phase – initial offset into the dash pattern

These parameters are set on the current context using the variants of the **vgSet** function. The values most recently set prior to calling **vgDrawPath** (see Section 8.8) are applied to generate the stroke.

#### _End Cap Styles_
<a name="End_Cap_Styles"></a>
Figure 13 illustrates the Butt (top), Round (center), and Square (bottom) end cap styles applied to a path consisting of a single line segment. Figure 14 highlights the additional geometry created by the end caps. The Butt end cap style terminates each segment with a line perpendicular to the tangent at each endpoint. The Round end cap style appends a semicircle with a diameter equal to the line width centered around each endpoint. The Square end cap style appends a rectangle with two sides of length equal to the line width perpendicular to the tangent, and two sides of length equal to half the line width parallel to the tangent, at each endpoint. The outgoing tangent is used at the left endpoint and the incoming tangent is used at the right endpoint.
![figure13](figures/figure13.PNG)
_Figure 13: End Cap Styles_
<a name="Figure13:End_Cap_Styles"></a>

![figure14](figures/figure14.PNG)
_Figure 14: End Cap Styles with Additional Geometry Highlighted_
<a name="Figure14:End_Cap_Styles_with_Additional_Geometry_Highlighted"></a>

#### _Line Join Styles_
<a name="Line_Join_Styles"></a>
Figure 15 illustrates the Bevel (left), Round (center), and Miter (right) line join styles applied to a pair of line segments. Figure 16 highlights the additional geometry created by the line joins. The Bevel join style appends a triangle with two vertices at the outer endpoints of the two “fattened” lines and a third vertex at the intersection point of the two original lines. The Round join style appends a wedge-shaped portion of a circle, centered at the intersection point of the two original lines, having a radius equal to half the line width. The Miter join style appends a trapezoid with one vertex at the intersection point of the two original lines, two adjacent vertices at the outer endpoints of the two “fattened” lines and a fourth vertex at the extrapolated intersection point of the outer perimeters of the two “fattened” lines.

When stroking using the Miter join style, the _miter length_ (_i.e_., the length between the intersection points of the inner and outer perimeters of the two “fattened” lines) is compared to the product of the user-set miter limit and the line width. If the miter length exceeds this product, the Miter join is not drawn and a Bevel join is substituted.
![figure15](figures/figure15.PNG)
_Figure 15: Line Join Styles_
<a name="Figure15:Line_Join_Styles"></a>

![figure16](figures/figure16.PNG)
_Figure 16: Line Join Styles with Additional Geometry Highlighted_
<a name="Figure16:Line_Join_Styles_with_Additional_Geometry_Highlighted"></a>

#### _Miter Length_
<a name="Miter_Length"></a>
The ratio of miter length to line width may be computed directly from the angle \theta between the two line segments being joined as 1/\sin { \left( \theta /2 \right) }. A number of angles with their corresponding miter limits for a line width of 1 are shown in Table 9.

| _Angle (degrees)_ | _Miter Limit_ | _Angle (degrees)_ |  _Miter Limit_ |
| :---: | :---: | :---: | :---: |
| 10 | 11.47 | 45 | 2.61 |
| 11.47 | 10 | 60 | 2 |
| 23 | 5 | 90 | 1.41 |
| 28.95 | 4 | 120 | 1.15 |
| 30 | 3.86 | 150 | 1.03 |
| 38.94 | 3 | 180 | 1 |
_Table 9: Corresponding Angles and Miter_
<a name="Table9:Corresponding_Angles_and_Miter"></a>

#### _Dashing_
<a name="Dashing"></a>
The dash pattern consists of a sequence of lengths of alternating “on” and “off” dash segments. The first value of the dash array defines the length, in user coordinates, of the first “on” dash segment. The second value defines the length of the following “off” segment. Each subsequent pair of values defines one “on” and one “off” segment.

The dash phase defines the starting point in the dash pattern that is associated with the start of the first segment of the path. For example, if the dash pattern is [ 10 20 30 40 ] and the dash phase is 35, the path will be stroked with an “on” segment of length 25 (skipping the first “on” segment of length 10, the following “off” segment of length 20, and the first 5 units of the next “on” segment), followed by an “off” segment of length 40. The pattern will then repeat from the beginning, with an “on” segment of length 10, an “off” segment of length 20, an “on” segment of length 30, etc. Figure 17 illustrates this dash pattern.

Conceptually, dashing is performed by breaking the path into a set of subpaths according to the dash pattern. Each subpath is then drawn independently using the end cap, line join style, and miter limit that were set for the path as a whole.

Dashes of length 0 are drawn only if the end cap style is `VG_CAP_ROUND` or `VG_CAP_SQUARE`. The incoming and outgoing tangents (which may differ if the dash falls at a vertex of the path) are evaluated at the point, using the **vgPointAlongPath** algorithm. The end caps are drawn using the orientation of each tangent, and a join is drawn between them if the tangent directions differ. If the end cap style is `VG_CAP_BUTT`, nothing will be drawn.

A dash, or space between dashes, with length less than 0 is treated as having a length of 0.

A negative dash phase is equivalent to the positive phase obtained by adding a suitable multiple of the dash pattern length.

![figure17](figures/figure17.PNG)
_Figure 17: Dash Pattern and Phase Example_
<a name="Figure17:Dash_Pattern_and_Phase_Example"></a>

### _8.7.4 Stroke Generation_
<a name="Stroke_Generation"></a>
The algorithm for generating a stroke is as follows. The steps described in this section conceptually take place in user coordinates, on a copy of the path being stroked in which all relative and implicit coordinates have been converted to absolute coordinates. An initial `MOVE_TO 0,0` segment is added if the path does not begin with a `MOVE_TO`.

The path to be stroked is divided into subpaths, each ending with a `MOVE_TO` or `CLOSE_PATH` segment command or with the final path segment. Subpaths consisting of only a single `MOVE_TO` segment are discarded.

A subpath consisting of a single point (_i.e_., a `MOVE_TO` segment followed by a sequence of `LINE_TO`, `QUAD_TO`, `CUBIC_TO`, and/or `ARC_TO` segments with all control points equal to the current point, possibly followed by a `CLOSE_PATH` segment) is collapsed to a lone vertex, which is marked as an END vertex (for later generation of end caps). A tangent vector of (1, 0) is used for Square end caps.

Subpaths that do not consist only of a single point have any zero-length segments removed.

If a subpath does not end with a `CLOSE_PATH` segment command, its first and last vertices are marked as END vertices. All the internal vertices that begin or end path segments within the subpath, as well as the initial/final vertex if the subpath ends with a `CLOSE_PATH` segment, are marked as JOIN vertices (for later generation of line joins).

Each subpath is processed in turn as described below until all subpaths have been stroked.

If dashing is enabled, the dash pattern and phase are used to break the subpath into a series of smaller subpaths representing the “on” portions of the dash pattern. New vertices are created at the endpoints of each dash subpath and marked as END vertices. The old subpath is discarded and replaced with the dash subpaths for the remainder of the stroke processing. The dash phase is advanced for each subsequent segment by the length of the previous segment (where `CLOSE_PATH` segments are treated as `LINE_TO`segments). If `VG_DASH_PHASE_RESET` is disabled (set to `VG_FALSE`), the final dash phase at the end of the subpath is used as the initial dash phase for the next subpath. Otherwise, the original dash phase is used for all subpaths.

For each END vertex, an end cap is created (if Square or Round end caps have been requested) using the orientation given by the tangent vector. The tangent vector is defined in the same manner as for the **vgPointAlongPath** function (?????see p. 92).

For each JOIN vertex, a line join is created using the orientations given by the tangent vectors of the two adjacent path segments. If Miter joins are being used, the length of the miter is computed and compared to the product of the line width and miter limit; if the miter would be too long, a Bevel join is substituted.

### _8.7.5 Setting Stroke Parameters_
<a name="Setting_Stroke_Parameters"></a>
Setting the line width of a stroke is performed using **vgSetf** with a `paramType`argument of `VG_STROKE_LINE_WIDTH`. A line width less than or equal to 0 prevents stroking from taking place.
```
VGfloat lineWidth;
vgSetf(VG_STROKE_LINE_WIDTH, lineWidth);
```

#### _VGCapStyle_
<a name="VGCapStyle"></a>
The `VGCapStyle` enumeration defines constants for the Butt, Round, and Square end cap styles:
```
typedef enum {
VG_CAP_BUTT = 0x1700,
VG_CAP_ROUND = 0x1701,
VG_CAP_SQUARE = 0x1702
} VGCapStyle;
```
Setting the end cap style is performed using **vgSeti** with a `paramType` argument of `VG_STROKE_CAP_STYLE` and a value from the `VGCapStyle` enumeration.
```
VGCapStyle capStyle;
vgSeti(VG_STROKE_CAP_STYLE, capStyle);
```
#### _VGJoinStyle_
<a name="VGJoinStyle"></a>
The `VGJoinStyle` enumeration defines constants for the Miter, Round, and Bevel line join styles:
```
typedef enum {
VG_JOIN_MITER = 0x1800,
VG_JOIN_ROUND = 0x1801,
VG_JOIN_BEVEL = 0x1802
} VGJoinStyle;
```
Setting the line join style is performed using **vgSeti** with a `paramType` argument of `VG_STROKE_JOIN_STYLE` and a value from the `VGJoinStyle` enum.
```
VGJoinStyle joinStyle;
vgSeti(VG_STROKE_JOIN_STYLE, joinStyle);
```
Setting the miter limit is performed using **vgSetf** with a `paramType` argument of `VG_STROKE_MITER_LIMIT`:
```
VGfloat miterLimit;
vgSetf(VG_STROKE_MITER_LIMIT, miterLimit);
```
Miter limit values less than 1 are silently clamped to 1.


#### _VG_MAX_DASH_COUNT_
<a name="VG_MAX_DASH_COUNT"></a>
The `VG_MAX_DASH_COUNT` parameter contains the maximum number of dash segments that may be supplied for the `VG_STROKE_DASH_PATTERN` parameter. All implementations must must support at least 16 dash segments (8 on/off pairs). If there is no implementation-defined limit, a value of `VG_MAXINT` may be returned. The value may be retrieved by calling **vgGeti**:
```
VGint maxDashCount = vgGeti(VG_MAX_DASH_COUNT);
```

#### _Setting the Dash Pattern_
<a name="Setting_the_Dash_Pattern"></a>
The dash pattern is set using **vgSetfv** with a `paramType` argument of `VG_STROKE_DASH_PATTERN`:
```
VGfloat dashPattern[DASH_COUNT];
VGint count = DASH_COUNT;
vgSetfv(VG_STROKE_DASH_PATTERN, count, dashPattern);
```
Dashing may be disabled by calling **vgSetfv** with a `count` of 0:
```
vgSetfv(VG_STROKE_DASH_PATTERN, 0, NULL);
```
The dash phase is set using **vgSetf** with a `paramType` argument of `VG_STROKE_DASH_PHASE`. The resetting behavior of the dash phase when advancing to a new subpath is set using **vgSeti** with a `paramType` argument of `VG_STROKE_DASH_PHASE_RESET`:
```
VGfloat dashPhase;
VGboolean dashPhaseReset;
vgSetf(VG_STROKE_DASH_PHASE, dashPhase);
vgSeti(VG_STROKE_DASH_PHASE_RESET, dashPhaseReset);
```
If the dash pattern has length 0, dashing is not performed. If the dash pattern has an odd number of elements, the final element is ignored. Note that this behavior is different from that defined by SVG; the SVG behavior may be implemented by duplicating the oddlength dash pattern to obtain one with even length.

If more than `VG_MAX_DASH_COUNT` dashes are specified, those beyond the first `VG_MAX_DASH_COUNT` are discarded immediately (and will not be returned by **vgGet**).

### _8.7.6 Non-Scaling Strokes_
<a name="Non-Scaling_Strokes"></a>
In some cases, applications may wish stroked geometry to appear with a particular stroke width in the surface coordinate system, independent of the current user-to-surface transformation. For example, a stroke representing a road on a map might stay the same width as the user zooms in and out of the map, since the stroke width is intended to indicate the type of road (_e.g_., one-way street, divided road, interstate highway or Autobahn) rather than its true width on the ground.

OpenVG does not provide direct support for this “non-scaling stroke” behavior. However, the behavior may be obtained relatively simply using a combination of features.

If the current user-to-surface transformation consists only of uniform scaling, rotation, and translation (_i.e_., no shearing or non-uniform scaling), then the stroke width may be set to the desired stroke width in drawing surface coordinates, divided by the scaling factor introduced by the transformation. This scaling factor may be known to the application _a priori_, or else it may be computed as the square root of the absolute value of the determinant (sx*sy – shx*shy) of the user-to-surface transformation.

If the user-to-surface transformation includes shearing or non-uniform scaling, the geometry to be stroked must be transformed into surface coordinates prior to stroking. The paint transformation must also be set to the concatenation of the paint-to-user and user-to-surface transformations in order to allow correct painting of the stroked geometry. The following code illustrates this technique:
```
VGPath srcPath; /* Path to be drawn with non-scaling stroke */
VGPath dstPath; /* Path in drawing surface coordinates */
VGfloat strokePaintToUser[9]; /* Paint-to-user transformation */
VGfloat pathUserToSurface[9]; /* User-to-surface transformation */
/* Transform the geometry into surface coordinates. */
vgSeti(VG_MATRIX_MODE, VG_MATRIX_PATH_USER_TO_SURFACE);
vgLoadMatrix(pathUserToSurface);
vgTransformPath(dstPath, srcPath);
/* Use the identity matrix for drawing the stroked path. */
vgLoadIdentity();
/* Set the paint transformation to the concatenation of the
* paint-to-user and user-to-surface transformations.
*/
vgSeti(VG_MATRIX_MODE, VG_MATRIX_FILL_PAINT_TO_USER);
vgLoadMatrix(pathUserToSurface);
vgMultMatrix(strokePaintToUser);
/* Stroke the transformed path. */
vgDrawPath(dstPath, VG_STROKE_PATH);
```

## _8.8 Filling or Stroking a Path_
<a name="Filling_or_Stroking_a_Path"></a>
#### _VGFillRule_
<a name="VGFillRule"></a>
The `VGFillRule` enumeration defines constants for the even/odd and non-zero fill rules.
```
typedef enum {
VG_EVEN_ODD = 0x1900,
VG_NON_ZERO = 0x1901
} VGFillRule;
```
To set the rule for filling, call **vgSeti** with a `type` parameter `value` of `VG_FILL_RULE` and a value parameter defined using a value from the VGFillRule enumeration. When the path is filled, the most recent setting of the fill rule on the current context is used. The fill rule setting has no effect on stroking.
```
VGFillRule fillRule;
vgSeti(VG_FILL_RULE, fillRule);
```
#### _VGPaintMode_
<a name="VGPaintMode"></a>
The `VGPaintMode` enumeration defines constants for stroking and filling paths, to be used by the **vgDrawPath**, **vgSetPaint**, and **vgGetPaint** functions.
```
typedef enum {
VG_STROKE_PATH = (1 << 0),
VG_FILL_PATH = (1 << 1)
} VGPaintMode;
```
#### _vgDrawPath_
<a name="vgDrawPath"></a>
Filling and stroking are performed by the **vgDrawPath** function. The `paintModes` argument is a bitwise OR of values from the `VGPaintMode` enumeration, determining whether the path is to be filled (`VG_FILL_PATH`), stroked (`VG_STROKE_PATH`), or both (`VG_FILL_PATH` | `VG_STROKE_PATH`). If both filling and stroking are to be performed, the path is first filled, then stroked.
```
void vgDrawPath(VGPath path, VGbitfield paintModes)
```

> **_Errors_**
>
> `VG_BAD_HANDLE_ERROR`
> * if `path` is not a valid path handle, or is not shared with the current context
>
> `VG_ILLEGAL_ARGUMENT_ERROR`
> * if `paintModes` is not a valid bitwise OR of values from the `VGPaintMode`
enumeration

#### _Filling a Path_
<a name="Filling_a_Path"></a>
Calling **vgDrawPath** with a `paintModes` argument of `VG_FILL_PATH` causes the given path to be filled, using the paint defined for the `VG_FILL_PATH` paint mode and the current fill rule.

The matrix currently set for the `VG_MATRIX_FILL_PAINT_TO_USER` matrix mode is applied to the paint used to fill the path outline. The matrix currently set for the `VG_MATRIX_PATH_USER_TO_SURFACE` matrix mode is used to transform the outline of the path and the paint into surface coordinates.
```
vgDrawPath(VGPath path, VG_FILL_PATH);
```

#### _Stroking a Path_
<a name="Stroking_a_Path"></a>
Calling **vgDrawPath** with a paintModes argument of `VG_STROKE_PATH` causes the given path to be stroked, using the paint defined for the `VG_STROKE_PATH` paint mode and the current set of stroke parameters.

The matrix currently set for the `VG_MATRIX_STROKE_PAINT_TO_USER` matrix mode is applied to the paint used to fill the stroked path outline. The matrix currently set for the `VG_MATRIX_PATH_USER_TO_SURFACE` matrix mode is used to transform the outline of the stroked path and the paint into surface coordinates.
```
vgDrawPath(VGPath path, VG_STROKE_PATH);
```
The following code sample shows how an application might set stroke parameters using variants of **vgSet**, and stroke a path object (defined elsewhere):
```
VGPath path;
/* Set the line width to 2.5 */
vgSetf(VG_STROKE_LINE_WIDTH, 2.5f);
/* Set the miter limit to 10.5 */
vgSetf(VG_STROKE_MITER_LIMIT, 10.5f);
/* Set the cap style to CAP_SQUARE */
vgSeti(VG_STROKE_CAP_STYLE, VG_CAP_SQUARE);
/* Set the join style to JOIN_MITER */
vgSeti(VG_STROKE_JOIN_STYLE, VG_JOIN_MITER);
/* Set the dash pattern */
VGfloat dashes[] = { 1.0f, 2.0f, 2.0f, 2.0f };
vgSetfv(VG_STROKE_DASH_PATTERN, 4, dashes);
/* Set the dash phase to 0.5 and reset it for every subpath */
vgSetf(VG_STROKE_DASH_PHASE, 0.5f);
vgSeti(VG_STROKE_DASH_PHASE_RESET, VG_TRUE);
/* Stroke the path */
vgDrawPath(path, VG_STROKE_PATH);
```

#### _Filling and Stroking a Path_
<a name="Filling_and_Stroking_a_Path"></a>
Calling **vgDrawPath** with a `paintModes` argument of (`VG_FILL_PATH` | `VG_STROKE_PATH`) causes the given path to be first filled, then stroked, exactly as if **vgDrawPath** were called twice in succession, first with a `paintModes` argument of `VG_FILL_PATH` and second with a `paintModes` argument of `VG_STROKE_PATH`.
```
vgDrawPath(VGPath path, VG_FILL_PATH | VG_STROKE_PATH);
```

# 9 Paint
<a name="Chapter09"></a> <a name="Paint"></a>
Paint defines a color and an alpha value for each pixel being drawn. _Color paint_ defines a constant color for all pixels; _gradient paint_ defines a linear or radial pattern of smoothly varying colors; and _pattern paint_ defines a possibly repeating rectangular pattern of colors based on a source image. It is possible to define new types of paint as extensions.

Paint is defined in its own coordinate system, which is transformed into user coordinates by means of the fill-paint-to-user and stroke-paint-to-user transformations (set using the `VG_MATRIX_FILL_PAINT_TO_USER` and `VG_MATRIX_STROKE_PAINT_TO_USER` matrix modes) depending on whether the current geometry is being filled or stroked.

Given a (fill or stroke) paint-to-user transformation Tp and user-to-surface transformation Tu, the paint color and alpha of a pixel to be drawn with surface coordinates (x, y) is defined by mapping its center point (x + ½, y + ½) through the inverse transformation (Tu ◦ Tp)-1, resulting in a sample point in the paint coordinate space. This transformation must be evaluated with sufficient accuracy to ensure a deviation from the ideal of no more than 1/8 of a pixel along either axis. The paint value nearest that point may be used (point sampling), or paint values from multiple points surrounding the central sample point may be combined to produce an interpolated paint value. Paint color values are processed in premultiplied alpha format during interpolation. The user-to-surface transformation Tu is taken from the path-user-to-surface transformation when fulfilling a **vgDrawPath** call, from the image-user-to-surface transformation when fulfilling a **vgDrawImage** call, or from the glyph-user-to-surface transformation when fulfilling a **vgDrawGlyph** or **vgDrawGlyphs** call.

If the inverse transformation cannot be computed due to a (near-)singularity, no drawing occurs.

## _9.1 Paint Definitions_
<a name="Paint_Definitions"></a>
The OpenVG context stores two paint definitions at a time, one to be applied to stroked shapes and one for filled shapes. This allows the interior of a path to be filled using one type of paint and its outline to be stroked with another kind of paint in a single **vgDrawPath** operation. Initially, default values are used.

#### _VGPaint_
<a name="VGPaint"></a>
`VGPaint` represents an opaque handle to a paint object. A `VGPaint` object is live; changes to a `VGPaint` object (using `vgSetParameter`, or by altering an attached pattern image) attached to a context will immediately affect drawing calls on that context. If a `VGPaint` object is accessed from multiple threads, the application must ensure (using **vgFinish** along with application-level synchronization primitives) that the paint definition is not altered from one context while another context may still be using it for drawing.
```
typedef VGHandle VGPaint;
```

### _9.1.1 Creating and Destroying Paint Objects_

#### _vgCreatePaint_
**vgCreatePaint** creates a new paint object that is initialized to a set of default values and
returns a `VGPaint` handle to it. If insufficient memory is available to allocate a
new object, `VG_INVALID_HANDLE` is returned.
```
VGPaint vgCreatePaint(void)
```

#### _vgDestroyPaint_
The resources associated with a paint object may be deallocated by calling
**vgDestroyPaint**. Following the call, the `paint` handle is no longer valid in any
of the contexts that shared it. If the paint object is currently active in a drawing
context, the context continues to access it until it is replaced or the context is
destroyed.
```
void vgDestroyPaint(VGPaint paint)
```

> **_Errors_**
>
> `VG_BAD_HANDLE_ERROR`
> * if `paint` is not a valid paint handle, or is not shared with the current context


### _9.1.2 Setting the Current Paint_

#### _vgSetPaint_
Paint definitions are set on the current context using the **vgSetPaint** function. The
`paintModes` argument is a bitwise OR of values from the `VGPaintMode`
enumeration, determining whether the paint object is to be used for filling

<!-----116----->
(`VG_FILL_PATH`), stroking (`VG_STROKE_PATH`), or both (`VG_FILL_PATH` |
`VG_STROKE_PATH`). The current `paint` replaces the previously set paint object, if
any, for the given paint mode or modes. If `paint` is equal to `VG_INVALID_HANDLE`,
the previously set paint object for the given mode (if present) is removed and the paint
settings are restored to their default values.
```
void vgSetPaint(VGPaint paint, VGbitfield paintModes)
```

<!-----117----->
> **_Errors_**
>
> `VG_BAD_HANDLE_ERROR`
> * if `paint` is neither a valid paint handle nor equal to `VG_INVALID_HANDLE`,
or is not shared with the current context
>
> `VG_ILLEGAL_ARGUMENT_ERROR`
>* if `paintModes` is not a valid bitwise OR of values from the `VGPaintMode`
enumeration


#### _vgGetPaint_
The **vgGetPaint** function returns the paint object currently set for the given
`paintMode`, or `VG_INVALID_HANDLE` if an error occurs or if no paint object is set
(_i.e_., the default paint is present) on the given context with the given `paintMode`.
```
VGPaint vgGetPaint(VGPaintMode paintMode)
```
> **_Errors_**
>
> `VG_ILLEGAL_ARGUMENT_ERROR`
> * if `paintMode` is not a valid value from the `VGPaintMode` enumeration

### _9.1.3 Setting Paint Parameters_
Paint functionality is controlled by a number of paint parameters that are stored in each
paint object.

#### _VGPaintParamType_
Values from the `VGPaintParamType` enumeration may be used as the `paramType`
argument to **vgSetParameter** and **vgGetParameter** to set and query various features of
a paint object:

<!-----118----->
```
typedef enum {
/* Color paint parameters */
VG_PAINT_TYPE = 0x1A00,
VG_PAINT_COLOR = 0x1A01,
VG_PAINT_COLOR_RAMP_SPREAD_MODE = 0x1A02,
VG_PAINT_COLOR_RAMP_STOPS = 0x1A03,
VG_PAINT_COLOR_RAMP_PREMULTIPLIED = 0x1A07,
/* Linear gradient paint parameters */
VG_PAINT_LINEAR_GRADIENT = 0x1A04,
/* Radial gradient paint parameters */
VG_PAINT_RADIAL_GRADIENT = 0x1A05,
/* Pattern paint parameters */
VG_PAINT_PATTERN_TILING_MODE = 0x1A06
} VGPaintParamType;
```

The default values that are used when no paint object is present (_i.e_., in a newly-created
context or following a call to **vgSetPaint** with a `paint` value of
`VG_INVALID_HANDLE`) are shown in Table 10. These values are also used as the
initial parameter value for a newly created paint object.

<!-----119----->
| _Parameter_ | _Datatype_ | _Default Value_ |
| :--- | :--- | :--- |
| `VG_PAINT_TYPE` | `VGPaintType` | `VG_PAINT_TYPE_COLOR` |
| `VG_PAINT_COLOR` | `VGfloat`[4] | { 0.0f, 0.0f, 0.0f, 1.0f } |
| `VG_PAINT_COLOR_RAMP_SPREAD_MODE` | `VGColorRampSpreadMode` | `VG_COLOR_RAMP_SPREAD_PAD` |
| `VG_PAINT_COLOR_RAMP_STOPS` | `VGfloat`&nbsp;&nbsp;* | Array of Length 0 |
| `VG_PAINT_COLOR_RAMP_PREMULTIPLIED` | `VGboolean` | `VG_TRUE` |
| `VG_PAINT_LINEAR_GRADIENT` | `VGfloat`[4] | { 0.0f, 0.0f, 1.0f, 0.0f } |
| `VG_PAINT_RADIAL_GRADIENT` | `VGfloat`[5] | { 0.0f, 0.0f, 0.0f, 0.0f, 1.0f } |
| `VG_PAINT_PATTERN_TILING_MODE` | `VGTilingMode` | `VG_TILE_FILL` |
_Table 10: VGPaintParamType Defaults_

#### _VGPaintType_
The `VGPaintType` enumeration is used to supply values for the
`VG_PAINT_TYPE` paint parameter to determine the type of paint to be applied.
```
typedef enum {
VG_PAINT_TYPE_COLOR = 0x1B00,
VG_PAINT_TYPE_LINEAR_GRADIENT = 0x1B01,
VG_PAINT_TYPE_RADIAL_GRADIENT = 0x1B02,
VG_PAINT_TYPE_PATTERN = 0x1B03
} VGPaintType;
```

## _9.2 Color Paint_
Color paint uses a fixed color and alpha for all pixels. An alpha value of 1 produces a
fully opaque color. Colors are specified in non-premultiplied sRGBA format.

#### _Setting Color Paint Parameters_
To enable color paint, use *vgSetParameteri* to set the paint type to
`VG_PAINT_TYPE_COLOR`.

<!-----120----->
The **vgSetParameterfv** function allows the color and alpha values to be set using the
`VG_PAINT_COLOR` paint parameter to values between 0 and 1. Values outside this
range are interpreted as the nearest endpoint of the range.
```
VGfloat fill_red, fill_green, fill_blue, fill_alpha;
VGfloat stroke_red, stroke_green, stroke_blue, stroke_alpha;
VGPaint myFillPaint, myStrokePaint;

VGfloat * fill_RGBA = {
fill_red, fill_green, fill_blue, fill_alpha
};
VGfloat * stroke_RGBA = {
stroke_red, stroke_green, stroke_blue, stroke_alpha
};

/* Fill with color paint */
vgSetParameteri(myFillPaint, VG_PAINT_TYPE, VG_PAINT_TYPE_COLOR);
vgSetParameterfv(myFillPaint, VG_PAINT_COLOR, 4, fill_RGBA);
/* Stroke with color paint */
vgSetParameteri(myStrokePaint, VG_PAINT_TYPE, VG_PAINT_TYPE_COLOR);
vgSetParameterfv(myStrokePaint, VG_PAINT_COLOR, 4, stroke_RGBA);
```
#### _vgSetColor_
As a shorthand, the **vgSetColor** function allows the `VG_PAINT_COLOR` parameter of a
given `paint` object to be set using a 32-bit non-premultiplied `sRGBA_8888`
representation (see Section 10.210.2). The `rgba` parameter is a `VGuint` with 8 bits of
red starting at the most significant bit, followed by 8 bits each of green, blue, and alpha.
Each color or alpha channel value is conceptually divided by 255.0f to obtain a value
between 0 and 1.
```
void vgSetColor(VGPaint paint, VGuint rgba)
```
> **_Errors_**
>
> `VG_BAD_HANDLE_ERROR`
> * if `paint` is not a valid paint handle, or is not shared with the current context

The code:
```
VGPaint paint;
VGuint rgba;
vgSetColor(paint, rgba)
```

<!-----121----->
is equivalent to the code:
```
VGfloat rgba_f[4];
rgba_f[0] = ((rgba >> 24) & 0xff)/255.0f;
rgba_f[1] = ((rgba >> 16) & 0xff)/255.0f;
rgba_f[2] = ((rgba >> 8) & 0xff)/255.0f;
rgba_f[3] = ( rgba & 0xff)/255.0f;
vgSetParameterfv(paint, VG_PAINT_COLOR, 4, rgba_f);
```

#### _vgGetColor_
The current setting of the `VG_PAINT_COLOR` parameter on a given `paint` object may
be queried as a 32-bit non-premultiplied `sRGBA_8888` value. Each color channel or
alpha value is clamped to the range [0, 1] , multiplied by 255, and rounded to obtain an
8-bit integer; the resulting values are packed into a 32-bit value in the same format as for
**vgSetColor**.
```
VGuint vgGetColor(VGPaint paint)
```

> **_Errors_**
>
> `VG_BAD_HANDLE_ERROR`
> * if `paint` is not a valid paint handle, or is not shared with the current context

<!-----122----->
The code:
```
VGPaint paint;
VGuint rgba;
rgba = vgGetColor(paint);
```
is equivalent to the code:
```
#define CLAMP(x) ((x) < 0.0f ? 0.0f : ((x) > 1.0f ? 1.0f : (x)))
VGfloat rgba_f[4];
int red, green, blue, alpha;
vgGetParameterfv(paint, VG_PAINT_COLOR, 4, rgba_f);
/*
* Clamp color and alpha values from vgGetParameterfv to the
* [0, 1] range, scale to 8 bits, and round to integer.
*/
red = (int)(CLAMP(rgba_f[0])*255.0f + 0.5f);
green = (int)(CLAMP(rgba_f[1])*255.0f + 0.5f);
blue = (int)(CLAMP(rgba_f[2])*255.0f + 0.5f);
alpha = (int)(CLAMP(rgba_f[3])*255.0f + 0.5f);
rgba = (red << 24) | (green << 16) | (blue << 8) | alpha;
```

## _9.3 Gradient Paint_
Gradients are patterns used for filling or stroking. They are defined
mathematically in two parts; a scalar-valued _gradient function_ defined at every
point in the two-dimensional plane (in paint coordinates), followed by a _color
ramp_ mapping.

### _9.3.1 Linear Gradients_
Linear gradients define a scalar-valued gradient function based on two points (_x0_, _y0_)
and (_x1_, _y1_) (in the paint coordinate system) with the following properties:
* It is equal to 0 at (_x0_, _y0_)
* It is equal to 1 at (_x1_, _y1_)
* It increases linearly along the line from (_x0_, _y0_) to (_x1_, _y1_)
* It is constant along lines perpendicular to the line from (x0, y0) to (x1, y1)

An expression for the gradient function is:

<!-----123----->
g  x , y =
x  x−x0  y  y− y0
 x2 y2
where Δx = x1 – x0 and Δy = y1 – y0. If the points (x0, y0) and (x1, y1) are coincident
(and thus Δx2 + Δy2 = 0), the function is given the value 1 everywhere.

#### _Setting Linear Gradient Parameters_
To enable linear gradient paint, use **vgSetParameteri** to set the paint type to
`VG_PAINT_TYPE_LINEAR_GRADIENT`.
The linear gradient parameters are set using **vgSetParameterfv** with a `paramType`
argument of `VG_PAINT_LINEAR_GRADIENT`. The gradient values are supplied as
a vector of 4 floats in the order { x0, y0, x1, y1 }.
```
VGfloat fill_x0, fill_y0, fill_x1, fill_y1;
VGfloat stroke_x0, stroke_y0, stroke_x1, stroke_y1;
VGPaint myFillPaint, myStrokePaint;
VGfloat * fill_linear_gradient = {
fill_x0, fill_y0, fill_x1, fill_y1
};
VGfloat * stroke_linear_gradient = {
stroke_x0, stroke_y0, stroke_x1, stroke_y1
};
/* Fill with linear gradient paint */
vgSetParameteri(myFillPaint, VG_PAINT_TYPE,
VG_PAINT_TYPE_LINEAR_GRADIENT);
vgSetParameterfv(myFillPaint, VG_PAINT_LINEAR_GRADIENT,
4, fill_linear_gradient);
/* Stroke with linear gradient paint */
vgSetParameteri(myStrokePaint, VG_PAINT_TYPE,
VG_PAINT_TYPE_LINEAR_GRADIENT);
vgSetParameterfv(myStrokePaint, VG_PAINT_LINEAR_GRADIENT,
4, stroke_linear_gradient);
```

### _9.3.2 Radial Gradients_
Radial gradients define a scalar-valued gradient function based on a gradient circle
defined by a center point (cx, cy), a radius r, and a focal point (fx, fy) that is forced to lie
within the circle. All parameters are given in the paint coordinate system.

The computation of the radial gradient function is illustrated in Figure 18. The function
is equal to 0 at the focal point and 1 along the circumference of the gradient circle.

<!-----124----->
Elsewhere, it is equal to the distance between (x, y) and (fx, fy) (shown as d1) divided by
the length of the line segment starting at (fx, fy), passing through (x, y), and ending on the
circumference of the gradient circle (shown as d2). If the radius is less than or equal to 0,
the function is given the value 1 everywhere.

An expression for the gradient function may be derived by defining the line between (fx,
fy) and (x, y) by the parametric expression (fx, fy) + t*(x – fx, y – fy) and determining the
positive value of t at which the line intersects the circle (x – cx)2 + (y – cy)2 = r2. Figure
18 illustrates the construction. The gradient value g(x, y) is then given by 1/t. The
resulting expression is:
g x , y = dx2dy2
 r2 dx2dy2−dx fy'−dy fx' 2−dx fx'dy fy' 
resulting expression is:
where fx' = fx – cx, fy' = fy – cy, dx = x – fx and dy = y – fy.
This may be rearranged and simplified to obtain a formula that does not require per-pixel
division:
g x , y =
dx fx'dy fy'  r2dx 2dy2−dx fy'−dy fx' 2
r2− fx '2 fy'2
One way to evaluate the gradient function efficiently is to rewrite it in the form:
g y  x =A xB C x2D xE
and to use forward differencing of Ax + B and Cx2 + Dx + E to evaluate it incrementally
along a scanline with several additions and a single square root per pixel.

<!-----125----->
![figure18](figures/figure18.PNG)
_Figure 18: Radial Gradient Function_

#### _Setting Radial Gradient Parameters_
To enable radial gradient paint, use **vgSetParameteri** to set the paint type to
`VG_PAINT_TYPE_RADIAL_GRADIENT`. The radial gradient parameters are set using
**vgSetParameterfv** with a `paramType` argument of
`VG_PAINT_RADIAL_GRADIENT`. The gradient values are supplied as a vector of
5 floats in the order { cx, cy, fx, fy, r }.

If (fx, fy) lies outside the circumference of the circle, the intersection of the line
from the center to the focal point with the circumference of the circle is used as
the focal point in place of the specified point. To avoid a division by 0, the
implementation may move the focal point along the line towards the center of
the circle by an amount sufficient to avoid numerical instability, provided the
new location lies at a distance of at least .99r from the circle center. The following
code illustrates the setting of radial gradient parameters:


<!-----126----->
```
VGPaint myFillPaint, myStrokePaint;
VGfloat fill_cx, fill_cy, fill_fx, fill_fy, fill_r;
VGfloat stroke_cx, stroke_cy, stroke_fx, stroke_fy, stroke_r;
VGfloat * fill_radial_gradient = { fill_cx, fill_cy,
fill_fx, fill_fy, fill_r };
VGfloat * stroke_radial_gradient = { stroke_cx, stroke_cy,
stroke_fx, stroke_fy, stroke_r };
vgSetParameteri(myFillPaint, VG_PAINT_TYPE, /* Fill */
VG_PAINT_TYPE_RADIAL_GRADIENT);
vgSetParameterfv(myFillPaint, VG_PAINT_RADIAL_GRADIENT,
5, fill_radial_gradient);
vgSetParameteri(myStrokePaint, VG_PAINT_TYPE, /* Stroke */
VG_PAINT_TYPE_RADIAL_GRADIENT);
vgSetParameterfv(myStrokePaint, VG_PAINT_RADIAL_GRADIENT,
5, stroke_radial_gradient);
```

### _9.3.3 Color Ramps_
Color ramps map the scalar values produced by gradient functions to colors. The
application defines the non-premultiplied sRGBA color and alpha value associated with
each of a number of values, called stops. A stop is defined by an _offset_ between 0 and 1,
inclusive, and a color value. Stops must be specified in increasing order; if they are not,
the entire sequence is ignored. It is legal to have multiple stops with the same offset
value, which will result in a discontinuity in the color ramp, with the first stop with a
given offset value defining the right endpoint of one interval and the last stop with the
same offset value defining the left endpoint of the next interval. At an offset value equal
to that of a stop, the color value is that of the last stop with the given offset. Intermediate
stops with the same offset value have no effect. Stops with offsets less than 0 or greater
than 1 are ignored.

&nbsp;&nbsp;&nbsp;&nbsp;If no valid stops have been specified (_e.g_., due to an empty input array, out-of-range,
or out-of-order stops), a stop at 0 with (_R_, _G_, _B_, _α_) color (0.0, 0.0, 0.0, 1.0) (opaque
black) and a stop at 1 with color (1.0, 1.0, 1.0, 1.0) (opaque white) are implicitly defined.
If at least one valid stop has been specified, but none has been defined with an offset of
0, an implicit stop is added with an offset of 0 and the same color as the first user-defined
stop. If at least one valid stop has been specified, but none has been defined with an
offset of 1, an implicit stop is added with an offset of 1 and the same color as the last
user-defined stop.

&nbsp;&nbsp;&nbsp;&nbsp;If a color or alpha value of a given stop falls outside of the range [0, 1], the closest
endpoint of the range is used instead.

&nbsp;&nbsp;&nbsp;&nbsp;If the paint’s `VG_PAINT_COLOR_RAMP_PREMULTIPLIED` flag is set to
`VG_TRUE`, color and alpha values at each gradient stop are multiplied together to form
premultiplied sRGBA values prior to interpolation. Otherwise, color and alpha values are
processed independently.

<!-----127----->
&nbsp;&nbsp;&nbsp;&nbsp;Color and alpha values at offset values between the stops are defined by means of
linear interpolation between the premultiplied or non-premultiplied color values defined
at the nearest stops above and below the given offset value.

#### _VG_MAX_COLOR_RAMP_STOPS_
The `VG_MAX_COLOR_RAMP_STOPS` parameter contains the maximum number of
gradient stops supported by the OpenVG implementation. All implementations must
support at least 32 stops. If there is no implementation-defined limit, a value of
VG_MAXINT may be returned. Implicitly defined stops at offsets 0 and 1 are not counted
against this maximum. The value may be retrieved by calling **vgGeti**:
```
VGint maxStops = vgGeti(VG_MAX_COLOR_RAMP_STOPS);
```

#### _VGColorRampSpreadMode_
The application may only define stops with offsets between 0 and 1. Spread modes
define how the given set of stops are repeated or extended in order to define interpolated
color values for arbitrary input values outside the [0,1] range. The
`VGColorRampSpreadMode` enumeration defines three modes:
* VG_COLOR_RAMP_SPREAD_PAD – extend stops
* VG_COLOR_RAMP_SPREAD_REPEAT – repeat stops
* VG_COLOR_RAMP_SPREAD_REFLECT – repeat stops in reflected order

```
typedef enum {
VG_COLOR_RAMP_SPREAD_PAD = 0x1C00,
VG_COLOR_RAMP_SPREAD_REPEAT = 0x1C01,
VG_COLOR_RAMP_SPREAD_REFLECT = 0x1C02
} VGColorRampSpreadMode;
```

In pad mode, the colors defined at 0 and 1 are used for all stop values less than 0 or
greater than 1, respectively.

In repeat mode, the color values defined between 0 and 1 are repeated indefinitely in
both directions. Gradient values outside the [0, 1] range are shifted by an integer amount
to place them into that range. For example, a gradient value of 5.6 will receive the same
color as a gradient value of 0.6. A gradient value of -5.6 will receive the same color as a
gradient value of 0.4 (since 0.4 = -5.6 + 6).

In reflect mode, the color values defined between 0 and 1 are repeated indefinitely in
both directions, but with alternate copies of the range reversed. A gradient value of 1.2
will receive the same color as a gradient value of 0.8, since 0.8 = 1.0 – 0.2 and 1.2
= 1.0 + 0.2. A gradient value of 2.4 will receive the same color as a gradient value


<!-----128----->
of 0.4.

The color ramp pad modes are illustrated schematically in Figure 19.

_Figure 19: Color Ramp Pad Modes_

#### _Setting Color Ramp Parameters_
Color ramp parameters are set using **vgSetParameter**. The
`VG_PAINT_COLOR_RAMP_SPREAD_MODE` parameter controls the spread mode
using a value from the `VGColorRampSpreadMode` enumeration. The
`VG_PAINT_COLOR_RAMP_PREMULTIPLIED` parameter takes a `VGboolean`
value and controls whether color and alpha values are interpolated in
premultiplied or non-premultiplied form. The `VG_PAINT_COLOR_RAMP_STOPS`
parameter takes an array of floating-point values giving the offsets and colors of
the stops, in order. Each stop is defined by a floating-point offset value and four
floating-point values containing the sRGBA color and alpha value associated
with each stop, in the form of a non-premultiplied (R, G, B, α) quad. The
**vgSetParameter** function will generate an error if the number of values
submitted is not a multiple of 5 (zero is acceptable). Up to
`VG_MAX_COLOR_RAMP_STOPS` 5-tuples may be set. If more than
`VG_MAX_COLOR_RAMP_STOPS` 5-tuples are specified, those beyond the first
`VG_MAX_COLOR_RAMP_STOPS` are discarded immediately (and will not be

<!-----129----->
returned by **vgGetParameter**).
```
VGPaint myFillPaint, myStrokePaint;
VGColorRampSpreadMode fill_spreadMode;
VGboolean fill_premultiplied;
VGfloat fill_stops[5*FILL_NUM_STOPS];
VGColorRampSpreadMode stroke_spreadMode;
VGboolean stroke_premultiplied;
VGfloat stroke_stops[5*STROKE_NUM_STOPS];
vgSetParameteri(myFillPaint, VG_PAINT_COLOR_RAMP_SPREAD_MODE,
fill_spreadMode);
vgSetParameteri(myFillPaint, VG_PAINT_COLOR_RAMP_PREMULTIPLIED,
fill_premultiplied);
vgSetParameterfv(myFillPaint, VG_PAINT_COLOR_RAMP_STOPS,
5*FILL_NUM_STOPS, fill_stops);
vgSetParameteri(myStrokePaint, VG_PAINT_COLOR_RAMP_SPREAD_MODE,
stroke_spreadMode);
vgSetParameteri(myStrokePaint, VG_PAINT_COLOR_RAMP_PREMULTIPLIED,
stroke_premultiplied);
vgSetParameterfv(myStrokePaint, VG_PAINT_COLOR_RAMP_STOPS,
5*STROKE_NUM_STOPS, stroke_stops);
```
A common set of color ramp settings are used for both linear and radial
gradients defined on a given paint object.


#### _Formal Definition of Spread Modes_
<a name="Formal_Definition_of_Spread_Modes"></a>
This section provides a formal definition of the color ramp spread modes.

In the following, assume that a sequence of stops \left\{ { S }_{ 0 },\quad { S }_{ 1 }, \quad... , { S }_{ N-1 }  \right\}  have been defined by the application, and/or by default or implicit values. The stop { S }_{ i } is defined to have offset
xi and color ci. The stops are assumed to be ordered by offset but may have duplicate
offsets; that is, for all i < j, xi ≤ xj. To determine the interpolated color value at a given
offset value v, determine the smallest i such that xi+1 > v. If xi = v, use the color ci,
otherwise perform linear interpolation between the stops Si and Si+1 to produce the color
ci + (ci+1 – ci)(v – xi)/(xi+1 – xi).

In pad mode, values smaller than 0 are assigned the color { c }_{ 0 } and values greater than or equal to 1 are assigned the color { c }_{ N-1 }.

In repeat mode, the offset value v is mapped to a new value v' that is guaranteed to lie between 0 and 1. Following this mapping, the color is defined as for pad mode:
