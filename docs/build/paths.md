
<a id='Paths-1'></a>

## Paths

<a id='Devices.Paths.Path' href='#Devices.Paths.Path'>#</a>
**`Devices.Paths.Path`** &mdash; *Type*.

---


```
type Path{T<:Real} <: AbstractArray{Tuple{Segment{T},Style},1}
    origin::Point{2,T}
    α0::Real
    style0::Style
    segments::Array{Segment{T},1}
    styles::Array{Style,1}
    Path(origin::Point{2,T}, α0::Real, style0::Style, segments::Array{Segment{T},1},
        styles::Array{Style,1}) = new(origin, α0, style0, segments, styles)
    Path(style::Style) =
        new(Point(zero(T),zero(T)), 0.0, style, Segment{T}[], Style[])
end
```

Type for abstracting an arbitrary styled path in the plane. Iterating returns tuples of (`segment`, `style`).

<a id='Devices.Paths.Path-Tuple{FixedSizeArrays.Point{2,T<:Real},Real,Devices.Paths.Style}' href='#Devices.Paths.Path-Tuple{FixedSizeArrays.Point{2,T<:Real},Real,Devices.Paths.Style}'>#</a>
**`Devices.Paths.Path`** &mdash; *Method*.

---


```
Path{T<:Real}(origin::Point{2,T}=Point(0.0,0.0), α0::Real=0.0, style0::Style=Trace(1.0))
```

Convenience constructor for `Path{T}` object.

<a id='Devices.Paths.Path-Tuple{Tuple{T<:Real,T<:Real}}' href='#Devices.Paths.Path-Tuple{Tuple{T<:Real,T<:Real}}'>#</a>
**`Devices.Paths.Path`** &mdash; *Method*.

---


```
Path{T<:Real}(origin::Tuple{T,T})
```

Convenience constructor for `Path{T}` object.

<a id='Devices.Paths.Path-Tuple{Tuple{T<:Real,T<:Real},Real}' href='#Devices.Paths.Path-Tuple{Tuple{T<:Real,T<:Real},Real}'>#</a>
**`Devices.Paths.Path`** &mdash; *Method*.

---


```
Path{T<:Real}(origin::Tuple{T,T}, α0::Real)
```

Convenience constructor for `Path{T}` object.

<a id='Devices.Paths.pathlength-Tuple{Devices.Paths.Path{T<:Real}}' href='#Devices.Paths.pathlength-Tuple{Devices.Paths.Path{T<:Real}}'>#</a>
**`Devices.Paths.pathlength`** &mdash; *Method*.

---


```
pathlength(p::Path)
```

Physical length of a path. Note that `length` will return the number of segments in a path, not the physical length.


<a id='Segments-1'></a>

## Segments

<a id='Devices.Paths.Segment' href='#Devices.Paths.Segment'>#</a>
**`Devices.Paths.Segment`** &mdash; *Type*.

---


```
abstract Segment{T<:Real}
```

Path segment in the plane. All Segment objects should have the implement the following methods:

  * `length`
  * `origin`
  * `α0`
  * `setorigin!`
  * `setα0!`
  * `lastangle`

<a id='Devices.Paths.Straight' href='#Devices.Paths.Straight'>#</a>
**`Devices.Paths.Straight`** &mdash; *Type*.

---


```
type Straight{T<:Real} <: Segment{T}
    l::T
    origin::Point{2,T}
    α0::Real
    f::Function
    Straight(l, origin, α0) = begin
        s = new(l, origin, α0)
        s.f = t->(s.origin+Point(t*s.l*cos(s.α0),t*s.l*sin(s.α0)))
        s
    end
end
```

A straight line segment is parameterized by its length. It begins at a point `origin` with initial angle `α0`.

The parametric function over `t ∈ [0,1]` describing the line segment is given by:

`t -> origin + Point(t*l*cos(α),t*l*sin(α))`

<a id='Devices.Paths.Turn' href='#Devices.Paths.Turn'>#</a>
**`Devices.Paths.Turn`** &mdash; *Type*.

---


