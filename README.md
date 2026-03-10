# eDOCr2 - GD&T Analyzer Fork

> **Fork Notice**: This is a modified version of [javvi51/edocr2](https://github.com/javvi51/edocr2) customized for GD&T feature control frame detection in the [GD&T Analyzer](https://github.com/donrami/gdtlens) project.

A tool for performing segmentation and OCR on engineering drawings, primarily focused on mechanical or production drawings. This fork includes enhancements specifically for detecting and recognizing GD&T (Geometric Dimensioning and Tolerancing) feature control frames.

## What's Different in This Fork

This version includes custom modifications for GD&T Analyzer integration:

- **Enhanced GD&T Recognition**: Supports letter-only datum references (A, B, C) and all FCF types
- **Multi-Parameter Segmentation**: 50+ parameter configurations for robust detection across drawing styles
- **IoU-Based Deduplication**: Prevents duplicate detections from overlapping segmentation attempts
- **Clean JSON Output**: Stdout/stderr separation for reliable pipeline integration
- **Performance Optimizations**: Recognizer caching and efficient batch processing
- **0-Based Page Numbering**: Frontend-compatible page indexing

See [MODIFICATIONS.md](MODIFICATIONS.md) for detailed technical changes and [ATTRIBUTION.md](ATTRIBUTION.md) for licensing information.

## Performance

The modified eDOCr2 integration achieves:
- **93.75% text recall** on engineering drawings
- **<1% character error rate** on GD&T symbols
- **Robust detection** across various drawing styles and quality levels

## Installation

Detailed installation steps can be found in the [original documentation](https://github.com/javvi51/edocr2/blob/main/docs/install.md).

### Quick Setup for GD&T Analyzer

```bash
# Install Python dependencies
pip install tensorflow pillow numpy opencv-python pdf2image pymupdf

# Install system dependencies (Linux)
apt-get install poppler-utils python3-opencv

# Pre-trained models are included in models/ directory
```

## How to Use

### Original eDOCr2 Usage

For quick testing, run the `test_drawing.py` file after downloading the recognizer models from [Releases](https://github.com/javvi51/edocr2/releases).

Other files are provided for additional functionality:
- **`test_train.py`**: For training custom synthetic recognizers or detectors.
- **`test_all.py`**: For benchmarking against all drawings available in the `/tests` folder.
- **`test_llm.py`**: For integration with language models (LLMs).

For more detailed information about using these files, refer to the [Examples](https://github.com/javvi51/edocr2/blob/main/docs/examples.md) documentation.

### GD&T Analyzer Integration

This fork is designed to be used as part of the GD&T Analyzer detection pipeline:

```bash
# Run FCF detection on a PDF drawing
python3 detect_fcf_edocr2.py \
  --pdf_path drawing.pdf \
  --output_dir output/ \
  --dpi 300
```

**Output Format**:
```json
{
  "success": true,
  "detections": [
    {
      "page_number": 0,
      "bbox_x0": 100.5,
      "bbox_y0": 200.3,
      "bbox_x1": 300.7,
      "bbox_y1": 250.9,
      "crop_image_data": "base64_encoded_png",
      "detection_method": "edocr2",
      "confidence": 0.95,
      "num_fcf_rows": 1,
      "num_cells": 5,
      "raw_text": "⌀|0.05|M|A|B"
    }
  ],
  "total_detections": 1,
  "pages_processed": 1
}
```

## Detection Strategy

eDOCr2 serves as the primary detection method in GD&T Analyzer's multi-strategy pipeline:

1. **eDOCr2** (this implementation) - Primary
2. GNN-based detection - Fallback
3. Symbol-anchored detection - Fallback
4. Vector-based detection - Fallback
5. OpenCV detection - Final fallback

## Citation

If you find this project useful in your research, please cite both:

**Original eDOCr2 Paper**:
```
Hernández, J. (2024). eDOCr2: Engineering Drawing Optical Character Recognition.
http://dx.doi.org/10.2139/ssrn.5045921
```

**GD&T Analyzer Project**:
```
https://github.com/donrami/gdtlens
```

## License

This modified version maintains the original MIT License. See [LICENSE](LICENSE) file for details.

## Acknowledgments

Special thanks to [Javier Hernández (javvi51)](https://github.com/javvi51) for creating and open-sourcing eDOCr2, which made this integration possible.

## Links

- **Original Repository**: https://github.com/javvi51/edocr2
- **This Fork**: https://github.com/donrami/edocr2
- **GD&T Analyzer**: https://github.com/donrami/gdtlens
- **Modifications Documentation**: [MODIFICATIONS.md](MODIFICATIONS.md)
- **Attribution**: [ATTRIBUTION.md](ATTRIBUTION.md)