# VibeCube - AI Coding Instructions

## Project Overview
**First-time hardware project** with tight deadline (Feb 15). Two physical cubes with NFC readers that enable synchronized music playback between friends. When both DJs insert the same NFC card (representing a SoundCloud track), the track plays simultaneously on both cubes.

**Critical Constraints:**
- Builder has no prior hardware experience - all guidance must be beginner-friendly
- 18-day timeline including component shipping and assembly
- Prioritize working prototype over perfect implementation

## Architecture
- **Cube Client** (`/cube/`): Python-based local software running on Raspberry Pi Zero 2 W
  - Hardware: PN532 NFC reader, 0.96" OLED display, Bluetooth for audio output
  - Connects to external Bluetooth speaker (DJs likely already own)
  - Real-time WebSocket connection to sync server
  - Handles NFC detection, audio playback via Bluetooth, display updates
- **Sync Server** (`/server/`): Coordination service using Python + FastAPI + WebSockets
  - Manages cube pairing, synch (NFC), pygame (audio), Luma.OLED (display), websockets
- **Server:** Python FastAPI + WebSockets (keep language consistent for beginners)
- **Hardware:** Raspberry Pi Zero 2 W, PN532 NFC module, SSD1306 OLED, USB audio
- **Development:** VS Code with Remote SSH extension for Pi development
Component Ordering (URGENT - Order Immediately)

**Amazon Prime** (2-day shipping, order today Jan 28):
- Raspberry Pi Zero 2 W (x2): $15 each - CRITICAL, check stock, needs built-in Bluetooth
- PN532 NFC Module (x2): ~$8 each - HiLetgo or Elechouse brand
- 0.96" OLED Display SSD1306 (x2): ~$5 each (I2C version, 4-pin)
- MicroSD Card 16GB (x2): ~$6 each - SanDisk or Samsung
- USB-C Power Supply 5V 2.5A (x2): ~$8 each
- NTAG215 NFC Cards (1 pack of 10): ~$8 - Tagmo compatible
- Female-Female Jumper Wires: ~$6 (40-piece set)

**Alternate if Pi Zero 2 unavailable:** Raspberry Pi 4 Model B 2GB (~$45)

**Local electronics store** (same day):
- Breadboard (x2) - for prototyping before soldering
- Header pins for Pi Zero (if not pre-soldered)

**User provides** (DJs likely already own):
- Bluetooth speaker (x2) - any portable Bluetooth speaker works
- Must support standard A2DP Bluetooth audio profile
- Recommend: JBL, UE Boom, Anker Soundcore, or similar

**3D Printing** (order print service if no printer):
- Shapeways or local makerspace
- Upload STL files for cube enclosure
- Material: Black PLA
- Cost: ~$30-50 per cube
- **Lead time: 7-10 days** - order by Jan 30 at latest

## Key Development Workflows

**Phase 1: Test on desktop FIRST (Jan 29-31)** - develop/test without Pi:
```bash
# Create virtual environment
python3 -m venv venv
source venv/bin/activate
pip install websockets pygame pillow

# Test with mocks
python cube/test_display.py --mock     # Terminal output instead of OLED
python cube/test_audio.py              # Uses computer speakers
python cube/main.py --mock-nfc --mock-display  # Full simulation
```

**Phase 2: Test each component on Pi (Feb 1-4)**:
```bash
# On Raspberry Pi via SSH
cd ~/vibecube

# Test 1: OLED Display
python cube/test_display.py
# Expected: "VibeCube Ready" displays on OLED

# Test 2: NFC Reader
python cube/test_nfc.py
# Expected: "Waiting for card..." then detects when card placed

# Test 3: Bluetooth Audio
python cube/test_bluetooth.py
# Expected: Connects to paired Bluetooth speaker, plays test tone

# Test 4: All together
python cube/main.py --cube-id cube1
```

**Phase 3: Run server (Feb 5-8)**:
```bash
# On development Mac (not on Pi)
cd server
python3 -m venv venv
source venv/bin/activate
pip install fastapi uvicorn websockets
python app.py
# Server runs on http://localhost:8000
```