```
type Turn{T<:Real} <: Segment{T}
    α::Real
    r::T
    origin::Point{2,T}
    α0::Real
    f::Function
    Turn(α, r, origin, α0) = begin
        s = new(α, r, origin, α0)
        s.f = t->begin
            cen = s.origin + Point(s.r*cos(s.α0+sign(s.α)*π/2), s.r*sin(s.α0+sign(s.α)*π/2))
            cen + Point(s.r*cos(s.α0-sign(α)*π/2+s.α*t), s.r*sin(s.α0-sign(α)*π/2+s.α*t))
        end
        s
    end
end
```

A circular turn is parameterized by the turn angle `α` and turning radius `r`. It begins at a point `origin` with initial angle `α0`.

The center of the circle is given by:

`cen = origin + Point(r*cos(α0+sign(α)*π/2), r*sin(α0+sign(α)*π/2))`

The parametric function over `t ∈ [0,1]` describing the turn is given by:

`t -> cen + Point(r*cos(α0-sign(α)*π/2+α*t), r*sin(α0-sign(α)*π/2+α*t))`

<a id='Devices.Paths.CompoundSegment' href='#Devices.Paths.CompoundSegment'>#</a>
**`Devices.Paths.CompoundSegment`** &mdash; *Type*.

---


```
type CompoundSegment{T<:Real} <: Segment{T}
    segments::Array{Segment{T},1}
    f::Function

    CompoundSegment(segments) = begin
        s = new(segments)
        s.f = param(s)
        s
    end
end
```

Consider an array of segments as one contiguous segment. Useful e.g. for applying styles, uninterrupted over segment changes.


<a id='Styles-1'></a>

## Styles

<a id='Devices.Paths.Style' href='#Devices.Paths.Style'>#</a>
**`Devices.Paths.Style`** &mdash; *Type*.

---


```
abstract Style
```

How to render a given path segment. All styles should implement the following methods:

  * `distance`
  * `extent`
  * `paths`
  * `width`
  * `divs`

<a id='Devices.Paths.Trace' href='#Devices.Paths.Trace'>#</a>
**`Devices.Paths.Trace`** &mdash; *Type*.

---


```
type Trace <: Style
    width::Function
    divs::Int
end
```

Simple, single trace.

  * `width::Function`: trace width.
  * `divs::Int`: number of segments to render. Increase if you see artifacts.

<a id='Devices.Paths.CPW' href='#Devices.Paths.CPW'>#</a>
**`Devices.Paths.CPW`** &mdash; *Type*.

---


```
type CPW <: Style
    trace::Function
    gap::Function
    divs::Int
end
```

Two adjacent traces can form a coplanar waveguide.

  * `trace::Function`: center conductor width.
  * `gap::Function`: distance between center conductor edges and ground plane
  * `divs::Int`: number of segments to render. Increase if you see artifacts.

May need to be inverted with respect to a ground plane, depending on how the pattern is written.

<a id='Devices.Paths.CompoundStyle' href='#Devices.Paths.CompoundStyle'>#</a>
**`Devices.Paths.CompoundStyle`** &mdash; *Type*.

---


```
type CompoundStyle{T<:Real} <: Style
    segments::Array{Segment{T},1}
    styles::Array{Style,1}
    f::Function
    CompoundStyle(segments, styles) = begin
        s = new(segments, styles)
        s.f = param(s)
        s
    end
end
```

Combines styles together for use with a `CompoundSegment`.

  * `segments`: Needed for divs function.
  * `styles`: Array of styles making up the object.
  * `f`: returns tuple of style index and the `t` to use for that style's parametric function.

<a id='Devices.Paths.DecoratedStyle' href='#Devices.Paths.DecoratedStyle'>#</a>
**`Devices.Paths.DecoratedStyle`** &mdash; *Type*.

---


```
type DecoratedStyle{S<:Real} <: Style
    s::Style
    ts::AbstractArray{Float64,1}
    offsets::Array{S,1}
    dirs::Array{Int,1}
    cells::Array{ASCIIString,1}
    DecoratedStyle(s) = begin
        a = new(s)
        a.ts = Float64[]
        a.offsets = S[]
        a.dirs = Int[]
        a.cells = ASCIIString[]
    end
    DecoratedStyle(s,t,o,r,c) = new(s,t,o,r,c)
end
```

Style with decorations, like periodic structures along the path, etc.


<a id='Path-interrogation-1'></a>

## Path interrogation

<a id='Devices.Paths.direction' href='#Devices.Paths.direction'>#</a>
**`Devices.Paths.direction`** &mdash; *Function*.

