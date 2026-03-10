# eDOCr2 Modifications for GD&T Analyzer

This document details all modifications made to the original eDOCr2 codebase for integration with GD&T Analyzer.

## Modified Files

### 1. `edocr2/tools/ocr_pipelines.py`

**Line 388**: Fixed invalid escape sequence warning
```python
# Before:
allowed_exceptions_eng = set('''?—!@#~;¢«#_%\&€$»[é]®§¥©'™="~'£<*""I|ZNOXiLlk \n''')

# After:
allowed_exceptions_eng = set('''?—!@#~;¢«#_%&€$»[é]®§¥©'™="~'£<*""I|ZNOXiLlk \n''')
```

**Line 165**: Removed digit-only filter to detect all FCF types
```python
# Before:
if any(char.isdigit() for char in pred):
    return pred
return None

# After:
if pred and pred.strip():
    return pred
return None
```

**Reason**: Original filter excluded FCFs with only letter references (A, B, C), which are valid in GD&T specifications.

## New Files Created

### 1. `detect_fcf_edocr2.py` (Integration Wrapper)

**Purpose**: Bridges eDOCr2 with GD&T Analyzer's detection pipeline

**Key Features**:
- Multi-parameter segmentation (50+ configurations)
- IoU-based deduplication
- Stdout/stderr separation for clean JSON output
- Recognizer caching for performance
- 0-based page numbering
- Base64 PNG crop generation

**Architecture**:
```python
main()
  ├─> _main_impl(args, json_output_stream)
  │     ├─> convert_from_path() [pdf2image]
  │     └─> run_edocr2_on_page() [for each page]
  │           ├─> segment_img() [50+ parameter sets]
  │           ├─> deduplicate_boxes() [IoU-based]
  │           ├─> ocr_gdt() [recognition]
  │           └─> format to app schema
  └─> Output JSON to stdout
```

**Segmentation Strategy**:
- Tries multiple `GDT_thres` values: 0.0001 to 0.1
- Tries multiple `binary_thres` values: 80, 100, 127, 150, 200
- Collects ALL boxes from ALL attempts
- Deduplicates using IoU threshold of 0.5

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

### 2. `ATTRIBUTION.md`

Documents original project, fork information, and modifications.

### 3. `MODIFICATIONS.md` (this file)

Detailed technical documentation of all changes.

## Integration Points

### Backend Service (`server/services/pdfProcessor.ts`)

**Method**: `detectEDOCr2()`
```typescript
private async detectEDOCr2(pdfPath: string, outputDir: string) {
  const result = await this.runCommand(
    "python3",
    [
      "server/python/detect_fcf_edocr2.py",
      "--pdf_path", pdfPath,
      "--output_dir", outputDir,
      "--dpi", "300"
    ],
    120000 // 120 second timeout
  );
  
  const parsed = JSON.parse(result.stdout);
  return {
    success: true,
    detected_fcfs: parsed.detections.map(det => ({
      page: det.page_number,
      bbox: [det.bbox_x0, det.bbox_y0, det.bbox_x1, det.bbox_y1],
      detection_method: det.detection_method,
      confidence: det.confidence,
      crop_image_base64: det.crop_image_data
    }))
  };
}
```

### Detection Strategy Hierarchy

```
processDrawing()
  ├─> detectEDOCr2() ← PRIMARY (this implementation)
  ├─> detectGNN() ← Fallback if eDOCr2 fails
  ├─> detectSymbolAnchored() ← Fallback if GNN fails
  ├─> detectVectorBased() ← Fallback if symbol fails
  └─> detectOpenCV() ← Final fallback
```

## Performance Optimizations

### 1. Recognizer Caching
```python
_recognizer_cache = None

def run_edocr2_on_page(page_image, page_num, dpi, recognizer_gdt=None):
    global _recognizer_cache
    
    if recognizer_gdt is None:
        if _recognizer_cache is None:
            # Load once
            _recognizer_cache = Recognizer(alphabet=gdt_alphabet, weights=None)
            _recognizer_cache.model.load_weights(str(model_path))
        else:
            recognizer_gdt = _recognizer_cache
```

**Benefit**: Avoids reloading 50MB+ model for each page

### 2. Stdout Redirection
```python
def main():
    original_stdout = sys.stdout
    sys.stdout = sys.stderr  # Redirect all prints to stderr
    
    try:
        _main_impl(args, original_stdout)
    finally:
        sys.stdout = original_stdout
```

**Benefit**: Prevents progress bars and warnings from corrupting JSON output

### 3. TensorFlow Suppression
```python
import os
os.environ['TF_CPP_MIN_LOG_LEVEL'] = '3'
tf.get_logger().setLevel('ERROR')

import warnings
warnings.filterwarnings('ignore')
```

**Benefit**: Reduces noise in logs, improves parsing reliability

## Testing

### Manual Test
```bash
python3 server/python/detect_fcf_edocr2.py \
  --pdf_path test_drawing.pdf \
  --output_dir temp/test \
  --dpi 300
```

### Expected Output
- Clean JSON on stdout
- Progress messages on stderr
- Exit code 0 on success

## Known Limitations

1. **High DPI Sensitivity**: Requires very low `GDT_thres` values (0.0001-0.1) for 300 DPI images
2. **Table Structure Dependency**: Works best with clear table structures
3. **Processing Time**: 15-30 seconds for complex drawings due to 50+ segmentation attempts

## Future Improvements

1. **Adaptive DPI**: Auto-adjust segmentation parameters based on image resolution
2. **Confidence Thresholding**: Filter low-confidence detections before returning
3. **Parallel Processing**: Process multiple pages concurrently
4. **Model Fine-tuning**: Train on customer-specific drawing styles

## Maintenance

### Updating from Upstream

If you need to pull updates from the original javvi51/edocr2:

```bash
cd /home/mainuser/Desktop/edocr2

# Add upstream remote (one-time)
git remote add upstream https://github.com/javvi51/edocr2.git

# Fetch upstream changes
git fetch upstream

# Merge upstream changes
git merge upstream/main

# Resolve conflicts if any
# Then push to your fork
git push origin gdtlens-integration
```

### Syncing with gdtlens

After updating the fork:

```bash
cd /home/mainuser/Desktop/gdtlens

# Copy updated files
cp -r /home/mainuser/Desktop/edocr2/* server/python/edocr2/

# Commit changes
git add server/python/edocr2
git commit -m "chore: update edocr2 from fork"
git push origin dev-full-drawing
```

## Version History

- **v1.0** (2026-03-10): Initial integration with GD&T Analyzer
  - Multi-parameter segmentation
  - IoU-based deduplication
  - Clean JSON output
  - 0-based page numbering
  - Recognizer caching

## Contact

For questions about these modifications, please open an issue at:
https://github.com/donrami/gdtlens/issues