**Phase 4: Full integration test (Feb 9-11)**:
```bash
# Terminal 1 (Mac): Start server
cd server && python app.py

# Terminal 2: SSH to cube1
ssh pi@cube1.local
cd ~/vibecube
python main.py --cube-id cube1 --server-url ws://YOUR_MAC_IP:8000

# Terminal 3: SSH to cube2  
ssh pi@cube2.local
cd ~/vibecube
python main.py --cube-id cube2 --server-url ws://YOUR_MAC_IP:8000

# Test: Insert same NFC card in both cubes
```

**Deploy server to cloud** (optional, for final version):
```bash
# Use free tier: Railway.app or Render.com
# Push to GitHub, connect to Railway, auto-deploy
   **Step 1 - OLED Display (I2C):**
   ```
   OLED VCC  → Pi Pin 1 (3.3V)
   OLED GND  → Pi Pin 6 (Ground)
   OLED SDA  → Pi Pin 3 (GPIO 2 / SDA)
   OLED SCL  → Pi Pin 5 (GPIO 3 / SCL)
   
   # Test immediately:
   i2cdetect -y 1  # Should show device at 0x3C
   ```

   **Step 2 - NFC Reader (SPI):**
   ```

## Timeline Milestones (Feb 15 Deadline)

**Week 1 (Jan 28 - Feb 3): Order & Basic Setup**
- ✅ Jan 28: Order all components (Amazon Prime)
- ✅ Jan 29-30: Develop software with mocks on desktop
- ✅ Jan 30: Components arrive, flash Raspberry Pi OS
- ✅ Jan 31: First boot, SSH access, install dependencies
- ✅ Feb 1-2: Wire and test OLED display
- ✅ Feb 3: Wire and test NFC reader

**Week 2 (Feb 4-10): Integration & Testing**
- ✅ Feb 4: Wire and test USB audio + speaker
- ✅ Feb 5-6: Develop server, test WebSocket communication
- ✅ Feb 7-8: Test full sync between 2 Pis
- ✅ Feb 9: Assemble second cube, repeat all tests
- ✅ Feb 10: Write NFC cards with track data

**Week 3 (Feb 11-15): Assembly & Polish**
- ✅ Feb 11: 3D printed enclosures arrive (ordered by Jan 30!)
- ✅ Feb 12: Assemble electronics into enclosures
- ✅ Feb 13: Final testing, bug fixes
- ✅ Feb 14: Print/design NFC card artwork
- 🎁 Feb 15: **Gift ready!**

**Contingency:** If 3D prints delayed, use temporary enclosure (cardboard box with holes)
   PN532 VCC  → Pi Pin 17 (3.3V)
   PN532 GND  → Pi Pin 20 (Ground)
   PN532 MOSI → Pi Pin 19 (GPIO 10 / MOSI)
   PN532 MISO → Pi Pin 21 (GPIO 9 / MISO)
   PN532 SCK  → Pi Pin 23 (GPIO 11 / SCLK)
   PN532 SS   → Pi Pin 24 (GPIO 8 / CE0)
   
   # Set PN532 to SPI mode (move switch or set jumpers)
   # Test in code with py532lib
   ```

   **Step 3 - USB Audio:**
   ```
   # Plug USB audio adapter into Pi's USB port
   # No wiring needed
   
  - **Beginner alternative:** Download MP3s locally, store in `/cube/tracks/` folder
  - Map track_id to filename: `"track_001" → "tracks/midnight_frequencies.mp3"`
  - Simpler for MVP, no API complexity
- **NTP:** Cubes must sync time with `pool.ntp.org` on boot for accurate playback sync
  - Pre-installed on Raspberry Pi OS, runs automatically
  - Verify with `timedatectl status` - should show "System clock synchronized: yes"
- **Network:** Requires <50ms latency between cubes and server for sub-100ms sync accuracy
  - Test with `ping YOUR_SERVER_IP` from Pi
  - Both cubes should be on same WiFi network for best results

## Troubleshooting Guide for Beginners

**Pi won't boot:**
- ✓ Check power supply provides 2.5A minimum
- ✓ Re-flash SD card with Raspberry Pi Imager
- ✓ Try different SD card (cheap cards often fail)

**Can't SSH to Pi:**
- ✓ Verify SSH enabled in Raspberry Pi Imager settings
- ✓ Check WiFi credentials are correct
- ✓ Use IP scanner app to find Pi on network
- ✓ Try `ssh pi@<IP_ADDRESS>` instead of `.local`

