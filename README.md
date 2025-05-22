# Spatial Reasoner

> _A flexible 3D Spatial Reasoning framework_

__Content__: [Features](#features), [Usage](#usage), [Motivation](#motivation), [Spatial Objects](#spatial-objects), [Inference Syntax](#syntax-of-spatial-inference-pipeline), [Reference Systems](#spatial-reference_systems), [Adjustment](#spatial-adjustment), [BBox Sectors](#bbox-sectors), [Spatial Relations](#spatial-relations), [Taxonomy](#spatial-taxonomy), [Use Cases](#use-cases), [Paper](#paper), [Implementation](#implementation), [License](#license)

## Features

* __XR-focused__: reasoning on detected, _real_ and generated, _virtual_ 3D objects
* __Small__: easy to integrate with existing Computer Vision, 3D, VR, XR and AR toolkits
* __Powerful__: inferencing on 3D objects and their spatial relations
* __Extensive__: 100+ spatial predicates and corresponding relations 
* __Comprehensible__: simple yet powerful inference pipeline in textual specification
* __Appropriate__: handles fuzzyness and confidence of spatial situations
* __Flexible__: use Spatial Reasoner Syntax for 3D queries, object classification, spatial rule engines, semantic processing in 3D, voice interaction in space, with spatial-related LLM or with Large World Models (LWM), ...
* __Technology-agnostic__: run inference pipeline on various technology platforms
  * use spatial reasoning in mobile, desktop, Web and server projects
  * Spatial Reasoner Syntax as portable inference language
  * independent of left-handed or right-handed 3D coordinate system
* __Cross-platform__: Spatial Reasoner libraries available in various programming languages
  * [__SRswift__](https://github.com/metason/SRswift) library in __Swift__ for iOS, macOS and visionOS
  * [__SRpy__](https://github.com/metason/SRpy) library in __Python__ for server-side processing
  * [__SRcsharp__](https://github.com/NicolasLoth/SRcsharp) library in __C#__ with bindings for Unity in [__SRunity__](https://github.com/NicolasLoth/SRunity)
  * __SRjs__ library in __JavaScript__ (not yet started)

## Usage

The main process of the Spatial Reasoner framework consists of the following sequence: 
- match 3D items of your application to spatial objects in fact base  
- derive spatial attributes (done automatically)
- deduce spatial relations (done automatically, configurable)
- run pipeline of inference operations (defined as textual specification)
- access result (on demand repeat with updated fact base)

<details open>
<summary>Swift</summary>

```swift
// map detected or created 3D entities to SpatialObject instances
let obj1 = SpatialObject(id: "1", x:-1.5, y:1.2, z:0, w:0.1, h:1, d:0.1)
let obj2 = SpatialObject(id: "2", x:0, y:0, z:0, w:0.8, h:1, d:0.6)
let obj3 = SpatialObject(id: "3", x:0, y:0, z:1, w:0.8, h:0.8, d:0.8)
obj3.angle = .pi/2.0

// initialize reasoner and run pipeline
let sr = SpatialReasoner()
sr.load([obj1, obj2, obj3])
let pipeline = "filter(volume > 0.4) | pick(left AND above) | log()"
if sr.run(pipeline) {
    // result of processed pipeline as list of SpatialObject 
    let result = sr.result()  
    ...
}
```
</details>

<details>
<summary>Python</summary>

```python
# map detected or created 3D entities to SpatialObject instances
obj1 = SpatialObject( "1", position=Vector3(-1.5, 1.2, 0), width=0.1, height=1.0, depth=0.1)
obj2 = SpatialObject( "2", position=Vector3(0, 0, 0), width=0.8, height=1.0, depth=0.6)
obj3 = SpatialObject( "3", position=Vector3(0, 0, 1.6), width=0.8, height=0.8, depth=0.8)
obj3.angle = .pi/2.0
 
# initialize reasoner and run pipeline
sr = SpatialReasoner()
sr.load([obj1, obj2, obj3])
pipeline = "filter(volume > 0.4) | pick(left AND above) | log()"
if sr.run(pipeline):
    # result of processed pipeline as list of SpatialObject
    result = sr.result()
```
</details>

<details>
<summary>C#</summary>

```C#

```
</details>

## Motivation

The Spatial Reasoner framework deals with representing and reasoning about the topology of spatial 3D objects using derived attributes and deduced relations, such as the adjacency between or the topological arrangement and assembly of spatial objects. Spatial reasoning is the ability to conceptualize the three-dimensional relationships of objects in space and to evaluate spatial conditions in a specific context such as indoor or outdoor environments. Reasoning in the Spatial Reasoner library is executed as a succession of inference operations in a pipeline which takes spatial attributes of and spatial relations between objects into consideration. 

Example of a 3D inference pipeline:
```
deduce(topology)
| filter(id == 'ego') 
| pick(disjoint) 
| filter(volume > 0.5 AND NOT moving) 
| sort(disjoint.delta <)
```

Spatial fuzziness affects information retrieval in space. Object detection in state-of-the-art computer vision, machine learning, and Augmented Reality toolkits results in detected objects that vary their locations and do change and improve over time their orientations and boundaries in space. The object description is usually fuzzy and imprecise, yet some non-trivial conclusion can anyhow be deduced. The geometric confidence typically improves over time. Additionally, by taking spatial domain knowledge into account, semantic interpretation and therefore overall confidence can be improved. It is the goal of the Spatial Reasoner libraries to improve object detection with domain knowledge using spatial semantic and three-dimensional conditions.

## Spatial Objects

The spatial reasoner is prepared by loading the fact base with spatial objects. These are either provided by a 3D object detector (e.g. using computer vision, ML, and point cloud processing) or generated virtually in a scene graph. The fact base is loaded with an array of `SpatialObject` instances or by loading JSON data.

A `SpatialObject` is represented by a detected, _real_ or a generated, _virtual_ 3D entity in space by its oriented bounding box (bbox).
The oriented bounding box is axis-aligned to the horizontal ground plane and rotated around the centered up vector. 
The rotation in the horizontal plane is expressed as angle in radiants in counter-clockwise direction (and deduced as yaw in degrees in clockwise direction).
The position (x, y, z) of a `SpatialObject` is the center of the base area. The extend of the bbox is defined by width, height and depth (w, d, h). The `id` must be unique and should kept the same if updated over time. 

The significant attributes of a `SpatialObject` to be set are: `id, x, y, z, w, h, d, yaw`.

The object’s position and rotation around its vertical axis
may differ depending on the orientation of the underlying
left-handed or right-handed coordinate system. The concrete
implementation of a Spatial Reasoner library abstracts such
differences but requires the user to adopt a consistent reference
frame for accurate inference when loading the fact base with spatial objects.

### Declared Object Attributes

The following attributes may be set in the preparation phase before loading into the fact base:

```swift
{
  // non-spatial characteristics
  "id" : "object_1234", // unique identifier
  "existence" : "real",
  "cause" : "object_detected",
  "label" : "",
  "type" : "",
  // spatial characteristics
  "position" : [-0.95, 0, 1.5], // center of base area
  "angle" : 1.5707, // yaw angle in radiants
  "width" : 0.4,
  "height" : 0.5,
  "depth" : 0.3,
  "shape" : "unknown",
  "look" : "",
  "visible" : true,
  "focused" : false,
  "velocity" : [0, 0, 0],
  "confidence" : {
    "pose" : 0,
    "look" : 0,
    "dimension" : 0,
    "label" : 0
  }
}
```

### Deduced Object Attributes

The following attributes will be automatically deduced from the declared attributes:

```swift
{
  "center" : [-0.95, 0.25, 1.5], // center of object
  "yaw" : 90, // yaw angle in degrees
  "footprint" : 0.12,
  "frontface" : 0.2,
  "sideface" : 0.15,
  "surface" : 0.94,
  "volume" : 0.06,
  "perimeter" : 1.4,
  "baseradius" : 0.25,
  "radius" : 0.3535,
  "direction" : 0,
  "length" : 0.5,
  "conceptual" : false,
  "thin" : false,
  "long" : false,
  "equilateral" : false,
  "moving" : false,
  "immobile" : false,
  "motion" : "unknown",
  "real" : true,
  "virtual" : false,
  "azimuth" : 90
}
```

## Syntax of Spatial Inference Pipeline

The spatial inference pipeline is defined as textual specification. The pipeline is a linear sequence of inference operations which cover:
- __adjust__: optional setup to adjust nearby, sector, and max deviation settings
- __deduce__: optional setup to specify relation categories to be deduced
- __filter__: filter objects by matching spatial attributes
- __isa__: filter objects that belong to a type in a class hierarchy (taxonomy)
- __pick__: pick objects along their spatial relations
- __select__: select objects having spatial relations with others
- __sort__: sort objects by metric attributes or by spatial relations
- __slice__: choose a subsection of spatial objects 
- __calc__: calculate global variables in fact base
- __map__: calculate values of object attributes
- __produce__: create new spatial objects by a contextual specification
- __backtrace__: output spatial objects of operation some steps back in inference pipeline
- __reload__: reload and output all spatial objects of fact base
- __halt__: stop processing the inference pipeline (for debug purposes)
- __log__: log the current status of the inference pipeline

The inference operations within the pipeline are separated by "|". An inference operation follows the principle of _input - process - output_. Input and output data are list of spatial objects. The data flows from left to right along the pipeline so that the output of the former becomes the input of the next operation. The pipeline starts with all spatial objects of the fact base as input to the first operation.

Example:
```
filter(volume > 0.4) 
| pick(left AND above) 
| log()
```

The filter, pick, select, slice, produce and reload operations do change the list of output objects to be different from the input. All other operations do pass the list of input objects to the output, but may change sort order of or add attribute values to the spatial objects.

| Op | Syntax | Examples |
| -------- | ------- | -------- | 
| [__adjust__](#adjust-operation)  | `adjust(`_settings_`)` | `adjust(max gap 0.05); adjust(sector fixed 1.5); adjust(nearby dimension 2.0); adjust(nearby limit 4.0; max gap 0.1)` |
| [__deduce__](#deduce-operation)  | `deduce(`_relation-categories_`)` | `deduce(topology); deduce(connectivity); deduce(visibility); deduce(topology similarity)` |
| [__filter__](#filter-operation)  | `filter(`_attribute-conditions_`)` | `filter(id == 'wall1'); filter(width > 0.5 AND height < 2.4); filter(type == 'furniture'); filter(thin AND volume > 0.4)` |
| [__isa__](#isa-operation)  | `isa(`_class-type_`)` | `isa('Bed'); isa(Furniture); isa(Computer OR Monitor)` |
| [__pick__](#pick-operation)  | `pick(`_relation-conditions_`)` | `pick(near); pick(ahead AND smaller); pick(near AND (left OR right))` |
| [__select__](#select-operation)  | `select(`_relation ? attribute-conditions_`)` | `select(opposite); select(ontop ? id == 'table1'); select(on ? type == 'floor'); select(ahead AND smaller ? footprint < 0.5)` |
| [__sort__](#sort-by-attributes-operation)  | `sort(`_object-attribute_ [_comparator_]`)` | `sort(length); sort(volume); sort(width <); sort(width >)` |
| [__sort__](#sort-by-relations-operation)  | `sort(`_relation-attribute_ [_comparator_ _steps_]`)` | `sort(near.delta); sort(frontside.angle); sort(near.delta >); sort(disjoint.delta < -2)` |
| [__slice__](#slice-operation)  | `slice(`_range_`)` | `slice(1); slice(2..3); slice(-1); slice(-3..-1); slice(1..-2)` |
| [__calc__](#calc-operation)  | `calc(`_variable-assignments_`)` | `calc(cnt = count(objects); calc(maxvol = max(objects.volume); median = median(objects.height))` |
| [__map__](#map-operation)  | `map(`_attribute-assignments_`)` | `map(weight = volume * 140.0); map(type = 'double bed')` |
| [__produce__](#produce-operation)  | `produce(`_specification_ : _attribute-assignments_`)` | `produce(group : type = 'room'); produce(by : label = 'corner'; h = 0.02)` |
| [__backtrace__](#backtrace-operation)  | `backtrace(`_steps_`)` | `backtrace(-2)` |
| [__reload__](#reload-operation)  | `reload()` | `reload()` |
| [__halt__](#halt-operation)  | `halt()` | `halt()` |
| [__log__](#log-operation)  | `log(base 3D `_relations_`)` | `log(); log(base); log(3D); log(near right); log(3D near right)` |

### `adjust()` Operation

Adjust settings of the spatial reasoner to fit the actual context, environment and dominant object size. Call `adjust(...)` at the beginning of the inference pipeline. 
By setting a calculation schema, the corrsponding factor value can optionally be specified as well. Multiple settings within an  `adjust(...)` call are separated by `;`. 

```
adjust(max gap 0.02)     // set max deviation
adjust(max angle 0.1)    // set max angle delta
adjust(nearby fixed)     // set nearby calc schema to .fixed
adjust(nearby fixed 1.2) // set nearby calc schema and nearby factor 
adjust(nearby circle)    // set nearby calc schema to .circle
adjust(nearby sphere 2)  // set nearby calc schema to .sphere
adjust(nearby perimeter) // set nearby calc schema to .perimeter
adjust(nearby area)      // set nearby calc schema to .area
adjust(sector fixed 0.5) // set sector calc schema to .fixed
adjust(sector dimension) // set nearby calc schema to .dimension
adjust(sector perimeter) // set nearby calc schema to .perimeter
adjust(sector area 2)    // set nearby calc schema to .area
adjust(sector nearby)    // set nearby calc schema to .nearby
adjust(long ratio 4.0)   // set long ratio
adjust(thin ratio 10.0)  // set thin ratio
adjust(max gap 0.05; sector dimension; thin ratio 10.0)  
```

If `adjust(...)` is not called, the default values are used. See [Spatial Adjustment](#spatial-adjustment) for more details. 

### `deduce()` Operation

Specify the relation categories to be deduced by the spatial reasoner.
Call `deduce(...)` at the beginning of the inference pipeline, e.g., `deduce(visibility)` or `deduce(topology connectivity comparability)`.
When `deduce(...)` is not called, only the topology category is setup by default.

Spatial relation categories that can be set in `deduce(...)` are:
- [topology](Relations.md#topology) (including [proximity](Relations.md#proximity), [directionality](Relations.md#directionality), [adjacency](Relations.md#adjacency), [orientation](Relations.md#orientation), and [assembly](Relations.md#assembly))
- [connectivity](Relations.md#connectivity) (having contact)
- [sectoriality](Relations.md#sectoriality) (being in sector)
- [comparability](Relations.md#comparability)
- [similarity](Relations.md#similarity)
- [visibility](Relations.md#visibility) (seen from observer)
- [geography](Relations.md#geography)

See the [spatial relation categories](Relations.md) and the corresponding grouping of spatial predicates.

### `filter()` Operation

Filter spatial objects by their [declared](#declared-object-attributes) and [deduced](#deduced-object-attributes) attributes with the `filter(`_attribute conditions_`)` operation. Attribute conditions are expressions supporting value comparisons (`==, !=, >, >=, <, <=`), string comparison operators (`BEGINSWITH, CONTAINS, ENDSWITH, LIKE, MATCHES`) and boolean compounds (`AND, OR, NOT`).

Examples:
```
filter(id == 'ego')
filter(label == 'wall')
filter(label == 'wall' AND confidence.label > 0.7)
filter(type == 'furniture')
filter(width > 0.5 AND height < 2.4)
filter(volume < 1.5)
```

Conditions with Boolean attributes (`true, false`) may omit the comparison, such as:

```
filter(real)
filter(virtual AND NOT moving)
filter(thin AND volume > 0.4)
filter(long AND (visible OR length > 1.5))
```

### `isa()` Operation 

Filter objects that belong to a type in a class hierarchy (taxonomy) using
`isa(`_class type_`)` operation. 

Examples:
```
isa(Bed)
isa('Bed')
isa(Furniture)
isa(furniture)
isa(Computer OR Monitor)
isa(dining table OR desk OR workbench)
```

To determine the entity class of a spatial object, primary its type attribute and secondary its label attribute is used. From the determined class the isa operator checks for correspondance with the type name along the class hierachy in the loaded taxonomy. The type names are compared case-insensitive and string quotes are opional. Therefore `isa('furniture')` is equal to `isa(furniture)` and is equal to `isa(Furniture)`. The check for correspondance is considering the label of the class as well as optionally specified synonyms. Combine multiple type checks with `OR`. 

Before using the `ìsa()` operator, you have to load a domain-specific taxonomy as XML file in the OWL/RDF format: 

```swift
// load specific taxonomy in OWL/RDF format as XML file
let url = URL(string: "https://service.metason.net/ar/onto/test.owl") 
SpatialTaxonomy.load(from: url)

// initialize reasoner and run pipeline
let sr = SpatialReasoner()
...
```

Get details on [how to create a domain-specific taxonomy](Taxonomy.md) and supported features of the OWL/RDF ontology.

### `pick()` Operation 

Pick objects along their spatial relations with the
`pick(`_relation conditions_`)` operation. 

Examples:
```
pick(near)
pick(ahead AND smaller)
pick(near AND (left OR right))
```

### `select()` Operation

Select objects that have spatial relations with other objects using the `select(`_relation ? attribute conditions_`)` operation.  Attribute conditions are optional and will filter the referenced object. You can read the select() operation analog to a SQL-SELECT as "SELECT object WHERE ...". 
It can be read as "SELECT spatial objects WHERE spatial relation exists with other spatial objects [that fulfill the attribute conditions]". 

Examples:
```
select(opposite)
select(ontop ? label == 'table')
select(on ? type == 'floor')
select(ahead AND smaller ? footprint < 0.5)
```

With the NOT statement, the select() operator selects only spatial objects without the specified spatial relation. With the ALL statement, the select() operator selects only spatial objects that have the specified spatial relationship with all other objects. If NOT or ALL is missing, it is interpreted as ANY (can but does not have to be specified).

Examples:
```
select(opposite) // = select(ANY opposite)
select(NOT opposite)
select(ALL opposite)
```

### `sort()` by Attributes Operation

Sort objects by metric attributes with the  `sort(`_metric attribute_`)` operation.

 Examples:
 ```
 sort(length)
 sort(volume)
 sort(width <) // ascending
 sort(width >) // descending
 ```

### `sort()` by Relations Operation

Sort objects by spatial relations with the `sort(`_relation attribute_`)` operation.

Examples:
```
sort(near.delta)
sort(frontside.angle)
sort(near.delta <)  // ascending
sort(near.delta >)  // descending
sort(near.delta > -2)  // descending and backtracing 2 steps
```

The relations for sorting are selected between current and backtraced objects. By default, backtracing goes one step back in the inference pipeline; if necessary, the backtracing steps can be set higher.

### `slice()` Operation 

Choose a subsection of spatial objects with the `slice(`_range_`)` operation.

Examples:
```
slice(1)
slice(2..3)
slice(-1)
slice(-3..-1)
slice(1..-2)
``` 

### `calc()` Operation

Calculate global variables in fact base with the
`calc(`_variable assignments_`)` operation.

Examples:
```
calc(cnt = count(objects)
calc(maxvol = max(objects.volume)
calc(median = median(objects.footprint)
calc(vol = objects[0].volume; avgh = average(objects.height))
```

The calculated variables are accessible as `data.`_key_, e.g., in `filter(height > data.avgh)`.


Available function calls: `average(), sum(), count(), median(), mode(), stddev(), min(), max()`.



### `map()` Operation

Set or calculate values of local object attributes with the `map(`_attribute assignments_`)` operation. 

```
map(shape = 'cylindrical')
map(type = 'bed')
map(weight = volume * 140.0)
```

### `produce()` Operation

Create new spatial objects according to a contextual specification and add them to fact base. Optionally change attributes of the new instances. The new  `id` is set automatically by concatenatiog the specification with the `id` of the input object separated with a colon (`spec:id`). When generated objects already exist (identified with their automatically generated `spec:id`), then they will be updated and not created again. Change the `id`in case of enforcing a new object creation.   

```
produce(on : ...)    // create object at footprint face
produce(by : ...)    // create object at touching edge
produce(at : ...)    // create object at meeting face
produce(copy : ...)  // create a copy
produce(group : ...) // create a group object containing all input objects, aligned with largest input object 
produce(sector_type : ...)  // create object at bbox sector 
```

### `backtrace()` Operation

Output spatial objects of the input of the operation some steps back in the inference pipeline. In case steps are not set, backtracing goes by default one step back in the inference pipeline. The sign of the step number has no effect.

```
backtrace(-1)  // backtrace one step
backtrace(1)   // same as backtrace(-1)
backtrace()    // same as backtrace(-1), default value is -1
backtrace(-3)  // goes back 3 steps to take input as its output
```

### `reload()` Operation

Reload all spatial objects of the fact base into the pipeline, also the new produced ones.

### `halt()` Operation

Stops further processing of the inference pipeline. Used for debug purposes: add the `halt()` operation at any position in the pipeline without the need of deleting following operations.

### `log()` Operation

Log files are used for debug purposes and are saved per default in the Downloads folder.

- `log()` or `log(selected relations)`
  - Log as markdown file
  - Overview list of spatial objects
  - Inference pipeline
  - Spatial relations graph (all or selection)
  - Connectivity graph (in case connectivity relations are activated)
  - List of deduced relations
- `log(3D)`
  - Scene of fact base as 3D file (USDZ format in SRswift, GLTF else)
  - Spatial objects rendered in 3D for visualization
- `log(base)`
  - Fact base as JSON file
  - Array of spatial objects with their atrributes
  - Calculated variables with their values
  - Chain of results from processed pipeline

Example of a spatial relation graph:
```mermaid
graph LR;
    subj
    obj
    ego
    obj -- left --> subj
    obj -- seenright --> subj
    ego -- left --> subj
    subj -- seenleft --> obj
    ego -- right --> obj
    obj -- right --> ego
```

Example list of deduced spatial relations:
* obj is near to subj (near 𝛥:1.05  𝜶:0.0°)
* obj is left of subj (left 𝛥:0.04  𝜶:0.0°)
* obj is seen right of subj (seenright 𝛥:0.94  𝜶:0.0°)
* ego is near to subj (near 𝛥:3.14  𝜶:153.0°)
* ...

Example of a connectivity graph:
```mermaid
graph TD;
    wall2 <-- by --> wall1
    wall2 <-- by --> wall3
    wall3 <-- by --> wall4
    wall4 <-- by --> wall1
    door -- in --> wall1
    window -- in --> wall2
    picture -- at --> wall4
    wall1 -- on --> floor
    wall2 -- on --> floor
    wall3 -- on --> floor
    wall4 -- on --> floor
    table -- on --> floor
```

## Spatial Reference Systems

The interpretation of some predicates of spatial relations are depending on the frame of reference. E.g., predicates such as left, right, in front, and at back have different meaning in different reference systems. Additionally, in English language the semantic of spatial predicates is sometimes vague and it is hardly possible to distinquish between terms and their synonyms (e.g., over, above, ontop). Therefore, the meaning of all spatial predicates used in the Spatial Reasoner framework are clearly specified. Although the ordinary meaning of the terms has been taken into consideration, the specification in the Spatial Reasoner framework might not correspond with its daily use in spoken English language.


The interpretation of spatial predicates and their corresponding relations are only valid in specifc reference systems:
- __World Coordinate System (WCS)__: spatial relations are encoded relative to a global reference point and its orientation.
- __Object Coordinate System (OCS)__: spatial relations are encoded relative to the local position and orientation of an object
- __Egocentric Coordinate System (ECS)__: spatial relations are encoded relative to the position and view direction of an observer
- __Geodetic Coordinate System (GCS)__: spatial relations are encoded relative to earth's projected latitude (north/south) and longitude (east/west)


## Spatial Adjustment

The spatial reasoner can be adjusted to fit the actual context, environment and dominant object size.
Set adjustment parameters before executing a inference pipeline or before calling the relate() method.
SpatialReasoner has its own local adjustment that should be set upfront programmatically. Otherwise use the `ajust()` operation to set the adjustment parameters.

```swift
class SpatialAdjustment {
    // Max deviations
    var maxGap:Float = 0.05 // max distance of deviation in all directions in meters
    var maxAngleDelta:Float = 0.05 * .pi // max delta of yaw orientation in both directions in radiants 
    // Sector size
    var sectorSchema:SectorSchema = .nearby
    var sectorFactor:Float = 1.0 // multiplying result of claculation schema
    var sectorLimit:Float = 2.5 // maximal length
    // Vicinity
    var nearbySchema:NearbySchema = .circle
    var nearbyFactor:Float = 2.0 //multiplying radius sum of object and subject (relative to size) as max distance
    var nearbyLimit:Float = 2.5 // maximal absolute distance
    // Proportions
    var longRatio:Float = 4.0 // one dimension is factor larger than both others
    var thinRatio:Float = 10.0 // one dimension is 1/factor smaller than both others
}
```

Calculation schema to determine nearby radius of objects's bounding box.

```swift
public enum NearbySchema {
    case fixed // use nearbyFactor as fix nearby radius
    case circle // use base circle radius of bbox multiplied with nearbyFactor
    case sphere // use sphere radius of bbox multiplied with nearbyFactor
    case perimeter // use base perimeter multiplied with nearbyFactor
    case area // use area multiplied with nearbyFactor
}
```

Calculation schema to determine sector size for extruding area to partition space along objects's bounding box (see next chapter).

```swift
public enum SectorSchema {
    case fixed // use sectorFactor for extruding area
    case dimension // use same dimension as object multiplied with factor
    case perimeter // use base perimeter multiplied with factor
    case area // use area multiplied with factor
    case nearby // use nearby settings for extruding
}
```

## BBox Sectors

In order to reason about the relative positioning of objects,
the Spatial Reasoner partitions the space around each reference object into sectors. Conceptually, the bounding box is extended along the three orthogonal axes—left-right, ahead-behind, and over-under—to form a 3 ×3 ×3 grid of 27 sectors.

![on](images/o_u.png) ![lr](images/l_r_a_b.png) ![lo_lu_ro_ru](images/lo_lu_ro_ru.png) ![alo_aro_blo_bro](images/alo_aro_blo_bro.png)

- [Zero Divergency](Sectors.md#zero-divergency) (`i`) for inner/inside sector
- [Single Divergency](Sectors.md#single-divergency) (`l, r, a, b, o, u`) denotes a
single principal direction offset (e.g., an object in sector
`l` is within the left sector of the reference bbox).
- [Double Divergency](Sectors.md#double-divergency) (`al, ar, bl, br, lo, lu,
ro, ru, ao, au, bo, bu`) captures a combination
of two directions (e.g., `ar` means “ahead-right,” `lu`
means “left-under”).
- [Triple Divergency](Sectors.md#tripe-divergency) (`alo, aru, blu, bru`, etc.) represents more nuanced positions like “ahead-left-over” or
“behind-right-under.”


Different sector schema options (e.g., fixed, dimension, nearby) and a sector factor can customize the size of each sector region, ensuring it fits the scale of the environment (e.g., large warehouse vs. small indoor room).
Example of different spatial adjustments and calculation scheme:

![adjustable sector size](images/sectors.png)

Left image: `.fized`; middle image: `.dimension`; right image: `.nearby`

See detailed description of all [BBox sectors](Sectors.md).


## Spatial Relations

Beyond bounding-box geometry, the Spatial Reasoner supports a rich set of qualitative predicates that formalize on a symbolic level how objects relate to each other. Spatial relationships are represented as ___subject - predicate - object___ patterns.

Spatial predicates are organized in categories:

- [Proximity](Relations.md#proximity): `near, far`
- [Directionality](Relations.md#directionality ): `left, right, ahead, behind, above, below`
- [Adjacency](Relations.md#adjacency): `leftside, frontside, beside, ontop`, ...
- [Orientation](Relations.md#orientation): `aligned, orthogonal, opposite`, ...
- [Connectivity](Relations.md#connectivity): `on, in, by, at`
- [Sectoriality](Relations.md#sectoriality): `i, o, u, al, alo`, ... (referring to bbox sectors)
- [Assembly](Relations.md#assembly): `disjoint, inside, touching, meeting`, ...
- [Visibility](Relations.md#visibility): `seenleft, infront`, ... (relative to an observer)
- [Comparability](Relations.md#comparability): `bigger, smaller, longer, shorter`, ...
- [Similarity](Relations.md#similarity): `samewidth, samevolume, congruent`, ...
- [Geography](Relations.md#geography): `north, west, south, northeast`, ...

Each predicate has an underlying geometric or semantic
definition. For instance, near and far rely on distance thresholds defined by the nearby schema, whereas on, in, and
by infer direct contact or containment taking a max gap
into consideration. Meanwhile, seenleft and infront consider
an observer’s viewpoint to relate two objects.

![ontop](images/ontop.png) ![leftside](images/leftside.png) 

Left image: subj is `ontop` of obj; right image: subj is at `leftside` of obj

![orthogonal](images/orthogonal.png) ![seenleft](images/seenleft.png) 

Left image: subj is `orthogonal` to obj; right image: subj is `seenleft` of obj


See detailed description of all [spatial relations](Relations.md).

## Spatial Taxonomy

### Spatial Relation Terminology

All spatial predicates with their code, preposition, verb, synonyms, antonym and inverse definitions are specified and are available as terminology in [terms.json](terms.json) as JSON file. These spatial terms build a foundation to integrate the Spatial Reasoner framework with LLM, NLP, and TTS. 

### Type System for Spatial Objects

Spatial objects have a `type` attribute which can be embedded into a class hierachy to reflect entity inheritance relations.
Before using the `ìsa()` operator, you have to load a domain-specific taxonomy into the Spatial Reasoner library. As a base import for an ontology that defines an entity taxonomy, the OWL/RDF format is supported. 

See more details on [spatial taxonomy](Taxonomy.md).

## Use Cases

### Property Filters 


Select a spatial object by its unique identifiier, e.g.:
```
filter(id == 'id1234')
```

Filter spatial objects by type attributes, e.g.:
```
filter(type == 'chair' OR type == 'table')
```

Filter spatial objects by boolean attributes, e.g.:
```
filter(virtual AND NOT moving)
```

Filter spatial objects by non-spatial attributes, e.g.:
```
filter(label == 'table' AND confidence.label > 0.7)
```

### Spatial Queries

Select spatial objects by their spatial attributes, e.g.:
```
filter(footprint > 0.5 && height > 1.5)
```

Pick spatial objects by their spatial relations, e.g.:
```
pick(near AND (left OR behind))
```

Select spatial objects by their attributes and their relations, e.g.:
```
filter(height < 0.6 && height > 0.25 && width < 1.3 && length > 1.8)
| select(beside ? type == 'Wall')
| sort(volume)
```

Get nearest spatial object from observer:
```
filter(id == 'observer') 
| pick(disjoint) 
| sort(disjoint.delta <)
| slice(1)
```

Get second left spatial object of type "picture" seen from observer:
```
deduce(topology)
| filter(id == 'ego') 
| pick(ahead AND left AND disjoint) 
| filter(type == 'picture') 
| sort(disjoint.delta > 2)
| slice(2)
```

The sort() operation is backtracing 2 steps to take as input the 'egp' object to compare the relations between 'ego' object and filtered objects with type 'picture'.

### Object Classification

Classify spatial objects by their attributes, e.g.:
```
filter(height < 0.6 && height > 0.25 && width > 1.5 && length > 1.8)
| map(type = 'double bed'; type = 'furniture'; confidence = 0.5)
```

Classify spatial objects by their attributes and their topological arrangement, e.g.:
```
filter(height > 1.5 && width > 1.0 && depth > 0.4)
| select(backside ? type == 'wall')
| map(type = 'cabinet'; type = 'furniture'; confidence = 0.75)
```

### Production Rules

Create a dublicate of a spatial object, e.g.:
```
filter(id == '1234')
| produce(copy : label = 'copy of 1234'; y = 2.0)
```

Agregate spatial objects by creating a group covering all children's bbox. E.g., create a room object by grouping the existing walls:
```
filter(type == 'wall')
| produce(group : label = 'room')
```

Generate objects at the position where they are connected (at connectivity relations) . E.g., create a spatial object at the corner of each touching wall:
```
filter(type == 'wall')
| produce(by : label = 'corner'; h = 0.02)
```

## Paper

Our paper will be published and presented at [ICVARS 2025](https://www.icvars.org) conference.

A [preprint](http://arxiv.org/abs/2504.18380) is available on arXiv.

Please cite our work if you use Spatial Reasoner.

```bibtex
@inproceedings{SR25,
  title     = {Spatial Reasoner: A 3D Inference Pipeline for XR Applications},
  author    = {Steven Häsler and Philipp Ackermann},
  booktitle = {Proceedings of the ICVARS `25},
  year      = {2025},
  publisher = {Association for Computing Machinery}
}
```

## Implementation

The design of the Spatial Reasoner Syntax has been first implemented and validated in [SRswift](https://github.com/metason/SRswift) using the Swift programming language by [Philipp Ackermann](mailto:philipp@metason.net?subject=Spatial%20Reasoner). Feel free to take this implementation as reference to implement the Spatial Reasoner Syntax in another programming language. 

Please consider the following hints:
- It is a clear intention that the textual specification of the spatial inference pipeline is cross-platform, so that all pipelines confirming to the Spatial Reasoner Syntax can run on any SR implementation.
- Make it transparent when only a subset of the Spatial Reasoner Syntax is covered by your impelementation.
- The Swift programming language is strongly typed which forces SRswift to manage two seperate instances of the fact base in parallel and in sync: 1) as list of objects in `var objects:[SpatialObject]`, and 2) as a list of key-value dictionaries in `var base:Dictionary<String, Any>`. The dictionary representation is needed to provide read/write access to the predicate evaluator and expression interpreter.
- If you implement a Spatial Reasoner in a programming language that is not strongly typed (e.g., Python or JavaScript), you may avoid a duplicate representation of the fact base.
- The trickiest part of implementing another SR library might be the interpretation of predicates and expressions. A predicate is a logical statement that evaluates to a Boolean value (true or false) and is used in SR to evaluate conditions of attributes and existance of relations. An expression does perform operations or calculations and is used in SR for attribute evaluation and assignments. Analyze in an early stage `SpatialInference.swift` to elaborate how to solve predicates and expressions in your implementation. 
- The interpretation of the Spatial Reasoner Syntax should be independent from the orientation of the left-handed or right-handed 3D coordinate system. The SRswift implementation is internally using a right-handed coordinate system. Take special care in translating the SRswift reference implementation if the underlying coordinate system of the 3D toolkit of your implementation is different. 
- In SRswift, spatial terms are hard-coded. All terms for spatial relations have been exported and are available in [terms.json](terms.json) so that in other implementations you may import the terms instead of hard-coding them. 
- SRswift includes extensive test cases. It is a good idea to also validate your own implementation with automatic tests.
- Some test cases are "misused" to generate visualizations that are included in this documentation. These test cases (named *Vis) do not have to be covered in your tests. 

If you plan to release your implementation as Open Source, please feel free to contact [Philipp](mailto:philipp@metason.net?subject=Spatial%20Reasoner) to get your SR library listed under [features](#features) in this document. Ideas for improvements are also welcome.

## License

Released under the [Creative Commons CC0 License](LICENSE).

## Contact

Philipp Ackermann, philipp@metason.net
