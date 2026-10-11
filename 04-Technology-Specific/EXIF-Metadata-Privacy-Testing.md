# 📁 EXIF Metadata & Geolocation Privacy — Security Testing Guide

*Contributed by Cyber_Spy.404 | Educational Bug Bounty Series*

> **Scope and ethics:** Testing image metadata involves evaluating whether user privacy is protected on public media feeds. Test strictly on authorized assets. **Never extract or disclose the real-world GPS coordinates or personal identity data of third-party platform users.** Test exclusively by uploading your own test image populated with synthetic, non-sensitive GPS coordinates.

---

## 📌 Executive Summary

**Exchangeable Image File Format (EXIF)** is a standard specification that formats and embeds metadata into image files (JPEG, TIFF, HEIC, WebP). When modern smartphones and digital cameras capture a photo, they automatically record extensive technical and contextual metadata inside the file.

If a social media platform, dating app, marketplace, or community forum hosts user-uploaded images without automatically stripping EXIF tags, sensitive personal information—most critically **precise GPS coordinates**—is leaked to anyone viewing or downloading the image.

```
+---------------------------------------------------------------------------------+
|                        EXIF Metadata Privacy Risks                              |
+---------------------------------------------------------------------------------+
        |                                |                                 |
        v                                v                                 v
  GPS Geolocation Coordinates     Device Hardware Info             Stored XSS in Metadata
  - Exact Latitude & Longitude   - Camera Make & Model             - <script> in Artist tag
  - Altitude & Direction         - Software / OS build             - Executed in admin portal
```

---

## 🔍 Sensitive Information Embedded in EXIF

When an application fails to sanitize uploads, the following metadata fields are exposed:

| EXIF Tag | Data Disclosed | Privacy / Security Impact |
| :--- | :--- | :--- |
| **`GPSLatitude` / `GPSLongitude`** | Exact decimal coordinates (e.g., `37.7749° N, 122.4194° W`) | **Critical Privacy Violation:** Exposes exact home, workplace, or physical location of users. |
| **`GPSAltitude` / `GPSImgDirection`**| Elevation and compass heading | Pinpoints exact building floor and viewpoint. |
| **`DateTimeOriginal`** | Exact timestamp and timezone | Discloses user schedules and movement timelines. |
| **`Make` & `Model`** | Device hardware (e.g. `Apple iPhone 15 Pro`) | Device fingerprinting and OS profiling. |
| **`Artist` / `CameraOwnerName`** | Registered user name or desktop profile | Discloses real names or employee usernames. |
| **`Software`** | Editing tools (e.g. `Adobe Photoshop 24.1`) | Version enumeration for desktop software. |

---

## 🔬 Core Testing Methodology

### Step 1: Create a Synthetic Test Image with Embedded EXIF
Never use real personal photos for testing. Create a clean image and inject synthetic test metadata using `exiftool`:

```bash
# Inject synthetic GPS coordinates and comment into a test JPEG
exiftool -GPSLatitude=37.774929 -GPSLatitudeRef=N \
         -GPSLongitude=-122.419416 -GPSLongitudeRef=W \
         -Artist="CyberSpy Test Account" \
         -Comment="Safe Metadata Testing Probe" \
         synthetic_test.jpg
```

---

### Step 2: Upload Image to Target Platform
Upload `synthetic_test.jpg` through the user profile avatar, public forum post, listing gallery, or comment attachment.

---

### Step 3: Download and Inspect Publicly Rendered Image
1. Log out or open a separate incognito browser window.
2. Navigate to the public URL where the image is displayed.
3. Right-click and save the image: `downloaded_avatar.jpg`.
4. Inspect the downloaded image with `exiftool`:
   ```bash
   exiftool downloaded_avatar.jpg | grep -i -E "GPS|Artist|Comment"
   ```

#### Evaluation:
- **Vulnerable Indicator:** Output displays the original `GPSLatitude`, `GPSLongitude`, or `Artist` tags. The application stored and served the original raw image without re-encoding or stripping tags!
- **Secure Indicator:** Output returns no GPS data, or displays metadata indicating the image was re-encoded (e.g., processed with ImageMagick or libvips).

---

## 💡 High-Value Bug Bounty Tips & Edge Cases

