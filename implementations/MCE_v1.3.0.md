# Mobile Curling Editor v1.3.0 Implementation Specification

[日本語](MCE_v1.3.0.ja.md)

The English version is the canonical Implementation Specification. The Japanese version is the official translation. The general format and MCE's implementation support are distinct.

This document covers the released Mobile Curling Editor v1.3.0:

- Android Build 39
- iOS Build 18

## 1. Format and supported input

The format is defined in [JSON Text Format v1.1 Specification](../spec/JSON_Text_Format_v1.1.md). Its general scale reference, arbitrary non-empty unit labels, radius-ratio conversion, and recommended resolution are distinct from MCE-specific validation and placement limits.

In the general format, `houseRadius.value` is a finite positive scale reference for the 12-foot circle. `m`, `ft`, and `px` are recommended labels, not an exhaustive enumeration. General conversion can use `R_target / R_input` without understanding the unit label. Avoiding rejection solely for an unknown label is a recommendation for general Readers, not an implemented MCE v1.3.0 rule.

The general resolution recommendation is approximately 1 mm or finer at radius 1.829 m, with an increment of approximately `houseRadius.value / 1829`. It is not an exact rounding rule or a unit-specific decimal-place requirement.

**MCE v1.3.0 provides partial support for JSON format v1.1.** Absent `houseRadius` identifies v1.0 and present `houseRadius` identifies v1.1.

| Input | MCE v1.3.0 Set behavior |
| --- | --- |
| No `houseRadius` | Accept as v1.0 metre coordinates, equivalent to standard 1.829 m |
| `{"value":1.829,"unit":"m"}` | Accept with coordinate factor 1 |
| `{"value":6,"unit":"ft"}` | Multiply all coordinates by 0.3048 to obtain metres |
| px, nonstandard m/ft, unknown unit, or uninterpretable radius | Reject the entire input |

Radius comparison tolerance is 0.0001 in the specified unit's value. This is not arbitrary scale normalization. For 6 ft input, both houseCenter and stone positions use the fixed factor 0.3048, not `1.829 / 6` (approximately 0.3048333).

Thus radius 300 with unit px or radius 100 with unit game-unit is valid as a general coordinate-system declaration, but rejected by MCE v1.3.0 at Set. Format validity does not imply support by this Reader. This document does not expand that support.

## 2. Set pipeline and whole-input rejection

Normal Set and 2D Code Generate use the same validation and application path.

```text
Parse and validate the whole input
→ Validate stones, normalize coordinates, remove exact duplicates
→ Convert to internal coordinates, snap boundaries, check playable area, finalize candidates
→ Clear the current board
→ Place stones
→ Update history and Autosave
→ Regenerate MCE JSON and update buttons
```

The entire input is rejected before changing the board when:

- JSON parsing fails, including when the input text is malformed JSON.
- Parsing succeeds, but the top-level JSON value is not an object.
- Any of metadata, positiveDirection, houseCenter, or stones is missing.
- positiveDirection is not a string or not one of the four allowed directions.
- houseCenter is not an object or x/y is not a finite JSON number.
- stones is not an array.
- houseRadius is present but invalid or unsupported.
- More than eight stones of either color pass basic validation.
- An object contains duplicate property names, at any nesting level, for known and unknown keys alike.

Whole-input rejection leaves the board, history, Autosave, and JSON field unchanged. Only the presence of metadata is checked; its type and value do not restrict the generating application. Unrecognized properties are not otherwise used for placement and do not cause rejection merely by being present; the duplicate-property-name check still applies.

## 3. Per-stone validation and skipping

Stones are processed in array order, without sorting by input number. A stone is skipped individually when:

- It is not an object or lacks a required field.
- color is not the string red or yellow.
- position.x/y is not a finite JSON number.
- number is not an integer value from 1 through 16.
- Normalized coordinates are non-finite or outside the float representation range.
- Its normalized coordinates exactly match an earlier stone.
- It remains outside the playable sheet after internal conversion and boundary correction.
- No corresponding unused stone object is available.

Numeric strings are not treated as numbers. An integer-valued number such as 1.0 is accepted. The input number is checked for an integer value in the accepted range and is not directly displayed on a stone. The Reader does not check input numbers for uniqueness or correspondence to distance order. Otherwise acceptable records with duplicate numbers can therefore be applied, and successful Set regenerates unique numbers in distance order. Such acceptance is implementation-specific handling of invalid input; duplicate numbers are not valid under the general specification.