**OLED shows nothing:**
- ✓ Run `i2cdetect -y 1` - should see device at 0x3C
- ✓ Check 3.3V power connection (NOT 5V!)
- ✓ Verify I2C enabled: `sudo raspi-config` → Interface Options → I2C

**NFC reader not detecting cards:**
- ✓ Check PN532 is in SPI mode (switches/jumpers)
- ✓ Verify SPI enabled in raspi-config
- ✓ Hold card flat, 1-2cm from reader
- ✓ Try different card (some aren't NTAG215)

**No audio output via Bluetooth:**
- ✓ Check Bluetooth speaker is paired: `bluetoothctl devices`
- ✓ Connect to speaker: `bluetoothctl connect XX:XX:XX:XX:XX:XX`
- ✓ Check PulseAudio sinks: `pactl list sinks short`
- ✓ Set default sink: `pactl set-default-sink bluez_sink.XX_XX_XX_XX_XX_XX.a2dp_sink`
- ✓ Test: `paplay /usr/share/sounds/alsa/Front_Center.wav`
- ✓ Increase volume: `pactl set-sink-volume @DEFAULT_SINK@ 80%`

**WebSocket won't connect:**
- ✓ Check server is running: `curl http://localhost:8000/health`
- ✓ Verify firewall allows port 8000
- ✓ Use Mac's local IP (not localhost) in cube config
- ✓ Find IP: `ifconfig | grep "inet "` on Mac

**Cubes out of sync:**
- ✓ Check NTP: `timedatectl status` on both Pis
- ✓ Increase buffer_duration in sync protocol (5-10 seconds)
- ✓ Test network latency: `ping` between Pi and server
- ✓ Use longer tracks (>2min) to reduce noticeable drift

## Simplified 3D Design (First-Timer Friendly)

**Option A: Use existing design** (Thingiverse search "NFC card holder box")
- Download STL, modify dimensions in Tinkercad
- No CAD experience needed

**Option B: Simple box design** (30 minutes in Tinkercad):
1. Create 120mm cube
2. Hollow out walls (3mm thickness)
3. Cut slot for NFC card (90mm × 60mm × 2mm)
4. Cut window for OLED (30mm × 15mm)
5. Add speaker grill holes (10mm diameter, grid pattern)
6. Export as STL

**Option C: No enclosure** (if time-constrained):
- Mount everything on acrylic sheet
- 3D print just the top panel with card slot
- Use standoffs/screws to secure components
- **This is perfectly acceptable for a gift!**
   ```

   **Step 4 - Speaker:**
   ```
   # Connect 3.5mm speaker cable to USB audio adapter output
   # Test with sample audio file:
   aplay -D plughw:1,0 /usr/share/sounds/alsa/Front_Center.wav
   ```

**Common beginner mistakes to avoid:**
- ❌ Connecting power while Pi is on (always shutdown first)
- ❌ Using 5V instead of 3.3V for OLED/NFC (will damage components)
- ❌ Forgetting to enable I2C/SPI in raspi-config
- ❌ Wrong PN532 mode (must be SPI, not I2C or UART)
- ❌ Not pairing Bluetooth speaker before testing audio
- ❌ Not testing each component individually before combining

**Decision: Use Raspberry Pi Zero 2 W only** - ESP32 requires C++ and more complex setup, not suitable for first hardware project

## Tech Stack
- **Cube:** Python 3.11+, nfcpy/py532lib (NFC), pygame/python-vlc (audio), Luma.OLED (display), websockets
- **Server:** Node.js (Express + Socket.io) OR Python (FastAPI + WebSockets), Redis, PostgreSQL
- **Hardware:** Raspberry Pi Zero 2 W, PN532 NFC module, SSD1306 OLED, USB audio

## Key Development Workflows

**Test cube hardware (without Pi):**
```bash
python cube/test_nfc.py          # Test NFC reader only
python cube/test_display.py      # Test OLED display
python cube/test_audio.py        # Test audio playback
```

**Run server locally:**
```bash
cd server && npm install && npm run dev     # Node.js version
# OR
cd server && pip install -r requirements.txt && python app.py
```

**Deploy to Raspberry Pi:**
```bash
# From development machine:
rsync -avz cube/ pi@cube1.local:~/vibecube/
ssh pi@cube1.local "cd ~/vibecube && python main.py"
```

**Sync both cubes for testing:**
```bash
# Terminal 1: Start server
cd server && npm run dev

# Terminal 2: Simulate cube A
python cube/main.py --cube-id cube_a --mock-nfc track_001

# Terminal 3: Simulate cube B
python cube/main.py --cube-id cube_b --mock-nfc track_001
```

## Critical Patterns

### Synchronization Protocol
Server broadcasts play command with Unix timestamp for simultaneous start:
```python
{
  "event": "play_synchronized",
  "track_url": "https://soundcloud.com/...",
  "start_timestamp": 1706400000.500,  # Server time + buffer (3-5s)
  "buffer_duration": 3.0
}
```

Cubes calculate wait time and start playback at exact timestamp:
```python
wait_time = msg["start_timestamp"] - time.time()
track.buffer()  # Pre-load audio
time.sleep(wait_time)
track.play()
```

### NFC Card Data Format
Store JSON on NTAG215 cards (NDEF format):
```json
{"track_id": "soundcloud_12345678", "track_name": "Midnight Frequencies", "artist": "DJ_Name"}
```

Read in `nfc_reader.py`:
```python
def on_card_detected(tag):
    data = json.loads(tag.ndef.records[0].text)
    emit_to_server("card_inserted", data)
```

### State Management
Each cube maintains local state synchronized with server:
```python
class CubeState:
    status: Enum["idle", "waiting", "synced", "playing"]
    inserted_card: Optional[dict]
    partner_ready: bool
```

### Error Handling
- **Network loss:** Cache last 3 tracks locally, continue playback offline
- **NFC read failure:** Retry up to 3 times before showing error on OLED
- **Audio buffer underrun:** Pre-buffer 5 seconds minimum before playback

## Hardware-Specific Considerations

**GPIO Pin Assignments (Raspberry Pi):**
```python
NFC_SPI_CS = 8      # PN532 via SPI
OLED_I2C_ADDR = 0x3C  # SSD1306 via I2C
BLUETOOTH_SPEAKER = "bluez_sink.XX_XX_XX_XX_XX_XX.a2dp_sink"  # Bluetooth audio via PulseAudio
```

**Power Management:**
- Implement standby mode after 5 minutes idle (dim OLED, stop polling NFC)
- Wake on GPIO interrupt when card approaches NFC reader

**Testing Without Hardware:**
- Use `--mock-nfc` flag to simulate card insertion via CLI
- Use `--mock-display` to print OLED output to terminal instead of hardware
- Audio can test with desktop speakers via pygame

## Conventions

- **Timing:** All timestamps in Unix seconds (float) with millisecond precision
- **Logging:** Use `logging` module with cube_id prefix: `[CUBE_A] Card detected: track_001`
- **Config:** Each cube has `config.json` with unique `cube_id`, WiFi credentials, server URL
- **Dependencies:** Pin versions in requirements.txt (hardware libraries can be finicky)
- **Deployment:** Use systemd service on Pi for auto-start: `vibecube.service`

## File Organization
```
/cube/               # Raspberry Pi client code
  main.py            # Main event loop
  nfc_reader.py      # NFC card detection
  audio_player.py    # Synchronized playback
  display_manager.py # OLED status display
  server_client.py   # WebSocket connection
  config.json        # Cube-specific settings

/server/             # Coordination backend
  app.py             # Main server entry
  sync_manager.py    # Playback synchronization logic
  soundcloud_api.py  # Track fetching

/hardware/           # 3D models and schematics
  cube_model.f3d     # Fusion 360 design
  wiring_diagram.png

/cards/              # NFC card designs
  card_template.psd

/tests/              # Hardware mocking and unit tests
```

## Common Tasks

**Add new track:** Write NFC card with `write_card.py` script, provide SoundCloud URL
**Update cube software:** SSH to Pi, git pull, restart systemd service
**Debug sync issues:** Check server logs for timestamp calculations, verify NTP is running on Pi
**Pair new cube:** Add entry to server's `cubes` table with unique cube_id

## External Dependencies
- **SoundCloud API:** Requires client_id in server env vars, rate limited to 15k requests/day
- **NTP:** Cubes must sync time with `pool.ntp.org` on boot for accurate playback sync
- **Network:** Requires <50ms latency between cubes and server for sub-100ms sync accuracy
