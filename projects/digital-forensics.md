---
layout: default
title: "Digital Forensics & Metadata Analysis"
permalink: /projects/digital-forensics/
---

# Digital Forensics: Metadata Extraction & Encrypted Volume Analysis

In the realm of Digital Forensics and Incident Response (DFIR), preserving the chain of custody and extracting hidden file attributes are critical during an investigation. This project showcases the development of automated Python tooling for metadata extraction and the application of low-level disk analysis to identify encrypted volumes.

## 🕵️‍♂️ Project Scope & Tooling

* **Objective:** Automate the extraction of hidden metadata (EXIF, PDF parameters), generate cryptographic hashes for evidence integrity, and identify BitLocker-encrypted partitions using raw hex analysis.
* **Forensic Tools:** FTK Imager, Autopsy, HxD (Hex Editor).
* **Development Stack:** Python (`PyPDF2`, `Pillow`, `hashlib`, `os`).

## ⚙️ Automated Forensic Toolkit (Python)

To streamline the investigation process, I developed a Python script capable of recursively scanning directories, extracting metadata from various file formats (Images, PDFs, Word documents), and calculating SHA-256 hashes to ensure data integrity for court-admissible evidence.

```python
import os
import hashlib
from PIL import Image
from PIL.ExifTags import TAGS
from PyPDF2 import PdfReader

def calculate_sha256(filepath):
    """Generates a SHA-256 hash to ensure forensic integrity of the file."""
    sha256_hash = hashlib.sha256()
    with open(filepath, "rb") as f:
        for byte_block in iter(lambda: f.read(4096), b""):
            sha256_hash.update(byte_block)
    return sha256_hash.hexdigest()

def extract_image_metadata(image_path):
    """Extracts hidden EXIF data (GPS, Camera Model, Timestamps) from images."""
    try:
        image = Image.open(image_path)
        exif_data = image.getexif()
        metadata = {}
        if exif_data:
            for tag_id, value in exif_data.items():
                tag_name = TAGS.get(tag_id, tag_id)
                metadata[tag_name] = value
        return metadata
    except Exception as e:
        return f"Error reading image: {e}"

def extract_pdf_metadata(pdf_path):
    """Extracts author, creation date, and software used from PDF files."""
    try:
        reader = PdfReader(pdf_path)
        meta = reader.metadata
        return {k.strip('/'): v for k, v in meta.items()} if meta else {}
    except Exception as e:
        return f"Error reading PDF: {e}"

# Example Usage in an investigation pipeline:
# file_hash = calculate_sha256("evidence/suspect_photo.jpg")
# print(f"[*] Evidence Integrity Hash (SHA-256): {file_hash}")
```

## 🔐 Low-Level Drive Encryption Analysis

Identifying whether a seized drive is encrypted (and what type of encryption is used) is the first step in digital forensics. Standard OS tools often fail to provide this context without mounting the drive, which risks altering the evidence.

**The Process:**
1. **Imaging:** Created a bit-by-bit forensic image (`.dd` / `.E01`) of the suspect USB drive using **FTK Imager** to preserve the original state.
2. **Hex Analysis:** Instead of relying on Windows Explorer, I analyzed the raw binary structure of the disk image using **HxD**.
3. **Signature Detection:** By examining the boot sector and volume headers at the hexadecimal level, I successfully identified the ASCII signature `-FVE-FS-` (Full Volume Encryption File System), definitively confirming the presence of Microsoft **BitLocker** encryption.

## 📉 Conclusions & Takeaways

* **Integrity is Paramount:** The use of cryptographic hashing (`MD5`/`SHA-256`) immediately upon evidence acquisition is non-negotiable to maintain the chain of custody.
* **Automation Speeds Triage:** Manual metadata extraction is unscalable during a major incident. Custom Python scripts drastically reduce the time needed to triage hundreds of files.
* **Low-Level Knowledge:** Relying solely on GUI tools is a vulnerability for an investigator. Understanding raw file systems and hexadecimal signatures is crucial for bypassing standard OS limitations and identifying encrypted containers.

<div class="back-link-container">
  <a href="/" class="btn-back">← Back to Home</a>
</div>