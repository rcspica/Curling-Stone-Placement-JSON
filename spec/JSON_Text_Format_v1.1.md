# Curling Stone Placement JSON Text Format v1.1 Specification

[日本語](JSON_Text_Format_v1.1.ja.md)

The English version is the canonical specification; the Japanese version is the official translation.

- Current specification version: **v1.1**
- Base version: **v1.0**

## 1. Purpose

This specification defines a JSON text format for exchanging curling stone placements between applications and tools.

The structural data format and application-specific display formatting are separate concerns. JSON whitespace, line breaks, and indentation do not change the meaning of the data and are not compatibility requirements.

## 2. Version identification

This format does not currently include a dedicated version field.

| Condition | Version |
| --- | --- |
| The top-level `houseRadius` field is absent. | v1.0 |
| The top-level `houseRadius` field is present. | v1.1 |

The only structural difference between v1.0 and v1.1 is the presence of `houseRadius`.

A different versioning mechanism may be considered if the format is extended further. No additional versioning rule is defined by this specification.

## 3. v1.0 structure

### 3.1 Top-level fields

| Field | Type | English description |
| --- | --- | --- |
| `metadata` | any JSON value | Contains descriptive information, such as the name of the application or tool that generated the JSON text. |
| `positiveDirection` | string | Indicates the positive direction of the coordinate system. Four values are allowed. |
| `houseCenter` | object | Coordinates of the house center. |
| `stones` | array | An array of stone placements. It is an empty array when no stones are present. |

### 3.2 `positiveDirection`

The coordinate system is defined with reference to one house, but the format does not restrict representable stone positions to the area around that house. The orientation below does not define a valid coordinate range between the back line and the hog line. Positions elsewhere on the sheet can be represented in the same coordinate system; an implementation's supported placement area is a separate matter.

`Up`, `Down`, `Left`, and `Right` are defined based on an orientation where the back line is above the house and the hog line is below it.

The four values specify the positive directions of the x and y axes in this reference orientation:

| Value | Positive x direction | Positive y direction |
| --- | --- | --- |
| `LeftDown` | Left | Down |
| `LeftUp` | Left | Up |
| `RightDown` | Right | Down |
| `RightUp` | Right | Up |

This reference is independent of an application's current screen orientation.

### 3.3 `houseCenter`

| Field | Type | English description |
| --- | --- | --- |
| `x` | number | X coordinate of the house center. |
| `y` | number | Y coordinate of the house center. |

### 3.4 `stones`

Each element of `stones` contains the following fields.

| Field | Type | English description |
| --- | --- | --- |
| `color` | string | Specifies `"red"` or `"yellow"` as the stone color. |
| `position` | object | Coordinates representing the stone position. |
| `number` | number | A unique number assigned to each stone in ascending order of distance from the house center. |

A valid placement contains at most eight stones of each color. Reader recovery behavior for input exceeding this limit, such as rejecting the entire input or ignoring excess stones, is implementation-dependent and is not specified here.

`number` values must be unique across all stones in the same placement, regardless of color. Stones closer to the house center receive lower numbers. If two or more stones are at the same distance from the house center, their relative numbering order is implementation-dependent; they must still have different numbers. Reader handling of invalid or duplicate numbers is implementation-dependent and is not specified here.

The numbering rule does not require the `stones` array itself to be sorted by `number`.

The `position` object contains:

| Field | Type | English description |
| --- | --- | --- |
| `x` | number | X coordinate of the stone position. |
| `y` | number | Y coordinate of the stone position. |

In v1.0, `position.x` and `position.y` represent real curling-scale distances in metres and are recorded to a maximum of three decimal places.

This specifies the maximum decimal precision, not fixed-point formatting. Trailing zeros are not required.

## 4. v1.1 extension

v1.1 adds the top-level `houseRadius` field to the v1.0 structure. No other structural change is introduced.

### 4.1 `houseRadius.value`: scale reference

