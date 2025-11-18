# Remote OTA Usermod

This usermod checks a remote server for firmware updates and notifies users in the web interface when updates are available, allowing one-click updates.

## Features

- **Periodic update checks** from a configured version check URL
- **Web UI notification** when updates are available
- **One-click update** button in the info section
- **Progress tracking** during updates
- **Configurable check interval** (default: 1 hour)
- **JSON API integration** for status and manual control
- **Works on both ESP32 and ESP8266**

## Installation

Add `usermod_v2_remote_ota` to your `platformio_override.ini`:

```ini
[env:my_custom_build]
extends = env:esp32dev
build_flags = 
  ${env:esp32dev.build_flags}
  -D USERMOD_ID_REMOTE_OTA=25  ; Assign a unique usermod ID
lib_deps = 
  ${env:esp32dev.lib_deps}
```

## Configuration

Configure via the WLED web interface under Settings > Usermods > RemoteOTA:

- **Enabled**: Enable/disable the usermod
- **Version Check URL**: URL that returns JSON with version info (e.g., `http://example.com/version.json`)
- **Firmware URL**: Direct URL to firmware binary (can be left empty if version.json provides it)
- **Check Interval Hours**: How often to check for updates (in hours, default: 1)

## Usage

### Web Interface

1. The usermod automatically checks for updates at the configured interval
2. When an update is available, the info section shows: **"Update available: [version]"** with a **"🔄 Update Now"** button
3. Click the button to start the update
4. Progress is displayed during the update
5. Device will reboot automatically when complete

### Manual Check via JSON API

Force an immediate update check:

```bash
curl -X POST http://<wled-ip>/json/state -d '{"rmOTA_check":true}'
```

Trigger update (if one is available):

```bash
curl -X POST http://<wled-ip>/json/state -d '{"rmOTA_update":true}'
```

Check status:

```bash
curl http://<wled-ip>/json/state
# Look for: rmOTA_available, rmOTA_version, rmOTA_current, rmOTA_updating, rmOTA_progress
```

## Version Check Server Requirements

The version check URL must return JSON in this format:

```json
{
  "version": "0.16.0",
  "url": "http://example.com/firmware/WLED_0.16.0_ESP32_S3_Icon256.bin"
}
```

- **version**: String representing the firmware version (compared against current WLED_VERSION)
- **url**: Direct download URL for the firmware binary

## Example Server Setup

### Static JSON File (Simple)

Create `version.json` on your web server:

```json
{
  "version": "0.16.0",
  "url": "https://myserver.com/firmware/latest.bin"
}
```

### Dynamic PHP Example

```php
<?php
header('Content-Type: application/json');
echo json_encode([
    'version' => '0.16.0',
    'url' => 'https://myserver.com/firmware/WLED_0.16.0_ESP32_S3.bin'
]);
?>
```

### Nginx Configuration

```nginx
server {
    listen 80;
    server_name firmware.example.com;
    
    location /version.json {
        root /var/www/firmware;
        add_header Access-Control-Allow-Origin *;
    }
    
    location /firmware/ {
        root /var/www;
    }
}
```

## Troubleshooting

- Check serial debug output for update status messages
- Ensure the update URL is accessible from the WLED device
- Verify sufficient free flash space for the update
- For ESP8266, OTA updates require at least 50% free flash space

## Change Log

2025-11-16
* Initial release
* Support for HTTP/HTTPS updates
* Configurable automatic update checks
* JSON API integration