### 1. Avatar vs Direct Attachment Inconsistencies
Applications often re-encode profile pictures (stripping EXIF), but leave raw files untouched on secondary upload features:
- Private messaging attachments.
- Support ticket attachments.
- Marketplace listing photo galleries.
- Cover photos / banners.

### 2. High-Resolution "View Original" Links
Even if thumbnails or compressed web versions are stripped, check if clicking "View Full Resolution" or "Download Original" serves the unstripped raw file.

### 3. Stored XSS via EXIF Headers
Certain administrative backends and digital asset management (DAM) platforms parse EXIF metadata and display image details in an admin dashboard:
```bash
exiftool -Artist='<script>alert(document.domain)</script>' test.jpg
exiftool -ImageDescription='<img src=x onerror=alert(1)>' test.jpg
```
If the backend dashboard renders the `Artist` or `Description` tag without HTML encoding, stored XSS triggers when the administrator views the media library.

---

## 🛠️ Safe Educational Verification Template

### Verification Command & Output
```bash
$ exiftool downloaded_image.jpg

ExifTool Version Number         : 12.50
File Name                       : downloaded_image.jpg
File Size                       : 24 kB
Camera Model Name               : Synthetic Test Camera
Artist                          : CyberSpy Test Account
GPS Latitude                    : 37 deg 46' 29.74" N
GPS Longitude                   : 122 deg 25' 9.90" W
GPS Position                    : 37 deg 46' 29.74" N, 122 deg 25' 9.90" W
```

---

## 🛡️ Mitigation & Remediation Guidelines

### 1. Automated Metadata Stripping with ImageMagick
Run the `-strip` command during post-processing:
```bash
# Strips all profiles, EXIF, and comments
convert input.jpg -strip output.jpg
```

### 2. Python (Pillow) Re-encoding
When processing images in Python, saving without the `exif` parameter automatically drops all metadata:
```python
from PIL import Image

def process_upload(input_path, output_path):
    # Open image
    image = Image.open(input_path)
    
    # Create clean image data
    clean_image = Image.new(image.mode, image.size)
    clean_image.putdata(list(image.getdata()))
    
    # Save without EXIF
    clean_image.save(output_path, "JPEG", quality=85)
```

### 3. Node.js (Sharp / libvips)
```javascript
const sharp = require('sharp');

// Sharp strips all EXIF metadata by default unless withMetadata() is called
await sharp(inputBuffer)
  .resize(800, 800, { fit: 'inside' })
  .jpeg({ quality: 80 })
  .toFile(outputPath);
```

---

## 📊 CVSS 3.1 Severity Reference

- **GPS Location Disclosure on Dating / Social / Children's Apps:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N` (Score: **7.5 - High**)
- **GPS Location Disclosure on General Web Forum:** `CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:N/A:N` (Score: **5.3 - Medium**)
- **Stored XSS via EXIF Metadata in Admin Console:** `CVSS:3.1/AV:N/AC:L/PR:L/UI:R/S:C/C:H/I:H/A:N` (Score: **9.0 - Critical**)

---

## ✅ Bug Bounty Triage Checklist

- [ ] Used strictly a synthetic test image with non-sensitive coordinates.
- [ ] Confirmed that the downloaded image from the public endpoint retains GPS tags.
- [ ] Checked multiple upload contexts (avatars, galleries, attachments).
- [ ] Included exact `exiftool` terminal output in the disclosure report.
- [ ] Recommended automated stripping via ImageMagick `-strip` or Sharp.
---

## 🧪 Hands-On Practice Labs & Real-World Challenges

Practice analyzing metadata retention and image parsing vulnerabilities:

| Platform | Lab / Challenge Name | Difficulty | Key Focus Area |
| :--- | :--- | :--- | :--- |
| **TryHackMe** | [Metadata Extraction Room](https://tryhackme.com/room/metadataextraction) | Easy | Using `exiftool` to locate embedded GPS latitude/longitude data |
| **Hack The Box** | [HTB Machine: Pilgrimage](https://app.hackthebox.com/machines/Pilgrimage) | Easy | ImageMagick CVE-2022-44268 extracting sensitive files via PNG profile |
| **OWASP** | [Juice Shop: GPS EXIF Privacy Challenge](https://owasp.org/www-project-juice-shop/) | Easy | Extracting confidential company location from uploaded media |
