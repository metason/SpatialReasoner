# Spatial Reasoner

> _A flexible 3D Spatial Reasoning library_

## Features

* __3D first__: designed from ground up for 3D (not a 2D/GIS extension)
* __Small__: easy to integrate with existing Computer Vision, 3D, VR, XR and AR toolkits
* __Powerful__: inferencing on 3D objects and their spatial relations
* __Extensive__: 100+ spatial predicates and corresponding relations 
* __Comprehensible__: simple yet powerful inference pipeline in textual specification
* __Appropriate__: handles fuzzyness and confidence of spatial situations
* __Flexible__: use for 3D queries, object classification, spatial rule engines, semantic processing in 3D, voice interaction in space, with spatial-related LLM or with Large World Models (LWM), ...
* __Technology-agnostic__: run on various technology platforms
  * use in mobile, desktop, Web and server projects
  * portable inference language
  * independent of left-handed or right-handed 3D coordinate system
* __Cross-platform__: library available in various programming languages
  * [__SRswift__](https://github.com/metason/SRswift) repository in Swift for iOS, macOS and visionOS
  * [__SRpy__](https://github.com/metason/SRpy) repository in Python (work in progress)
  * __SRmono__ / SRdotnet / SRunity / SRcsharp? repository in C# for Unity (work in progress)
  * __SRjs__ repository in JavaScript (not yet)

## Usage

The main process of the Spatial Reasoner library consists of the following sequence: 
- match 3D items of your application to spatial objects in fact base  
- derive spatial attributes (done automatically)
- deduce spatial relations (done automatically, configurable)
- run pipeline of inference operations (defined as text)
- access result (on demand repeat with updated fact base)

<details open>
<summary>Swift</summary>

```swift
// map detected or created 3D entities to SpatialObject instances
let obj1 = SpatialObject(id: "1", position: .init(x: -1.5, y: 0, z: 0), width: 0.1, height: 1.0, depth: 0.1)
let obj2 = SpatialObject(id: "2", position: .init(x: 0, y: 0, z: 0), width: 0.8, height: 1.0, depth: 0.6)
let obj3 = SpatialObject(id: "3", position: .init(x: 0, y: 0, z: 1.6), width: 0.8, height: 0.8, depth: 0.8)
obj3.angle = .pi/2.0

// initialize reasoner and run pipeline
let sr = SpatialReasoner()
sr.load([obj1, obj2, obj3])
let pipeline = "filter(volume > 0.4) | pick(left AND above) | log()"
if sr.run(pipeline) {
    // access list of SpatialObject resulted from processed pipeline
    let result = sr.result()  
    ...
}
```
</details>

<details>
<summary>Python</summary>
tbd
</details>

<details>
<summary>C#</summary>
tbd
</details>

## Motivation

This library deals with representing and reasoning about the topology of spatial 3D objects using derived attributes and deduced relations, such as the adjacency between or the topological arrangement of spatial objects. Spatial reasoning is the ability to conceptualize the three-dimensional relationships of objects in space and to evaluate spatial conditions in a specific indoor or outdoor context. Reasoning in the Spatial Reasoner library is executed as a succession of inference operations in a pipeline which takes spatial attributes of and spatial relations between objects into consideration. 

Spatial fuzziness affects information retrieval in space. Object detection in state-of-the-art computer vision, machine learning, and Augmented Reality toolkits results in detected objects that vary their locations and do change and improve over time their orientations and boundaries in space. The object description is usually fuzzy and imprecise, yet some non-trivial conclusion can anyhow be deduced. The geometric confidence typically improves over time. Additionally by taking spatial domain knowledge into account, semantic interpretation and therefore overall confidence can be improved. It is the goal of the Spatial Reasoner library to improve object detection with domain knowledge using spatial semantic and three-dimensional conditions.

## Syntax of Spatial Inference Pipeline

The spatial inference pipeline is defined as text specification. The pipeline is a linear sequence of inference operations which cover:
- __filter__: filter objects by matching spatial attributes
- __pick__: pick objects along their spatial relations
- __select__: select objects having spatial relations with others
- __log__: log the current status of the processing pipeline
- __deduce__: optional setup to specify relation categories to be deduced
- __sort__: sort objects by metric attributes of spatial objects
- __calc__: calculate global variables in fact base
- __map__: calculate values of (new) object attributes
- __produce__: create new spatial objects relative to relations and add to fact base

The inference operations within the pipeline are separated by "|". An inference operation follows the principle of _input - process - output_. Input and output data are list of spatial objects. The data flows from left to right along the pipeline so that the output of the former becomes the input of the next operation. The pipeline starts with all spatial objects of the fact base as input to the first operation.

Example:
```
filter(volume > 0.4) 
| pick(left AND above) 
| log()
```

The filter, pick, and select operations do change the list of output objects to be different from the input. All other operations do pass the list of input objects to the output, but may change sort order or add attribute values of the spatial objects.

| Op | Syntax | Examples |
| -------- | ------- | -------- | 
| __filter__  | `filter(`_attribute conditions_`)` | `filter(id == 'wall1'); filter(width > 0.5 AND height < 2.4); filter(type == 'furniture'); filter(thin AND volume > 0.4)` |
| __pick__  | `pick(`_relation conditions_`)` | `pick(near); pick(ahead AND smaller); pick(near AND (left OR right))` |
| __select__  | `select(`_relation ? attribute conditions_`)` | `select(ontop ? id == 'table1'); select(on ? type == 'floor'); select(ahead AND smaller ? footprint < 0.5)` |
| __log__  | `log(base 3D `_relations_`)` | `log(); log(base); log(3D); log(near right); log(3D near right)` |
| __deduce__  | `deduce(`_relation categories_`)` | `deduce(topology); deduce(connectivity); deduce(visibility)` |
| __sort__  | `sort(`_metric attribute_`)` | `sort(length); sort(volume); sort(width <); sort(width >)` |
| __sort__  | `sort(`_relation attribute_`)` | `sort(near.delta); sort(frontside.angle); sort(near.delta <);` |
| __map__  | `map(`_attribute assignment_`)` | `map(weight = volume * 140.0)` |
| __calc__  | `calc(`_variable assignment_`)` | `calc(cnt = objects.@count); calc(maxvol = objects.volume@max; median = objects.volume@median)` |
| __produce__  | `produce(`_relation conditions_` : `_type wxdxh_`) | `produce(container : room); produce(wall by wall on floor : corner 0.2x0.2x0.2)` |



## Logging Operation log()

Log files are used for debug purposes and are saved per default in the Downloads folder.

- `log()` or `log(selected relations)`
  - Log as markdown file
  - Overview list of spatial objects
  - Spatial relations graph (all or selection)
  - Contact graph
  - List of relations
- `log(3D)`
  - Scene of fact base as USDZ file
  - Spatial objects rendered in 3D for visualization
- `log(base)`
  - Fact base as JSON file
  - Array of spatial objects with their atrributes
  - Calculated variables with their values
  - Chain of results from processed pipeline

## Setup Operation deduce()

Spatial predicate categories of relations:
- topology
- connectivity (= contacts)
- sectoriality (= sectors)
- comparability
- visibility
- geography

## Spatial Reference Systems

The interpretation of some predicates of spatial relations are depending on the frame of reference. E.g., predicates such as left, right, in front, and at back have different meaning in different reference systems. Additionally, in English language the semantic of spatial predicates is sometimes vague and it is hardly possible to distinquish between terms and their synonyms (e.g., over, above, ontop). Therefore, the meaning of all spatial predicates used in the Spatial Reasoner library are clearly specified. Although the ordinary meaning of the terms has been taken into consideration, the specification in the Spatial Reasoner library might not corrspond with its daily use in spoken English language.


The interpretation of spatial predicates and their corresponding relations are only valid in specifc reference systems:
- __World Coordinate System (WCS)__: spatial relations are encoded relative to a global reference point and its orientation.
- __Object Coordinate System (OCS)__: spatial relations are encoded relative to the local position and orientation of an object
- __Egocentric Coordinate System (ECS)__: spatial relations are encoded relative to the position and view direction of an observer
- __Geodetic Coordinate System (GCS)__: spatial relations are encoded relative to earth's projected latitude (north/south) and longitude (east/west)

## Spatial Adjustment

The spatial reasoner can be adjusted to fit the actual context, environment and dominant object size.
Set adjustment parameters before executing a inference pipeline or before calling the relate() method.
SpatialReasoner has its own local adjustment that should be set upfront.

```swift
class SpatialAdjustment {
    // Max deviations
    var maxgap:Float = 0.05 // max distance of deviation in all directions in meters
    var maxangle:Float = 0.05 * .pi // max delta of yaw orientation in radiants in both directions
    // Sector size
    var sectorSchema:SectorSchema = .wide
    var sectorFactor:Float = 1.0 // sectorFactor is multiplying the result of claculation schema
    var sectorLimit:Float = 2.5 // sectorLimit is maximal length
    var fixSectorLenght:Float = 0.25
    var wideSectorLenght:Float = 10.0
    // Vicinity
    var nearbyFactor:Float = 1.0 // nearbyFactor is multiplying radius sum of object and subject (relative to size) as max distance
    var nearbyLimit:Float = 2.0 // nearbyLimit is maximal absolute distance
    // Proportions
    var longRatio:Float = 4.0 // one dimension is factor larger than both others
    var thinRatio:Float = 10.0 // one dimension is 1/factor smaller than both others
}
```

Calculation schema to determine sector size for extruding area to partition space along objects's bounding box (see next chapter).

```swift
public enum SectorSchema {
    case fixed // use specified fix lenght for extruding area
    case dimension // use same dimension as object multiplied with factor
    case perimeter // use area perimeter multiplied with factor
    case area // use area multiplied with factor
    case nearby // use nearby settings for extruding
    case wide // use fix wide
}
```

## BBox Sectors

Object-centric partitioning of space

Different BBox sectors size depending on calculation schema and adjustment settings.

Example of different spatial adjustments and calculation scheme:

![adjustable sector size](images/sectors.png)
left image: `.fized`, middle image: `.dimension`, right image: `.nearby`

See detailed description of all [BBox sectors](Sectors.md).


## Spatial Object


## Spatial Relations

- Spatial relation: subject - predicate - object
- Spatial predicates

See detailed description of all [spatial relations](Relations.md).
