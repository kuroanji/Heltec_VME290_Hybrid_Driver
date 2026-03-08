# GxEPD2 E290 Ghosting Fix (for non INKHUD firmware)

## Problem

The standard Meshtastic firmware for Heltec Vision Master E290 exhibits ghosting artifacts during fast (partial) refresh. This happens because the display controller's differential refresh requires both NEW and OLD memory buffers to be in sync.

## Root Cause

In the original `GxEPD2_290_BN8` driver, `writeImageAgain()` only writes to OLD memory (0x26). However, for proper differential refresh on this display, both buffers need to be updated after each fast refresh — similar to how InkHUD's `finalizeUpdate()` works.

## Fix

The fix modifies `writeImageAgain()` to write the image to **both** NEW (0x24) and OLD (0x26) memory buffers instead of just OLD.

### Changed code in `GxEPD2_290_BN8.cpp`:

```cpp
// Write image to both NEW and OLD memory after fast refresh
// This ensures differential refresh works correctly by keeping both buffers in sync
void GxEPD2_290_BN8::writeImageAgain(const uint8_t bitmap[], int16_t x, int16_t y, int16_t w, int16_t h, bool invert,
                                     bool mirror_y, bool pgm)
{
    // Write to NEW memory first (required for proper differential refresh)
    _writeImage(0x24, bitmap, x, y, w, h, invert, mirror_y, pgm);

    // Then write to OLD memory
    _writeImage(0x26, bitmap, x, y, w, h, invert, mirror_y, pgm);

    // Terminate image write without triggering update
    _writeCommand(0x7F);
    _waitWhileBusy("writeImageAgain", 200);
}
```

## How to Apply

### Option 1: Replace files manually

Copy the fixed driver files to your GxEPD2 library:

```bash
cp GxEPD2_290_BN8.cpp .pio/libdeps/heltec-vision-master-e290/GxEPD2/src/epd/
cp GxEPD2_290_BN8.h .pio/libdeps/heltec-vision-master-e290/GxEPD2/src/epd/
```

**Note**: Files in `.pio/libdeps/` are regenerated on clean build. You'll need to reapply after `pio run --target clean`.

### Option 2: Fork meshtastic/GxEPD2

1. Fork https://github.com/meshtastic/GxEPD2
2. Apply the fix to `src/epd/GxEPD2_290_BN8.cpp`
3. Update `lib_deps` in `variants/esp32s3/heltec_vision_master_e290/platformio.ini` to point to your fork

### Option 3: Use extra_scripts (recommended for permanent fix)

Add a PlatformIO script that patches the file after library installation.

## Affected Builds

- `heltec-vision-master-e290` (standard Meshtastic UI)

## Not Affected

- `heltec-vision-master-e290-inkhud` (uses separate driver)
- `heltec-vision-master-e290-inkhud2` (uses separate driver)

## Technical Details

The SSD1680 controller uses two memory buffers for differential refresh:
- **NEW memory (0x24)**: The target image
- **OLD memory (0x26)**: The previous image (used to determine which pixels changed)

During differential refresh, the controller compares these buffers to decide which pixels need to transition (B→W, W→B, B→B, W→W) and applies the appropriate waveform from the LUT.

If OLD memory doesn't match what's actually on screen, the controller makes incorrect assumptions about pixel states, causing ghosting.
