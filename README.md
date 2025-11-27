# Image Upload API Documentation

## Overview

The Image Upload API allows you to upload image files to a server with dynamic directory structure based on a path string parameter. The API accepts image files via POST request and organizes them in a hierarchical directory structure.

## Endpoint

```
POST /upload.php
```

## Request Format

The API accepts `multipart/form-data` requests with the following fields:

### Required Parameters

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `image` | File | The image file to upload | `photo.jpg` |
| `path` | String | Directory path structure (alphanumeric, slashes, hyphens, underscores only) | `9897/profile` |

### Request Headers

```
Content-Type: multipart/form-data
```

## Supported Image Formats

- JPEG (`.jpg`, `.jpeg`)
- PNG (`.png`)
- GIF (`.gif`)
- WebP (`.webp`)

## File Size Limits

- Maximum file size: **5MB** (5,242,880 bytes)

## Directory Structure

The uploaded image will be saved to:
```
uploads/{path_string}/{unique_filename}
```

Where:
- `{path_string}` is the sanitized path parameter (e.g., `9897/profile`)
- `{unique_filename}` is generated as: `{timestamp}_{random_hex}.{extension}`

### Example

If you send:
- Path: `9897/profile`
- File: `myphoto.jpg`

The file will be saved to:
```
uploads/9897/profile/1699123456_a1b2c3d4e5f6g7h8.jpg
```

## Response Format

All responses are returned in JSON format.

### Success Response

**HTTP Status Code:** `200 OK`

```json
{
    "success": true,
    "message": "Image uploaded successfully.",
    "file_path": "uploads/9897/profile/1699123456_a1b2c3d4e5f6g7h8.jpg",
    "filename": "1699123456_a1b2c3d4e5f6g7h8.jpg"
}
```

### Error Responses

**HTTP Status Code:** `400 Bad Request` or `405 Method Not Allowed` or `500 Internal Server Error`

```json
{
    "success": false,
    "error": "Error message describing what went wrong"
}
```

## Error Codes and Messages

| HTTP Code | Error Message | Description |
|-----------|---------------|-------------|
| 405 | Method not allowed. Only POST requests are accepted. | Request method is not POST |
| 400 | Missing required fields. Both image and path are required. | Missing `image` or `path` parameter |
| 400 | File exceeds upload_max_filesize directive in php.ini | File size exceeds PHP configuration limit |
| 400 | File exceeds MAX_FILE_SIZE directive in HTML form | File size exceeds form limit |
| 400 | File was only partially uploaded | Upload was interrupted |
| 400 | No file was uploaded | No file was provided |
| 400 | File size exceeds maximum allowed size of 5MB. | File exceeds 5MB limit |
| 400 | Invalid file type. Only image files (JPEG, PNG, GIF, WebP) are allowed. | File is not a valid image type |
| 400 | Invalid path string. Path must contain only alphanumeric characters, slashes, hyphens, and underscores. | Path contains invalid characters |
| 400 | File extension does not match file type. | File extension doesn't match MIME type |
| 500 | Failed to create directory structure. | Could not create target directory |
| 500 | Failed to move uploaded file to destination. | File could not be moved to target location |

## Usage Examples

### HTML Form Example

```html
<!DOCTYPE html>
<html>
<head>
    <title>Image Upload</title>
</head>
<body>
    <form action="upload.php" method="POST" enctype="multipart/form-data">
        <div>
            <label for="image">Select Image:</label>
            <input type="file" name="image" id="image" accept="image/*" required>
        </div>
        <div>
            <label for="path">Path:</label>
            <input type="text" name="path" id="path" placeholder="9897/profile" required>
        </div>
        <button type="submit">Upload Image</button>
    </form>
</body>
</html>
```

### JavaScript (Fetch API) Example

