# d3-zoom

Pan and zoom SVG, HTML or Canvas using mouse or touch input.

## Installing

If you use NPM, `npm install d3-zoom`. Otherwise, download the [latest release](https://github.com/d3/d3-zoom/releases/latest). You can also load directly from [d3js.org](https://d3js.org), either as a [standalone library](https://d3js.org/d3-zoom.v0.2.min.js) or as part of [D3 4.0](https://github.com/d3/d3). AMD, CommonJS, and vanilla environments are supported. In vanilla, a `d3_zoom` global is exported:

```html
<script src="https://d3js.org/d3-color.v0.4.min.js"></script>
<script src="https://d3js.org/d3-dispatch.v0.4.min.js"></script>
<script src="https://d3js.org/d3-ease.v0.7.min.js"></script>
<script src="https://d3js.org/d3-interpolate.v0.8.min.js"></script>
<script src="https://d3js.org/d3-selection.v0.7.min.js"></script>
<script src="https://d3js.org/d3-timer.v0.4.min.js"></script>
<script src="https://d3js.org/d3-transition.v0.2.min.js"></script>
<script src="https://d3js.org/d3-drag.v0.2.min.js"></script>
<script src="https://d3js.org/d3-zoom.v0.2.min.js"></script>
<script>

var zoom = d3_zoom.zoom();

</script>
```

[Try d3-zoom in your browser.](https://tonicdev.com/npm/d3-zoom)

## API Reference

This table describes how the zoom behavior interprets native events:

| Event        | Listening Element | Zoom Event  | Default Prevented? |
| ------------ | ----------------- | ----------- | ------------------ |
| mousedown⁵   | selection         | start       | no¹                |
| mousemove²   | window¹           | zoom        | yes                |
| mouseup²     | window¹           | end         | yes                |
| dragstart²   | window            | -           | yes                |
| selectstart² | window            | -           | yes                |
| click³       | window            | -           | yes                |
| dblclick     | selection         | *multiple*⁶ | yes                |
| wheel        | selection         | zoom⁷       | yes                |
| touchstart   | selection         | *multiple*⁶ | no⁴                |
| touchmove    | selection         | zoom        | yes                |
| touchend     | selection         | end         | no⁴                |
| touchcancel  | selection         | end         | no⁴                |

The propagation of all consumed events is [immediately stopped](https://dom.spec.whatwg.org/#dom-event-stopimmediatepropagation).

¹ Necessary to capture events outside an iframe; see [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>² Only applies during an active, mouse-based gesture; see [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>³ Only applies immediately after a non-empty, mouse-based gesture.
<br>⁴ Necessary to allow [click emulation](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/HandlingEvents/HandlingEvents.html#//apple_ref/doc/uid/TP40006511-SW7) on touch input; see [d3-drag#9](https://github.com/d3/d3-drag/issues/9).
<br>⁵ Ignored if within 500ms of a touch gesture ending; assumes [click emulation](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/HandlingEvents/HandlingEvents.html#//apple_ref/doc/uid/TP40006511-SW7).
<br>⁶ Double-click and double-tap initiate a transition that emits start, zoom and end events.
<br>⁷ The first wheel event emits a start event; an end event is emitted when no wheel events are received for 150ms.

<a href="#zoom" name="zoom">#</a> d3.<b>zoom</b>()

Creates a new zoom behavior. The returned behavior, [*zoom*](#_drag), is both an object and a function, and is typically applied to selected elements via [*selection*.call](https://github.com/d3/d3-selection#selection_call).

<a href="#_zoom" name="_zoom">#</a> <i>zoom</i>(<i>selection</i>)

Applies this zoom behavior to the specified [*selection*](https://github.com/d3/d3-selection), binding the necessary event listeners to allow panning and zooming, and initializing the [zoom transform](#zoom-transforms) on each selected element to the identity transform if not already defined. This function is typically not invoked directly, and is instead invoked via [*selection*.call](https://github.com/d3/d3-selection#selection_call). For example, to instantiate a zoom behavior and apply it to a selection:

```js
selection.call(d3.zoom().on("zoom", zoomed));
```

Internally, the zoom behavior uses [*selection*.on](https://github.com/d3/d3-selection#selection_on) to bind the necessary event listeners for zooming. The listeners use the name `.zoom`, so you can subsequently unbind the zoom behavior as follows:

```js
selection.on(".zoom", null);
```

Applying the zoom behavior also sets the [-webkit-tap-highlight-color](https://developer.apple.com/library/mac/documentation/AppleApplications/Reference/SafariWebContent/AdjustingtheTextSize/AdjustingtheTextSize.html#//apple_ref/doc/uid/TP40006510-SW5) style to transparent, disabling the tap highlight on iOS. If you want a different tap highlight color, remove or re-apply this style after applying the drag behavior.

<a href="#zoom_transform" name="zoom_transform">#</a> <i>zoom</i>.<b>transform</b>(<i>selection</i>, <i>transform</i>)

If *selection* is a selection, sets the [current zoom transform](#zoomTransform) of the selected elements to the specifed *transform*, instantaneously emitting start, zoom and end [events](#zoom-events). If *selection* is a transition, defines a “zoom” tween to the specified *transform* using [d3.interpolateZoom](https://github.com/d3/d3-interpolate#interpolateZoom), emitting a start event when the transition starts, zoom events for each tick of the transition, and then an end event when the transition ends (or is interrupted). The *transform* may be specified either as a [zoom transform](#zoom-transforms) or as a function that returns a zoom transform. If a function, it is invoked for each selected element, being passed the current datum `d` and index `i`, with the `this` context as the current DOM element.

This function is typically not invoked directly, and is instead invoked via [*selection*.call](https://github.com/d3/d3-selection#selection_call) or [*transition*.call](https://github.com/d3/d3-transition#transition_call). For example, to reset the zoom transform to the [identity transform](#zoomIdentity) instantaneously:

```js
selection.call(zoom.transform, d3.zoomIdentity);
```

To smoothly reset the zoom transform to the identity transform over 750 milliseconds:

```js
selection.transition().duration(750).call(zoom.transform, d3.zoomIdentity);
```

This method requires that you specify the new zoom transform completely, and does not enforce the defined [scale extent](#zoom_scaleExtent) and [translate extent](#zoom_translateExtent), if any. To derive a new transform from the existing transform, and to enforce the scale and translate extents, see the convenience methods [*zoom*.translateBy](#zoom_translateBy), [*zoom*.scaleBy](#zoom_scaleBy) and [*zoom*.scaleTo](#zoom_scaleTo).

<a href="#zoom_translateBy" name="zoom_translateBy">#</a> <i>zoom</i>.<b>translateBy</b>(<i>selection</i>, <i>x</i>, <i>y</i>)

If *selection* is a selection, [translates](#transform_translate) the [current zoom transform](#zoomTransform) of the selected elements by *x* and *y*, such that the new *t<sub>x1</sub>* = *t<sub>x0</sub>* + *k* × *x* and *t<sub>y1</sub>* = *t<sub>y0</sub>* + *k* × *y*. If *selection* is a transition, defines a “zoom” tween translating the current transform. This method is a convenience method for [*zoom*.transform](#zoom_transform). The *x* and *y* translation amounts may be specified either as numbers or as functions that returns numbers. If a function, it is invoked for each selected element, being passed the current datum `d` and index `i`, with the `this` context as the current DOM element.

<a href="#zoom_scaleBy" name="zoom_scaleBy">#</a> <i>zoom</i>.<b>scaleBy</b>(<i>selection</i>, <i>k</i>)

If *selection* is a selection, [scales](#transform_scale) the [current zoom transform](#zoomTransform) of the selected elements by *k*, such that the new *k₁* = *k₀* × *k*. If *selection* is a transition, defines a “zoom” tween translating the current transform. This method is a convenience method for [*zoom*.transform](#zoom_transform). The *k* scale factor may be specified either as numbers or as functions that returns numbers. If a function, it is invoked for each selected element, being passed the current datum `d` and index `i`, with the `this` context as the current DOM element.

<a href="#zoom_scaleTo" name="zoom_scaleTo">#</a> <i>zoom</i>.<b>scaleTo</b>(<i>selection</i>, <i>k</i>)

If *selection* is a selection, [scales](#transform_scale) the [current zoom transform](#zoomTransform) of the selected elements to *k*, such that the new *k₁* = *k*. If *selection* is a transition, defines a “zoom” tween translating the current transform. This method is a convenience method for [*zoom*.transform](#zoom_transform). The *k* scale factor may be specified either as numbers or as functions that returns numbers. If a function, it is invoked for each selected element, being passed the current datum `d` and index `i`, with the `this` context as the current DOM element.

<a href="#zoom_filter" name="zoom_filter">#</a> <i>zoom</i>.<b>filter</b>([<i>filter</i>])

If *filter* is specified, sets the filter to the specified function and returns the zoom behavior. If *filter* is not specified, returns the current filter, which defaults to:

```js
function filter() {
  return !event.button;
}
```

If the filter returns falsey, the initiating event is ignored and no zoom gestures are started. Thus, the filter determines which input events are ignored. The default filter ignores mousedown events on secondary buttons, since those buttons are typically intended for other purposes, such as the context menu.

<a href="#zoom_extent" name="zoom_extent">#</a> <i>zoom</i>.<b>extent</b>([<i>extent</i>])

If *extent* is specified, sets the viewport extent to the specified array of points [[*x0*, *y0*], [*x1*, *y1*]], where [*x0*, *y0*] is the top-left corner of the viewport and [*x1*, *y1*] is the bottom-right corner of the viewport, and returns this zoom behavior. The *extent* may also be specified as a function which returns such an array; if a function, it is invoked for each selected element, being passed the current datum `d` and index `i`, with the `this` context as the current DOM element. If *extent* is not specified, returns the current extent accessor, which defaults to:

```js
function extent() {
  var svg = this.ownerSVGElement;
  return [[0, 0], svg
      ? [svg.width.baseVal.value, svg.height.baseVal.value]
      : [this.clientWidth, this.clientHeight]];
}
```

The viewport extent affects several functions: the center of the viewport remains fixed during changes by [*zoom*.scaleBy](#zoom_scaleBy) and [*zoom*.scaleTo](#zoom_scaleTo); the viewport center and dimensions affect the path chosen by [d3.interpolateZoom](https://github.com/d3/d3-interpolate#interpolateZoom); and the viewport extent is needed to enforce the optional [translate extent](#zoom_translateExtent.)

<a href="#zoom_scaleExtent" name="zoom_scaleExtent">#</a> <i>zoom</i>.<b>scaleExtent</b>([<i>extent</i>])

If *extent* is specified, sets the scale extent to the specified array of numbers [*k0*, *k1*] where *k0* is the minimum allowed scale factor and *k1* is the maximum allowed scale factor, and returns this zoom behavior. If *extent* is not specified, returns the current scale extent, which defaults to [0, ∞]. The scale extent restricts zooming in and out. It is enforced on interaction and when using [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) and [*zoom*.translateBy](#zoom_translateBy); however, it is not enforced when using [*zoom*.transform](#zoom_transform) to set the transform explicitly.

<a href="#zoom_translateExtent" name="zoom_translateExtent">#</a> <i>zoom</i>.<b>translateExtent</b>([<i>extent</i>])

If *extent* is specified, sets the translate extent to the specified array of points [[*x0*, *y0*], [*x1*, *y1*]], where [*x0*, *y0*] is the top-left corner of the world and [*x1*, *y1*] is the bottom-right corner of the world, and returns this zoom behavior. If *extent* is not specified, returns the current translate extent, which defaults to [[-∞, -∞], [+∞, +∞]]. The translate extent restricts panning, and may cause translation on zoom out. It is enforced on interaction and when using [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) and [*zoom*.translateBy](#zoom_translateBy); however, it is not enforced when using [*zoom*.transform](#zoom_transform) to set the transform explicitly.

<a href="#zoom_duration" name="zoom_duration">#</a> <i>zoom</i>.<b>duration</b>([<i>duration</i>])

If *duration* is specified, sets the duration for zoom transitions on double-click and double-tap to the specified number of milliseconds and returns the zoom behavior. If *duration* is not specified, returns the current duration, which defaults to 250 milliseconds. If the duration is not greater than zero, double-click and -tap trigger instantaneous changes to the zoom transform rather than initiating smooth transitions.

<a href="#zoom_on" name="zoom_on">#</a> <i>zoom</i>.<b>on</b>(<i>typenames</i>[, <i>listener</i>])

If *listener* is specified, sets the event *listener* for the specified *typenames* and returns the zoom behavior. If an event listener was already registered for the same type and name, the existing listener is removed before the new listener is added. If *listener* is null, removes the current event listeners for the specified *typenames*, if any. If *listener* is not specified, returns the first currently-assigned listener matching the specified *typenames*, if any. When a specified event is dispatched, each *listener* will be invoked with the same context and arguments as [*selection*.on](https://github.com/d3/d3-selection#selection_on) listeners: the current datum `d` and index `i`, with the `this` context as the current DOM element.

The *typenames* is a string containing one or more *typename* separated by whitespace. Each *typename* is a *type*, optionally followed by a period (`.`) and a *name*, such as `zoom.foo` and `zoom.bar`; the name allows multiple listeners to be registered for the same *type*. The *type* must be one of the following:

* `start` - after zooming begins (such as on mousedown).
* `zoom` - after a change to the zoom transform (such as on mousemove).
* `end` - after zooming ends (such as on mouseup ).

See [*dispatch*.on](https://github.com/d3/d3-dispatch#dispatch_on) for more.

### Zoom Events

When a [zoom event listener](#zoom_on) is invoked, [d3.event](https://github.com/d3/d3-selection#event) is set to the current zoom event. The *event* object exposes several fields:

* `target` - the associated [zoom behavior](#zoom).
* `type` - the string “start”, “zoom” or “end”; see [*zoom*.on](#zoom_on).
* `transform` - the current [zoom transform](#zoom-transforms).
* `sourceEvent` - the underlying input event, such as mousemove or touchmove.

### Zoom Transforms

The zoom behavior stores the zoom state on the element to which the zoom behavior was [applied](#_zoom), not on the zoom behavior itself. This is because the zoom behavior can be applied to many elements simultaneously, and each element can be zoomed independently. The zoom state can change either on user interaction or programmatically via [*zoom*.transform](#zoom_transform).

To retrieve the zoom state, use *event*.transform on the current [zoom event](#zoom-events) within a zoom event listener (see [*zoom*.on](#zoom_on)), or use [d3.zoomTransform](#zoomTransform) for a given node. The latter is particularly useful for modifying the zoom state programmatically, say to implement buttons for zooming in and out.

<a href="#zoomTransform" name="zoomTransform">#</a> d3.<b>zoomTransform</b>(<i>node</i>)

Returns the current transform for the specified *node*. Internally, an element’s transform is stored as *element*.\_\_zoom; however, you should use this method rather than accessing it directly. If the given *node* has no defined transform, returns the [identity transformation](#zoomIdentity). The returned transform represents a two-dimensional [transformation matrix](https://en.wikipedia.org/wiki/Transformation_matrix#Affine_transformations) of the form:

*k* 0 *t<sub>x</sub>*
<br>0 *k* *t<sub>y</sub>*
<br>0 0 1

(This matrix is capable of representing only scale and translation; a future release may also allow rotation, though this would probably not be a backwards-compatible change.) The position ⟨*x*,*y*⟩ is transformed to ⟨*x* × *k* + *t<sub>x</sub>*,*y* × *k* + *t<sub>y</sub>*⟩. The transform object exposes the following properties:

* `x` - the translation amount *t<sub>x</sub>* along the *x*-axis.
* `y` - the translation amount *t<sub>y</sub>* along the *y*-axis.
* `k` - the scale factor *k*.

These properties should be considered read-only; instead of mutating a transform, use [*transform*.scale](#transform_scale) and [*transform*.translate](#transform_translate) to derive a new transform. Also see [*zoom*.scaleBy](#zoom_scaleBy), [*zoom*.scaleTo](#zoom_scaleTo) and [*zoom*.translateBy](#zoom_translateBy) for convenience methods on the zoom behavior. To create a transform with a given *k*, *t<sub>x</sub>*, and *t<sub>y</sub>*:

```js
var t = d3.zoomIdentity.translate(x, y).scale(k);
```

To apply the transformation to a [Canvas 2D context](https://www.w3.org/TR/2dcontext/), use [*context*.translate](https://www.w3.org/TR/2dcontext/#dom-context-2d-translate) followed by [*context*.scale](https://www.w3.org/TR/2dcontext/#dom-context-2d-scale):

```js
context.translate(transform.x, transform.y);
context.scale(transform.k, transform.k);
```

Similarly, to apply the transformation to HTML elements via [CSS](https://www.w3.org/TR/css-transforms-1/):

```js
div.style("transform", "translate(" + transform.x + "px," + transform.y + "px) scale(" + transform.k + ")");
```

To apply the transformation to [SVG](https://www.w3.org/TR/SVG/coords.html#TransformAttribute):

```js
g.attr("transform", "translate(" + transform.x + "," + transform.y + ") scale(" + transform.k + ")");
```

Or more simply, taking advantage of [*transform*.toString](#transform_toString):

```js
g.attr("transform", transform);
```

Note that the order of transformations matters! The translate must be applied before the scale.

<a href="#transform_scale" name="transform_scale">#</a> <i>transform</i>.<b>scale</b>(<i>k</i>)

Returns a transform whose scale *k₁* is equal to *k₀* × *k*, where *k₀* is this transform’s scale.

<a href="#transform_translate" name="transform_translate">#</a> <i>transform</i>.<b>translate</b>(<i>x</i>, <i>y</i>)

Returns a transform whose translation *t<sub>x1</sub>* and *t<sub>y1</sub>* is equal to *t<sub>x0</sub>* + *x* and *t<sub>y0</sub>* + *y*, where *t<sub>x0</sub>* and *t<sub>y0</sub>* is this transform’s translation.

<a href="#transform_apply" name="transform_apply">#</a> <i>transform</i>.<b>apply</b>(<i>point</i>)

Returns the transformation of the specified *point* which is a two-element array of numbers [*x*, *y*]. The returned point is equal to [*x* × *k* + *t<sub>x</sub>*, *y* × *k* + *t<sub>y</sub>*].

<a href="#transform_applyX" name="transform_applyX">#</a> <i>transform</i>.<b>applyX</b>(<i>x</i>)

Returns the transformation of the specified *x*-coordinate, *x* × *k* + *t<sub>x</sub>*.

<a href="#transform_applyY" name="transform_applyY">#</a> <i>transform</i>.<b>applyY</b>(<i>y</i>)

Returns the transformation of the specified *y*-coordinate, *y* × *k* + *t<sub>y</sub>*.

<a href="#transform_invert" name="transform_invert">#</a> <i>transform</i>.<b>invert</b>(<i>point</i>)

Returns the inverse transformation of the specified *point* which is a two-element array of numbers [*x*, *y*]. The returned point is equal to [(*x* - *t<sub>x</sub>*) / *k*, (*y* - *t<sub>y</sub>*) / *k*].

<a href="#transform_invertX" name="transform_invertX">#</a> <i>transform</i>.<b>invertX</b>(<i>x</i>)

Returns the inverse transformation of the specified *x*-coordinate, (*x* - *t<sub>x</sub>*) / *k*.

<a href="#transform_invertY" name="transform_invertY">#</a> <i>transform</i>.<b>invertY</b>(<i>y</i>)

Returns the inverse transformation of the specified *y*-coordinate, (*y* - *t<sub>y</sub>*) / *k*.

<a href="#transform_rescaleX" name="transform_rescaleX">#</a> <i>transform</i>.<b>rescaleX</b>(<i>x</i>)

Returns a [copy](https://github.com/d3/d3-scale#continuous_copy) of the [continuous scale](https://github.com/d3/d3-scale#continuous-scales) *x* whose [domain](https://github.com/d3/d3-scale#continuous_domain) is transformed. This is implemented by first applying the [inverse *x*-transform](#transform_invertX) on the scale’s [range](https://github.com/d3/d3-scale#continuous_range), and then applying the [inverse scale](https://github.com/d3/d3-scale#continuous_invert) to compute the corresponding domain:

```js
function rescaleX(x) {
  var range = x.range().map(transform.invertX, transform),
      domain = range.map(x.invert, x);
  return x.copy().domain(domain);
}
```

The scale *x* must use [d3.interpolateNumber](https://github.com/d3/d3-interpolate#interpolateNumber); do not use [*continuous*.rangeRound](https://github.com/d3/d3-scale#continuous_rangeRound) as this reduces the accuracy of [*continuous*.invert](https://github.com/d3/d3-scale#continuous_invert) and can lead to an inaccurate rescaled domain. This method does not modify the input scale *x*; *x* thus represents the untransformed scale, while the returned scale represents its transformed view.

<a href="#transform_rescaleY" name="transform_rescaleY">#</a> <i>transform</i>.<b>rescaleY</b>(<i>y</i>)

Returns a [copy](https://github.com/d3/d3-scale#continuous_copy) of the [continuous scale](https://github.com/d3/d3-scale#continuous-scales) *y* whose [domain](https://github.com/d3/d3-scale#continuous_domain) is transformed. This is implemented by first applying the [inverse *y*-transform](#transform_invertY) on the scale’s [range](https://github.com/d3/d3-scale#continuous_range), and then applying the [inverse scale](https://github.com/d3/d3-scale#continuous_invert) to compute the corresponding domain:

```js
function rescaleY(y) {
  var range = y.range().map(transform.invertY, transform),
      domain = range.map(y.invert, y);
  return y.copy().domain(domain);
}
```

The scale *y* must use [d3.interpolateNumber](https://github.com/d3/d3-interpolate#interpolateNumber); do not use [*continuous*.rangeRound](https://github.com/d3/d3-scale#continuous_rangeRound) as this reduces the accuracy of [*continuous*.invert](https://github.com/d3/d3-scale#continuous_invert) and can lead to an inaccurate rescaled domain. This method does not modify the input scale *y*; *y* thus represents the untransformed scale, while the returned scale represents its transformed view.

<a href="#transform_toString" name="transform_toString">#</a> <i>transform</i>.<b>toString</b>()

Returns a string representing the [SVG transform](https://www.w3.org/TR/SVG/coords.html#TransformAttribute) corresponding to this transform. Implemented as:

```js
function toString() {
  return "translate(" + this.x + "," + this.y + ") scale(" + this.k + ")";
}
```

<a href="#zoomIdentity" name="zoomIdentity">#</a> d3.<b>zoomIdentity</b>

The identity transform, where *k* = 1, *t<sub>x</sub>* = *t<sub>y</sub>* = 0.