---


```
direction(p::Function, t)
```

For some parameteric function `p(t)↦Point(x(t),y(t))`, returns the angle at which the path is pointing for a given `t`.

<a id='Devices.Paths.pathlength' href='#Devices.Paths.pathlength'>#</a>
**`Devices.Paths.pathlength`** &mdash; *Function*.

---


```
pathlength(p::AbstractArray{Segment})
```

Total physical length of segments.

```
pathlength(p::Path)
```

Physical length of a path. Note that `length` will return the number of segments in a path, not the physical length.

<a id='Devices.Paths.origin' href='#Devices.Paths.origin'>#</a>
**`Devices.Paths.origin`** &mdash; *Function*.

---


```
origin(p::Path)
```

First point of a path.

```
origin{T}(s::Segment{T})
```

Return the first point in a segment (calculated).

<a id='Devices.Paths.setorigin!' href='#Devices.Paths.setorigin!'>#</a>
**`Devices.Paths.setorigin!`** &mdash; *Function*.

---


```
setorigin!(s::CompoundSegment, p::Point)
```

Set the origin of a compound segment.

```
setorigin!(s::Turn, p::Point)
```

Set the origin of a turn.

```
setorigin!(s::Straight, p::Point)
```

Set the origin of a straight segment.

<a id='Devices.Paths.α0' href='#Devices.Paths.α0'>#</a>
**`Devices.Paths.α0`** &mdash; *Function*.

---


```
α0(p::Path)
```

First angle of a path.

```
α0(s::Segment)
```

Return the first angle in a segment (calculated).

<a id='Devices.Paths.setα0!' href='#Devices.Paths.setα0!'>#</a>
**`Devices.Paths.setα0!`** &mdash; *Function*.

---


```
setα0!(s::CompoundSegment, α0′)
```

Set the starting angle of a compound segment.

```
setα0!(s::Turn, α0′)
```

Set the starting angle of a turn.

```
setα0!(s::Straight, α0′)
```

Set the angle of a straight segment.

<a id='Devices.Paths.lastpoint' href='#Devices.Paths.lastpoint'>#</a>
**`Devices.Paths.lastpoint`** &mdash; *Function*.

---


```
lastpoint(p::Path)
```

Last point of a path.

```
lastpoint{T}(s::Segment{T})
```

Return the last point in a segment (calculated).

<a id='Devices.Paths.lastangle' href='#Devices.Paths.lastangle'>#</a>
**`Devices.Paths.lastangle`** &mdash; *Function*.

---


```
lastangle(p::Path)
```

Last angle of a path.

```
lastangle(s::Segment)
```

Return the last angle in a segment (calculated).

<a id='Devices.Paths.firststyle' href='#Devices.Paths.firststyle'>#</a>
**`Devices.Paths.firststyle`** &mdash; *Function*.

---


```
firststyle(p::Path)
```

Style of the first segment of a path.

<a id='Devices.Paths.laststyle' href='#Devices.Paths.laststyle'>#</a>
**`Devices.Paths.laststyle`** &mdash; *Function*.

---


```
laststyle(p::Path)
```

Style of the last segment of a path.


<a id='Path-building-1'></a>

## Path building

<a id='Base.append!-Tuple{Devices.Paths.Path{T<:Real},Devices.Paths.Path{T<:Real}}' href='#Base.append!-Tuple{Devices.Paths.Path{T<:Real},Devices.Paths.Path{T<:Real}}'>#</a>
**`Base.append!`** &mdash; *Method*.

---


```
append!(p::Path, p′::Path)
```

Given paths `p` and `p′`, path `p′` is appended to path `p`. The origin and initial angle of the first segment from path `p′` is modified to match the last point and last angle of path `p`.

<a id='Devices.Paths.adjust!' href='#Devices.Paths.adjust!'>#</a>
**`Devices.Paths.adjust!`** &mdash; *Function*.

---


```
adjust!(p::Path, n::Integer=1)
```

Adjust a path's parametric functions starting from index `n`. Used internally whenever segments are inserted into the path.

<a id='Devices.Paths.launch!' href='#Devices.Paths.launch!'>#</a>
**`Devices.Paths.launch!`** &mdash; *Function*.

---


