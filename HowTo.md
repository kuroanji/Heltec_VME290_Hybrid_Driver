## Installation of Heltec VME290 Hybrid Driver for InkHUD (Legacy)

### Step 1: Copy Driver Files

Copy these files to your Meshtastic firmware:

```
HeltecVME290.h   ->  src/graphics/niche/Drivers/EInk/HeltecVME290.h
HeltecVME290.cpp ->  src/graphics/niche/Drivers/EInk/HeltecVME290.cpp
```

### Step 2: Modify nicheGraphics.h

Edit `variants/esp32s3/heltec_vision_master_e290/nicheGraphics.h`:

**Add include at top:**
```cpp
#include "graphics/niche/Drivers/EInk/HeltecVME290.h"
```

**Find this code (around line 96):**
```cpp
    // E-Ink Driver
    // -----------------------------

    Drivers::EInk *driver = new Drivers::DEPG0290BNS800;
    driver->begin(hspi, PIN_EINK_DC, PIN_EINK_CS, PIN_EINK_BUSY);
```

**Replace with:**
```cpp
    // E-Ink Driver
    // -----------------------------
    // Use HeltecVME290 instead of DEPG0290BNS800 to fix ghosting

    Drivers::EInk *driver = new Drivers::HeltecVME290;
    driver->begin(hspi, PIN_EINK_DC, PIN_EINK_CS, PIN_EINK_BUSY, PIN_EINK_RES);
```

Note: We add `PIN_EINK_RES` (reset pin) for deep sleep support.

### Step 3: Build

```bash
pio run -e heltec-vision-master-e290-inkhud
```

---
