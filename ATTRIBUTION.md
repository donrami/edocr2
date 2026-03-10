# eDOCr2 Attribution

This directory contains a modified version of eDOCr2, an academic-grade OCR system for engineering drawings.

## Original Project

- **Repository**: https://github.com/javvi51/edocr2
- **Author**: Javier Hernández (javvi51)
- **License**: MIT License (see LICENSE file in this directory)
- **Paper**: "eDOCr2: Engineering Drawing Optical Character Recognition"

## Fork Information

A fork of this project is maintained at:
- **Fork**: https://github.com/donrami/edocr2
- **Branch**: gdtlens-integration

## Modifications for GD&T Analyzer

This version includes custom modifications for integration with the GD&T Analyzer application:

### 1. OCR Pipeline Modifications (`edocr2/tools/ocr_pipelines.py`)
- Fixed invalid escape sequence warning in `allowed_exceptions_eng` set
- Enhanced GD&T recognition to support letter-only references (A, B, C)
- Removed digit-only filter to detect all FCF types

### 2. Integration Wrapper (`detect_fcf_edocr2.py`)
- Created custom detection script for GD&T Analyzer integration
- Implemented multi-parameter segmentation with 50+ configurations
- Added IoU-based deduplication to prevent duplicate detections
- Fixed stdout/stderr separation for clean JSON output
- Implemented recognizer caching for performance
- Added 0-based page numbering for frontend compatibility

### 3. Model Files
- Pre-trained models included:
  - `models/recognizer_gdts.keras` - GD&T symbol recognizer
  - `models/recognizer_gdts.txt` - GD&T alphabet definition
  - `models/recognizer_dimensions_2.keras` - Dimension recognizer
  - `models/recognizer_dimensions_2.txt` - Dimension alphabet

## Performance

The modified eDOCr2 integration achieves:
- **93.75% text recall** on engineering drawings
- **<1% character error rate** on GD&T symbols
- **Robust detection** across various drawing styles and quality levels

## Usage in GD&T Analyzer

eDOCr2 serves as the primary detection strategy (Strategy A) in the detection pipeline:

1. **eDOCr2** (this implementation) - Primary
2. GNN-based detection - Fallback
3. Symbol-anchored detection - Fallback
4. Vector-based detection - Fallback
5. OpenCV detection - Final fallback

## Citation

If you use this modified version in academic work, please cite both:

1. The original eDOCr2 paper
2. The GD&T Analyzer project: https://github.com/donrami/gdtlens

## License

This modified version maintains the original MIT License. See LICENSE file for details.

## Acknowledgments

Special thanks to Javier Hernández (javvi51) for creating and open-sourcing eDOCr2, which made this integration possible.
