# Curling Stone Placement JSON

[日本語](README.ja.md)

A JSON text format for exchanging curling stone placements between applications and tools.

Format version: **v1.1**, with v1.0 compatibility.

## Language policy

The English version is the canonical specification. The Japanese version is provided as an official translation. This policy also applies to the paired MCE Implementation Specification. Translation discrepancies must be reviewed and corrected against the English text.

## Documents

| Document | English | Japanese |
| --- | --- | --- |
| Format structure, scale, precision, and compatibility | [Specification](spec/JSON_Text_Format_v1.1.md) | [一般仕様書](spec/JSON_Text_Format_v1.1.ja.md) |
| MCE v1.3.0 Reader/Writer support and behavior | [Implementation Specification](implementations/MCE_v1.3.0.md) | [実装仕様書](implementations/MCE_v1.3.0.ja.md) |

## Specification and implementation support

v1.1 adds houseRadius to the v1.0 structure. Its numerical value defines the coordinate scale; its unit is any non-empty label. The Specification recommends radius-ratio normalization and suitable coordinate resolution.

The Specification defines valid data and its meaning. Reader support, Writer formatting, and recovery from invalid input are documented separately in each Implementation Specification. See the Implementation Specification above for application-specific behavior.

## License

This documentation is licensed under the [Creative Commons Attribution-NoDerivatives 4.0 International License (CC BY-ND 4.0)](https://creativecommons.org/licenses/by-nd/4.0/).

Attribution should be given to **RC spica Factory**.

This license applies to the specifications and documentation in this repository. It does not restrict the implementation of this specification or the creation, use, modification, or exchange of JSON data conforming to this specification. This includes implementing applications, Readers, Writers, and tools, and editing, converting, storing, and distributing conforming JSON data. The NoDerivatives condition applies to the documentation, not to such implementations or JSON data.

The documentation may be shared and redistributed in unadapted form under the license terms. The license does not grant permission to share or redistribute adapted versions of the documentation.

See [LICENSE](LICENSE) for the license notice and the [official legal code](https://creativecommons.org/licenses/by-nd/4.0/legalcode.en) for the full terms.
