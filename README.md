# Planet Field Boundary Extension Specification

- **Title:** Planet Field Boundary
- **Identifier:** <https://fiboa.github.io/planet-extension/v0.1.0/schema.yaml>
- **Property Name Prefix:** planet
- **Extension Maturity Classification:** Proposal
- **Owner**: @cholmes

This document explains the Planet Field Boundary Extension to the
[Field Boundaries for Agriculture (fiboa) Specification](https://github.com/fiboa/specification).

Planet produces fully automated field boundaries from satellite imagery for anywhere on earth. The full 
[technical specification](https://planet.widen.net/s/5vq8w5wjvf/2403.08_mar-9444-field-boundaries-technical-specification-sheet-3) explains all 
the details of the methodology, and the attributes of the data. Data is currently distributed in a GeoPackage. It can
be converted to fiboa with the [fiboa cli] - you can see [the code for Planet conversion](https://github.com/fiboa/cli/blob/main/fiboa_cli/datasets/planet_afb.py).

Any Planet field boundary output can be converted with `fiboa convert planet_afb -i FIELD_BOUNDARIES_v1.0.0_S2_P1M-20221201T000000Z_fb.gpkg -o planet-fiboa-2022-12.parquet

- Examples:
  - [GeoJSON](examples/geojson/)
  - [GeoParquet](examples/geoparquet/)
- [Schema](schema/schema.yaml)
- [Changelog](./CHANGELOG.md)

## Properties

The properties in the table below can be used in these parts of fiboa documents:

- [ ] Collection
- [x] Feature Properties

| Property Name   | Type   | Description |
| --------------- | ------ | ----------- |
| planet:micd | float | **REQUIRED** Maximum Inscribed Circle Diameter (MICD)  is an intuitive proxy for the width of a field, even in case of rotated and narrow-but-curved fields.  |
| planet:ca_ratio | float  | **REQUIRED** The Circumference-Area ratio (CA ratio) is calculated by dividing the circumference of a given polygon by the square root of its area. This ratio is then adjusted so that a circle corresponds to 0, and scaled so that a square corresponds to 1 |
| planet:qa | uint8 |  **REQUIRED** Quality Assessment attribute, either 0 (good, micd > 30m), 1 (quality not guaranteed, micd < 30m) or 2 (polygons intersecting data availability grid) |

### planet:mcid

While the polygon area is a general descriptor for the size of a polygon, it carries no information about its shape.
While evaluating and validating the FB product, it was observed that the shape of the polygon correlates more
with the performance of the algorithm, rather than its size only. This has to do with the limitations in spatial
resolution of the input imagery, which affect the minimum side, e.g. height or width, of the polygon that can be
detected.

A very narrow but long polygon can have a large value of area, but nevertheless cannot be reliably detected by
the algorithm due to its narrower side. Maximum Inscribed Circle Diameter was identified to be a suitable metric
for estimating the polygon width. The maximum inscribed circle is the largest circle which can be inscribed within
the polygon, see Figure 2. MICD is added as an attribute, and used to flag polygons of uncertain accuracy.

![mcid diagram](https://github.com/cholmes/planet-fb-extension/assets/407017/5a50d9ca-0ba1-478f-86e9-5ce049ca3c98)

### planet:ca_ratio

A normalized Circumference-Area ratio is added to each polygon to additionally describe its shape. The formula
for the CA ratio is as follows:

![CA Ratio formula](https://github.com/cholmes/planet-fb-extension/assets/407017/657b8d3d-eee4-4d45-adaa-c73ffe8052f9)


The CA ratio can be used in conjunction to the Area and MICD attributes to identify polygons of circular shape,
e.g. with CA ratio close to 0, of squared shape, e.g. with CA ratio around 1, or of more irregular shape, having CA
ratio larger than 10. Examples of CA ratio are shown in Figure 3.

![CA Ratio diagram](https://github.com/cholmes/planet-fb-extension/assets/407017/77cd2c6f-3c5e-4f5e-81d0-c4aa5b32825b)

### planet:qa

A quality assessment (QA) attribute denotes polygons with known limitations. The attribute is an unsigned
integer. The values presented in the table below are provided for the current version of the FB product.

The ‘qa’ attribute presents three values:

| Value | Description |
| ----- | ------------|
| 0 | Polygons for which the validation scores are representative, i.e. with MICD > 30 m. |
| 1 | Polygons for which the validation scores are not representative, i.e. with MICD < 30 m. The quality for these polygons cannot be guaranteed. |
| 2 | Polygons that are intersecting the border of the data availability grid |

The provided example illustrates four types of polygons. Polygons outside of the AOI are removed from the output. The remaining ones (depicted with solid and diagonal
fill) intersect with the AOI and are therefore kept in the output. Polygons that are intersecting the border of the data availability grid are also kept in the output and are
assigned a value of qa==2. We suggest removing such polygons from further analyses, as they only partially cover the underlying field.

![qa diagram](https://github.com/cholmes/planet-fb-extension/assets/407017/204f29b0-f723-4e22-a278-2ae49206a823)

## Contributing

See the [contributing guideline](CONTRIBUTING.md) for more details.