`houseRadius.value` specifies the numerical radius of the 12-foot circle in the JSON coordinate system. It defines the scale of the entire coordinate system, including `houseCenter` and `stones[].position.x/y`. It can represent real-size coordinates, image pixels, game or virtual-world coordinates, and scaled model coordinates.

| Field | Type | English description |
| --- | --- | --- |
| `value` | number | A finite positive number usable for coordinate conversion, defining the radius of the 12-foot circle. |
| `unit` | string | Any non-empty string labeling the unit or coordinate system. |

Zero, negative numbers, non-finite values, and values that cannot be treated as valid numbers are not valid scale references.

Examples of coordinate-system declarations

```json
"houseRadius":{"value":1.829,"unit":"m"}
```

```json
"houseRadius":{"value":6,"unit":"ft"}
```

```json
"houseRadius":{"value":300,"unit":"px"}
```

```json
"houseRadius":{"value":100,"unit":"game-unit"}
```

These declarations are valid under the general v1.1 specification; support by a particular Reader is a separate question.

### 4.2 `houseRadius.unit`: label and recommended spellings

`unit` describes the unit or coordinate system used by `value` and the coordinates. It is not restricted to an enumeration. The following are representative recommended spellings, not an exhaustive list of allowed values.

| Label | English meaning |
| --- | --- |
| `"m"` | metres |
| `"ft"` | feet |
| `"px"` | pixels |

A Reader does not need to understand the label to normalize coordinates using the radius ratio. Readers are recommended not to reject otherwise valid data solely because the unit label is unknown. The common specification does not require case folding or synonym matching: `"m"`, `"M"`, and `"meter"` need not be interpreted as equivalent labels.

### 4.3 Radius-ratio normalization

For input radius `R_input` and target radius `R_target`, coordinates in the same origin and direction can be converted using:

```text
scaleFactor = R_target / R_input
targetPosition = inputPosition × scaleFactor
```

Apply the same factor to `houseCenter` when converting absolute coordinates; equivalently, subtract the input house center first and scale the relative coordinates. Origin correction and `positiveDirection` remain separate from scale conversion.

For a target radius of 1.829, a radius of 300 uses `1.829 / 300`, and a radius of 6 uses `1.829 / 6`. The numerical conversion does not require interpreting the `unit` string. This is coordinate-scale normalization, not a requirement to perform physical feet-to-metres conversion.

### 4.4 Recommended coordinate resolution

Coordinate values should retain a resolution approximately equivalent to 1 mm or finer when the radius of the 12-foot circle is interpreted as 1.829 m.

An approximate coordinate increment corresponding to 1 mm is:

```text
recommendedResolution = houseRadius.value / 1829
```

| `houseRadius.value` | Approximate 1 mm increment |
| --- | ---: |
| 1.829 | 0.001 |
| 6 | 0.00328 |
| 300 | 0.164 |

This is an interoperability recommendation, not an exact quantization or rounding rule for Writers. Finer resolution is permitted. v1.1 imposes no unit-specific decimal-place limits, fixed significant-figure count, or mandatory trailing zeros on `houseRadius.value` or coordinates. Preserve sufficient precision in the radius used as the scale reference as well. The v1.0 precision rule in section 3.4 does not become a v1.1 requirement.

For example, `6` and `420` may be written without adding zeros as `6.000` or `420.00`. Actual Reader support and Writer precision are documented separately in each Implementation Specification; an application's fixed output precision is not a common v1.1 rule.

## 5. Compatibility

### 5.1 v1.0 input in a v1.1 implementation

If `houseRadius` is absent, the JSON is v1.0 and must be interpreted as equivalent to:

```json
"houseRadius":{"value":1.829,"unit":"m"}
```

This allows a v1.1-compatible application to read v1.0 JSON without modifying the data.

### 5.2 v1.1 input in a v1.0 implementation

A v1.0-only implementation may be able to parse v1.1 JSON if it ignores the unknown `houseRadius` field. This is structural compatibility only.