```javascript
async function uploadImage(imageFile, pathString) {
    const formData = new FormData();
    formData.append('image', imageFile);
    formData.append('path', pathString);

    try {
        const response = await fetch('upload.php', {
            method: 'POST',
            body: formData
        });

        const result = await response.json();

        if (result.success) {
            console.log('Upload successful!');
            console.log('File path:', result.file_path);
            console.log('Filename:', result.filename);
            return result;
        } else {
            console.error('Upload failed:', result.error);
            return result;
        }
    } catch (error) {
        console.error('Network error:', error);
        return { success: false, error: error.message };
    }
}

// Usage
const fileInput = document.querySelector('input[type="file"]');
const pathInput = document.querySelector('input[name="path"]');

fileInput.addEventListener('change', async (e) => {
    const file = e.target.files[0];
    const path = pathInput.value;
    
    if (file && path) {
        const result = await uploadImage(file, path);
        // Handle result
    }
});
```

### cURL Example

```bash
curl -X POST \
  http://your-domain.com/upload.php \
  -F "image=@/path/to/your/image.jpg" \
  -F "path=9897/profile"
```

### PHP Example

```php
<?php
function uploadImage($imagePath, $pathString) {
    $url = 'admin.sallaspare/upload.php';
    
    $postData = [
        'image' => new CURLFile($imagePath),
        'path' => $pathString
    ];
    
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_POST, true);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $postData);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    
    $response = curl_exec($ch);
    $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
    curl_close($ch);
    
    return json_decode($response, true);
}

// Usage
$result = uploadImage('/path/to/image.jpg', '9897/profile');
if ($result['success']) {
    echo "Uploaded to: " . $result['file_path'];
} else {
    echo "Error: " . $result['error'];
}
?>
```

## Path String Rules

The `path` parameter must follow these rules:

- **Allowed characters:** Letters (a-z, A-Z), numbers (0-9), forward slashes (`/`), hyphens (`-`), and underscores (`_`)
- **Invalid characters:** Any special characters, backslashes, or directory traversal patterns (`..`) will be removed
- **Examples:**
  - Valid: `9897/profile`
  - Valid: `user-123/images/avatar`
  - Valid: `2024/01/15`
  - Invalid: `../../etc/passwd` (will be sanitized)
  - Invalid: `user@123` (special characters removed)

## Security Features

1. **Path Sanitization:** Prevents directory traversal attacks by removing dangerous patterns
2. **MIME Type Validation:** Verifies that uploaded files are actually images
3. **File Extension Validation:** Ensures file extension matches the actual file type
4. **Unique Filenames:** Prevents file overwrites and conflicts
5. **File Size Limits:** Prevents abuse and server overload
6. **Secure File Handling:** Uses PHP's `move_uploaded_file()` function

## Server Requirements

- PHP 7.0 or higher
- `upload_tmp_dir` must be writable
- Target `uploads/` directory must be writable (or parent directory must be writable for directory creation)
- `mime_content_type()` function must be available

## PHP Configuration Recommendations

Ensure your `php.ini` has appropriate settings:

```ini
upload_max_filesize = 5M
post_max_size = 6M
max_file_uploads = 20
file_uploads = On
```

## Notes

- The API automatically creates directory structures if they don't exist
- Filenames are automatically generated to prevent conflicts
- Original file extensions are preserved
- The `uploads/` directory will be created relative to the script location
- All responses are in JSON format regardless of success or failure

## Troubleshooting

### Common Issues

1. **"Failed to create directory structure"**
   - Check that the parent directory (`uploads/`) has write permissions
   - Ensure PHP has permission to create directories

2. **"File size exceeds maximum allowed size"**
   - Reduce file size or increase the limit in the code (line 56)

3. **"Invalid file type"**
   - Ensure the file is a valid image format (JPEG, PNG, GIF, WebP)
   - Check that the file isn't corrupted

4. **"Method not allowed"**
   - Ensure you're using POST method
   - Check that the request includes `Content-Type: multipart/form-data`

## Version

- **API Version:** 1.0
- **Last Updated:** 2024