**Color counts are incremented immediately after basic color/position/number validation, before duplicate removal and playable-area checks.** A record later skipped as duplicate or out of bounds still counts toward the color limit. Invalid basic records do not count. The two limits of eight imply a maximum of sixteen; there is no separate “17 or more” test. Whole-input rejection is MCE's handling of input exceeding the general format's per-color limit; the general format does not prescribe this recovery behavior.

Duplicate detection uses exact equality of the double x/y pair after conversion to metres, origin correction, and direction normalization. It ignores color and discards the later record. No additional duplicate check is performed after conversion to the placement representation or boundary snapping.

An empty array is valid. If every stone is skipped individually but whole-input validation succeeds, Set succeeds with an empty board.

## 4. Coordinate normalization

After unit conversion, subtract houseCenter from position, then normalize to LeftDown:

| Input direction | Transformation of the relative (x, y) |
| --- | --- |
| LeftDown | (x, y) |
| RightDown | (-x, y) |
| LeftUp | (x, -y) |
| RightUp | (-x, -y) |

houseCenter need not be (0, 0). Extreme finite values alone do not cause whole-input rejection; invalid or out-of-range normalized results are handled per stone.

## 5. Output and renumbering

After Generate or successful Set, on-sheet stones are sorted by distance from the current house center and renumbered 1 through N across both colors. Numbers in the output are unique; no stable tie-breaking order is specified for equally distant stones. Fixed output fields and an illustrative stone are:

```json
{ "metadata": "MobileCurlingEditor",
  "positiveDirection": "LeftDown",
  "houseCenter": {"x": 0, "y": 0},
  "houseRadius": {"value": 1.829, "unit": "m"},
  "stones": [
{"color":"red","position":{"x":0.315,"y":-0.428},"number":1}
]}
```

Coordinates are written with three decimal places. This is a rule of the MCE v1.3.0 Writer, not of the general format. Top-level lines after the first start with two spaces; each stone occupies one line and commas occur at line ends. Non-empty output places `]}` on a separate final line. The dedicated empty-board generator ends with `  "stones": []}`. Whitespace is not a format compatibility condition.

Successful Set replaces the input text with MCE-standard JSON describing the resulting board.

## 6. JSON boundary handling

MCE compensates for small rounding differences near sheet boundaries when exporting and applying JSON.

- Before rounding coordinates to three decimal places, the Writer clamps JSON Y to the intersection of the legal ranges for House Top and House Bottom. This boundary adjustment changes neither X nor actual board positions.
- At Set, very small overshoots of either X or Y are snapped to the current sheet boundary before the playable-area check. This accommodates boundary overshoots caused by coordinate rounding during JSON round trips.
- Stones still outside the playable area are skipped individually. This correction does not expand the playable area or make arbitrary out-of-bounds positions acceptable.

These are MCE Reader/Writer behaviors, not general format requirements. Boundary adjustment and rounding mean exported coordinates need not preserve board coordinates exactly.

## 7. JSON Save and Load

The JSON field is editable and multiline, and starts with empty-board JSON. Load only fills the field; it does not automatically Set. Unsupported-scale documents can be loaded as text and are rejected when Set or Generate performs validation.

The suggested initial filename is `<SaveFilenamePrefix>yyyyMMdd_HHmmss.json`. `<SaveFilenamePrefix>` is user-configurable text, up to six characters in this version, and is retained in the application's settings. MCE supplies the generated name to the save picker as the initial filename; it does not fix the final filename. The user can change the filename in the picker before saving, subject to the picker and destination's filename rules.

On devices, MCE creates a temporary file with this generated name and exports it through Native File Picker, which is also used for Load. JSON Load is aborted if a content:// path is returned. In the Editor, the generated name is passed directly to the save dialog, and file dialogs initially use the SaveData folder.

## 8. Relationship to 2D Code

Generate validates and applies input through the normal Set path, then encodes the regenerated MCE-standard JSON in the QR code. Failure does not produce a new preview. Successful Generate also updates history, Autosave, numbering, and JSON. It does not preserve the input text verbatim.

QR generation uses ZXing.Net, QR_CODE, UTF-8, and Margin 4. This MCE transport behavior adds no structural requirements to JSON Text Format.

Image Read and camera Scan only place the decoded string in the JSON field. They do not automatically validate JSON, Set, or change the board. Cancelling Read leaves the field unchanged.

## 9. Internal Autosave and exclusions

Internal Autosave/Restore uses a separate format and path from external JSON Set. Autosave does not use the external Writer's three-decimal formatting or shared Y clamping; Restore does not use external Set validation or its boundary correction.

Arbitrary finite positive radius-ratio input normalization, px and unknown-unit support, and arbitrary-scale output are not implemented v1.3.0 features. This document makes no commitment to their delivery in a future version.