If the numerical coordinate scale differs from the v1.0 scale (radius 1.829), a v1.0-only implementation cannot correctly interpret it merely by parsing the structure. A different unit label alone does not determine numerical scale compatibility.

### 5.3 Two meanings of compatibility

| Compatibility | English description |
| --- | --- |
| Structural compatibility | The JSON can be parsed and known fields can be read. |
| Semantic coordinate compatibility | The coordinate scale and unit can be interpreted correctly. |

Structural compatibility does not guarantee semantic coordinate compatibility.

## 6. JSON examples

Whitespace is included in some examples for readability. It is not part of the structural requirements.

### 6.1 v1.0, no stones, minified

```json
{"metadata":{},"positiveDirection":"LeftDown","houseCenter":{"x":0,"y":0},"stones":[]}
```

### 6.2 v1.0, 16 stones, minified

This example contains eight red and eight yellow stones, numbered by distance from the house center.

```json
{"metadata":{},"positiveDirection":"LeftDown","houseCenter":{"x":0,"y":0},"stones":[{"color":"red","position":{"x":-0.326,"y":0.133},"number":1},{"color":"yellow","position":{"x":0.349,"y":0.347},"number":2},{"color":"red","position":{"x":-0.191,"y":0.474},"number":3},{"color":"red","position":{"x":-0.761,"y":0.377},"number":4},{"color":"red","position":{"x":-0.534,"y":1.057},"number":5},{"color":"yellow","position":{"x":1.061,"y":0.554},"number":6},{"color":"red","position":{"x":0.204,"y":-1.287},"number":7},{"color":"red","position":{"x":-0.948,"y":0.960},"number":8},{"color":"red","position":{"x":0.769,"y":1.187},"number":9},{"color":"yellow","position":{"x":-0.409,"y":1.417},"number":10},{"color":"yellow","position":{"x":1.217,"y":0.881},"number":11},{"color":"yellow","position":{"x":1.518,"y":1.188},"number":12},{"color":"yellow","position":{"x":-0.431,"y":1.879},"number":13},{"color":"red","position":{"x":-1.759,"y":1.130},"number":14},{"color":"yellow","position":{"x":0.965,"y":2.268},"number":15},{"color":"yellow","position":{"x":-0.038,"y":3.439},"number":16}]}
```

### 6.3 v1.1, metres, minified

```json
{"metadata":{},"positiveDirection":"LeftDown","houseCenter":{"x":0,"y":0},"houseRadius":{"value":1.829,"unit":"m"},"stones":[]}
```

### 6.4 v1.1, feet, minified

```json
{"metadata":{},"positiveDirection":"LeftDown","houseCenter":{"x":0,"y":0},"houseRadius":{"value":6,"unit":"ft"},"stones":[]}
```

## 7. Implementation recommendations

The following recommendations support interoperability but do not add structural requirements to the format.

- Determine v1.0 or v1.1 by the absence or presence of `houseRadius`.  
- A v1.1 reader should interpret a missing `houseRadius` as `1.829 m`.  
- Do not use whitespace, indentation, or line breaks to determine compatibility.  
- A v1.0 reader that ignores unknown fields must not assume semantic coordinate compatibility when the numerical radius scale differs from 1.829.  
- For v1.1, use the radius-ratio principle and the resolution recommendation in section 4; unknown labels alone should not cause rejection of otherwise valid data.  

## 8. Scope boundaries

This document does not define requirements that were not established for v1.0 or v1.1. In particular, it does not introduce an internal version field or any structural difference other than `houseRadius`.

This specification primarily defines the meaning of valid JSON data. Unless explicitly stated for interoperability, specific recovery or rejection behavior for invalid input is implementation-dependent. An implementation's acceptance or recovery of invalid input does not make that input valid under this specification.

The presence of additional properties not defined here does not, by itself, make a document invalid. This specification does not assign meanings to those properties or prescribe how Readers handle them.
