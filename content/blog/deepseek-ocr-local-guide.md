---
title: "DeepSeek OCR: Running State-of-the-Art OCR Locally"
date: 2026-02-01
description: "A complete guide to running DeepSeek-OCR offline on your own hardware using the local_ai_ocr project. Covers setup, configuration, quantization, and optimization."
tags: ["AI", "OCR", "DeepSeek", "Local AI", "Privacy", "Computer Vision", "Quantization"]
showTableOfContents: true
showReadingTime: true
showWordCount: false
---

## Introduction

In the age of AI, Optical Character Recognition (OCR) has evolved from simple pattern matching to sophisticated vision-language models that can understand context, preserve formatting, and handle complex documents. **DeepSeek-OCR** represents the cutting edge of this evolution — and the best part? You can run it entirely offline on your own hardware.

This guide covers:
- What makes DeepSeek-OCR special
- How to run it locally using the `local_ai_ocr` project
- Configuration, quantization, and optimization tips
- Real-world use cases

---

## What is DeepSeek-OCR?

DeepSeek-OCR is a vision-language model specifically trained for optical character recognition. Unlike traditional OCR engines (Tesseract, etc.), DeepSeek-OCR:

- **Understands context** — It doesn't just recognize characters; it understands document structure
- **Preserves formatting** — Tables, lists, and layouts are maintained
- **Multilingual** — Vietnamese, English, Chinese, Japanese, and more
- **Handles complexity** — Handwriting, degraded scans, mixed content

The model comes in a **3B parameter variant** (`deepseek-ocr:3b`), which is small enough to run on consumer hardware while maintaining excellent accuracy.

---

## Local AI OCR: The Easiest Way to Run DeepSeek-OCR