```
launch!(p::Path; extround=5, trace0=300, trace1=5,
        gap0=150, gap1=2.5, flatlen=250, taperlen=250)
```

Add a launcher to the path. Somewhat intelligent in that the launcher will reverse its orientation depending on if it is at the start or the end of a path.

There are numerous keyword arguments to control the behavior:

  * `extround`: Rounding radius of the outermost corners; should be less than `gap0`.
  * `trace0`: Bond pad width.
  * `trace1`: Center trace width of next CPW segment.
  * `gap0`: Gap width adjacent to bond pad.
  * `gap1`: Gap width of next CPW segment.
  * `flatlen`: Bond pad length.
  * `taperlen`: Length of taper region between bond pad and next CPW segment.

Returns a `Style` object suitable for continuity with the next segment. Ignore the returned style if you are terminating a path.

<a id='Devices.Paths.meander!' href='#Devices.Paths.meander!'>#</a>
**`Devices.Paths.meander!`** &mdash; *Function*.

---


```
meander!{T<:Real}(p::Path{T}, len, r, straightlen, α::Real)
```

Alternate between going straight with length `straightlen` and turning with radius `r` and angle `α`. Each turn goes the opposite direction of the previous. The total length is `len`. Useful for making resonators.

The straight and turn segments are combined into a `CompoundSegment` and appended to the path `p`.

<a id='Devices.Paths.param' href='#Devices.Paths.param'>#</a>
**`Devices.Paths.param`** &mdash; *Function*.

---


```
param{T<:Real}(c::CompoundSegment{T})
```

Return a parametric function over the domain [0,1] that represents the compound segment.

<a id='Devices.Paths.simplify!' href='#Devices.Paths.simplify!'>#</a>
**`Devices.Paths.simplify!`** &mdash; *Function*.

---


```
simplify!(p::Path)
```

All segments of a path are turned into a `CompoundSegment` and all styles of a path are turned into a `CompoundStyle`. The idea here is:

  * Indexing the path becomes more sane when you can combine several path segments into one logical element. A launcher would have several indices in a path unless you could simplify it.
  * You don't need to think hard about boundaries between straights and turns when you want a continuous styling of a very long path.

<a id='Devices.Paths.straight!' href='#Devices.Paths.straight!'>#</a>
**`Devices.Paths.straight!`** &mdash; *Function*.

---


```
straight!(p::Path, l::Real)
```

Extend a path `p` straight by length `l` in the current direction.

<a id='Devices.Paths.turn!' href='#Devices.Paths.turn!'>#</a>
**`Devices.Paths.turn!`** &mdash; *Function*.

---


```
turn!(p::Path, s::ASCIIString, r::Real, sty::Style=laststyle(p))
```

Turn a path `p` with direction coded by string `s`:

  * "l": turn by π/2 (left)
  * "r": turn by -π/2 (right)
  * "lrlrllrrll": do those turns in that order

```
turn!(p::Path, α::Real, r::Real, sty::Style=laststyle(p))
```

Turn a path `p` by angle `α` with a turning radius `r` in the current direction. Positive angle turns left.


<a id='Interfacing-with-gdspy-1'></a>

## Interfacing with gdspy


The Python package `gdspy` is used for rendering paths into polygons. Ultimately we intend to remove this dependency.

<a id='Devices.Paths.distance' href='#Devices.Paths.distance'>#</a>
**`Devices.Paths.distance`** &mdash; *Function*.

---


For a style `s` and parameteric argument `t`, returns the distance between the centers of parallel paths rendered by gdspy.

<a id='Devices.Paths.extent' href='#Devices.Paths.extent'>#</a>
**`Devices.Paths.extent`** &mdash; *Function*.

---


For a style `s` and parameteric argument `t`, returns a distance tangential to the path specifying the lateral extent of the polygons rendered by gdspy.

<a id='Devices.Paths.paths' href='#Devices.Paths.paths'>#</a>
**`Devices.Paths.paths`** &mdash; *Function*.

---


For a style `s` and parameteric argument `t`, returns the number of parallel paths rendered by gdspy.

<a id='Devices.Paths.width' href='#Devices.Paths.width'>#</a>
**`Devices.Paths.width`** &mdash; *Function*.

---


For a style `s` and parameteric argument `t`, returns the width of paths rendered by gdspy.
