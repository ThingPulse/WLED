# Icon256 Word Clock Button Configuration

This usermod configures three hardware buttons for the Icon256 Word Clock device.

## Hardware Setup

Three buttons are connected to the following pins with internal pullup resistors:
- **Button 1** (Pin 11): Next Palette
- **Button 2** (Pin 12): Previous Palette  
- **Button 3** (Pin 13): Toggle Display On/Off

## Button Functions

### Button 1: Next Palette (Pin 11)
Cycles to the next color palette in the palette list.

### Button 2: Previous Palette (Pin 12)
Cycles to the previous color palette in the palette list.

### Button 3: Toggle Display (Pin 13)
Toggles the LED display on and off.

## Preset Configuration

The buttons are configured to trigger presets 251-253. These presets need to be created via the WLED web UI or API.

### Creating the Presets

You can create these presets in two ways:

#### Method 1: Via Web UI
1. Go to the WLED web interface
2. Navigate to Settings → Presets
3. Create three new presets with IDs 251, 252, and 253

#### Method 2: Via API
Send POST requests to `/json/state` with the following JSON for each preset:

**Preset 251 (Next Palette):**
```bash
curl -X POST http://[device-ip]/json/state \
  -H "Content-Type: application/json" \
  -d '{"psave":251,"seg":{"pal":"r+"},"n":"Next Palette"}'
```

**Preset 252 (Previous Palette):**
```bash
curl -X POST http://[device-ip]/json/state \
  -H "Content-Type: application/json" \
  -d '{"psave":252,"seg":{"pal":"r-"},"n":"Previous Palette"}'
```

**Preset 253 (Toggle Display):**
```bash
curl -X POST http://[device-ip]/json/state \
  -H "Content-Type: application/json" \
  -d '{"psave":253,"on":"t","n":"Toggle Display"}'
```

### Preset Details

- **Preset 251**: `{"seg":{"pal":"r+"}}` - Cycles to next palette using relative increment
- **Preset 252**: `{"seg":{"pal":"r-"}}` - Cycles to previous palette using relative decrement
- **Preset 253**: `{"on":"t"}` - Toggles display on/off state

## Technical Details

The button configuration is set in the `setup()` method of the usermod:

```cpp
// Configure buttons on pins 11, 12, 13 with pullup
btnPin[0] = 11;     // Button 1: Next palette
btnPin[1] = 12;     // Button 2: Previous palette
btnPin[2] = 13;     // Button 3: Toggle display

buttonType[0] = BTN_TYPE_PUSH;
buttonType[1] = BTN_TYPE_PUSH;
buttonType[2] = BTN_TYPE_PUSH;

// Assign presets to button macros
macroButton[0] = 251;  // Button 1 -> Preset 251 (next palette)
macroButton[1] = 252;  // Button 2 -> Preset 252 (previous palette)
macroButton[2] = 253;  // Button 3 -> Preset 253 (toggle display)
```

## Wiring Diagram

```
ESP32-S3 Pin 11 ----[Button 1]---- GND
ESP32-S3 Pin 12 ----[Button 2]---- GND
ESP32-S3 Pin 13 ----[Button 3]---- GND
```

The internal pullup resistors are enabled, so external resistors are not required.

## Troubleshooting

### Buttons not responding
- Verify presets 251-253 are created correctly
- Check button wiring and connections
- Test presets manually via the web UI to ensure they work
- Check serial output for button press detection

### Wrong action triggered
- Verify preset IDs match (251, 252, 253)
- Check preset JSON content matches the examples above
- Clear and recreate the presets if needed