[**local_ai_ocr**](https://github.com/th1nhhdk/local_ai_ocr) is an open-source project that wraps DeepSeek-OCR in a user-friendly GUI. It's designed to be:

- **100% Offline** — After initial setup, no internet required
- **Portable** — Run from any folder, no installation needed
- **Privacy-first** — Your documents never leave your machine

### Key Features

| Feature | Description |
|---------|-------------|
| **GPU Acceleration** | Auto-detects NVIDIA GPUs for 5-10x speedup |
| **CPU Fallback** | Works without GPU (slower but functional) |
| **Multiple Formats** | PNG, JPG, WebP, HEIC, PDF support |
| **PDF Page Selection** | Process specific pages from large documents |
| **Queue System** | Batch process multiple files |
| **Formatted Output** | Copy results directly to Word with formatting preserved |
| **Live Visualization** | See bounding boxes as AI reads the document |

---

## System Requirements

### Recommended Specs

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| **OS** | Windows 10 | Windows 10/11 22H2+ |
| **CPU** | 4 cores / 8 threads | 8+ cores |
| **RAM** | 16 GB | 32 GB |
| **Storage** | 11 GB free | SSD preferred |
| **GPU** | None (CPU fallback) | NVIDIA with 8GB+ VRAM |

### GPU Notes

- **NVIDIA required** for GPU acceleration (uses CUDA)
- Driver version **531+** required
- The software will attempt GPU even with less VRAM — it may work with reduced performance

---

## Installation & Setup

### Step 1: Download

1. Go to [Releases](https://github.com/th1nhhdk/local_ai_ocr/releases)
2. Download the latest `.zip` file
3. Extract to any folder (e.g., `C:\Tools\local_ai_ocr`)

### Step 2: Initial Setup

```cmd
# Run the setup script
env_setup.cmd
```

This downloads the AI model weights (**~6.67 GB**). After this, the software works completely offline.

### Step 3: Launch

```cmd
# For GPU acceleration (recommended)
run.cmd

# For CPU-only mode
run_cpu-only.cmd

# With logging (for debugging)
run_wlog.cmd
```

---

## Configuration Deep Dive

The configuration lives in `config.toml`:

```toml
[engine]
ip_address = "http://127.0.0.1"
port = "11435"
model = "deepseek-ocr:3b"
```

### Configuration Options

| Key | Default | Description |
|-----|---------|-------------|
| `ip_address` | `http://127.0.0.1` | Local server address |
| `port` | `11435` | Port for the OCR engine |
| `model` | `deepseek-ocr:3b` | Model variant to use |

### Model Variants

Currently, the project uses **`deepseek-ocr:3b`** — a 3-billion parameter model that balances:
- **Accuracy**: Near state-of-the-art OCR performance
- **Speed**: Reasonable inference times on consumer hardware
- **Memory**: ~6-8GB VRAM for GPU, ~12-16GB RAM for CPU

---

## Quantization Explained

The model uses **quantization** to reduce memory footprint and increase inference speed. Here's what that means:

### What is Quantization?

Neural networks normally use 32-bit floating point numbers (FP32). Quantization reduces this to:
- **FP16** — 16-bit floating point (2x memory savings)
- **INT8** — 8-bit integers (4x memory savings)
- **INT4** — 4-bit integers (8x memory savings)

### DeepSeek-OCR Quantization

The `deepseek-ocr:3b` model appears to use **4-bit quantization** (based on the ~6.67GB download size for a 3B model):

```
Full FP32: 3B × 4 bytes = 12 GB
FP16:      3B × 2 bytes = 6 GB  
INT4:      3B × 0.5 bytes = 1.5 GB + overhead ≈ 6-7 GB
```

This aggressive quantization is why it can run on consumer GPUs with 8GB VRAM.

### Performance Impact

| Precision | Memory | Speed | Accuracy |
|-----------|--------|-------|----------|
| FP32 | 12+ GB | Baseline | 100% |
| FP16 | ~6 GB | 1.5-2x faster | ~99.9% |
| INT8 | ~3 GB | 2-3x faster | ~99.5% |
| INT4 | ~1.5 GB | 3-4x faster | ~98-99% |

The INT4 quantization used here provides excellent accuracy for OCR tasks while fitting in consumer hardware.

---

## Processing Modes

The software offers three OCR modes:

### 1. Markdown Document (Default — Best)

```
Best for: Structured documents, tables, forms
Preserves: Headers, tables, lists, formatting
Output: Markdown syntax
```

Use this for:
- Invoices and receipts
- Academic papers
- Forms and applications
- Any document with clear structure

### 2. Free OCR

```
Best for: Complex layouts, mixed content
Preserves: Better spatial layout than Standard
Output: Plain text with spacing
```

Use when default mode produces empty output (happens with very complex images).

### 3. Standard OCR

```
Best for: Simple text extraction
Preserves: Basic text only
Output: Plain text
```

Use for: Simple documents where formatting doesn't matter.

---

## Running on macOS / Linux

The official release targets Windows, but you can run it on other platforms:

### Using Docker

```bash
# Pull Ollama
docker run -d -v ollama:/root/.ollama -p 11435:11434 --gpus all ollama/ollama

# Pull the model
docker exec -it <container> ollama pull deepseek-ocr:3b
```

### Using Ollama Directly

```bash
# Install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Pull the model
ollama pull deepseek-ocr:3b

# Run server on custom port
OLLAMA_HOST=127.0.0.1:11435 ollama serve
```

Then use the Python components from the `src/` folder to interact with it.

---

## Performance Optimization Tips

### 1. GPU Memory Management

The software auto-unloads the model after 5 minutes of inactivity. For manual control:
- Click **"Unload AI Model"** when done processing
- This frees VRAM for other applications

### 2. Batch Processing

Use the queue system for multiple files:
1. Add all files first
2. Click "Start Processing" once
3. Let it batch process — avoids repeated model loading

### 3. VRAM Optimization

If you're hitting VRAM limits:
```cmd
# Force CPU mode for large batches
run_cpu-only.cmd
```

CPU is slower but can handle larger documents without VRAM constraints.

### 4. Driver Updates

For NVIDIA GPUs:
- Minimum: Driver 531+
- Recommended: Latest Game Ready or Studio driver
- Check: `nvidia-smi` in terminal

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| GPU not detected | Update NVIDIA driver to 531+ |
| Setup fails at [1/6] | Upgrade Windows to 22H2+ |
| Empty output | Try "Free OCR" mode instead of default |
| Infinite loop | Click STOP, try smaller image sections |
| Slow performance | Check GPU is being used, not CPU fallback |

---

## Use Cases

### Document Digitization
Scan old documents → OCR → Searchable archive

### Data Extraction
Invoices, receipts → OCR → Structured data for accounting

### Accessibility
Convert image-based PDFs → Screen-reader compatible text

### Research
Academic papers, books → OCR → Quotable, searchable text

### Privacy-Sensitive Documents
Medical records, legal docs → Local OCR → Zero cloud exposure

---

## Conclusion

DeepSeek-OCR via `local_ai_ocr` represents a paradigm shift: **state-of-the-art AI running entirely on your hardware**. No cloud APIs, no subscriptions, no privacy concerns.

The 3B quantized model hits the sweet spot of accuracy, speed, and accessibility. Whether you're digitizing a personal archive or processing sensitive business documents, this setup gives you full control.

**Links:**
- [GitHub Repository](https://github.com/th1nhhdk/local_ai_ocr)
- [Releases / Downloads](https://github.com/th1nhhdk/local_ai_ocr/releases)
- [DeepSeek AI](https://deepseek.com)
