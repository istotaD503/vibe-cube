# VibeCube - Complete Build Guide

**Gift for Two DJ Friends | Deadline: February 15, 2026**

## 🎯 Project Overview

VibeCube creates a shared listening experience for two DJs. Each physical cube reads NFC cards representing their SoundCloud tracks. When both DJs insert the same card, the tracks play **simultaneously** on their Bluetooth speakers, creating a synchronized "vibe" session despite physical distance.

**Key Design Decisions:**
- ✅ **Raspberry Pi Zero 2 W** - Easy Python development, built-in WiFi/Bluetooth
- ✅ **External Bluetooth speakers** - DJs already own quality speakers (saves $36/cube)
- ✅ **Simple in-memory server** - Skip Redis/PostgreSQL for MVP
- ✅ **Local MP3 storage option** - Simpler than SoundCloud API for first version

**Total Cost: ~$100 for both cubes** (down from $150 by using Bluetooth)

---

## 🚨 URGENT: Order Components TODAY (Jan 28)

**These must arrive by Jan 30-31 to hit Feb 15 deadline!**

### Amazon Prime 2-Day Shipping
```
Component                           Qty    Unit    Total
──────────────────────────────────────────────────────
Raspberry Pi Zero 2 W               x2     $15     $30
  Alt: Pi 4 2GB if Zero unavailable x2     $45     $90
PN532 NFC RFID Module               x2     $8      $16
  (HiLetgo or Elechouse brand, SPI/I2C/UART)
SSD1306 OLED 0.96" I2C Display      x2     $5      $10
  (Must be I2C: 4 pins, NOT SPI 7 pins)
MicroSD Card 16GB Class 10          x2     $6      $12
  (SanDisk or Samsung + SD adapter)
USB-C Power Supply 5V 2.5A          x2     $8      $16
  (Or use existing phone chargers)
NTAG215 NFC Cards 10-pack           x1     $8      $8
  (amiibo-compatible)
Female-Female Jumper Wires 40pc     x1     $6      $6
──────────────────────────────────────────────────────
TOTAL                                              $98
```

**Search Amazon for these exact terms:**
- "Raspberry Pi Zero 2 W GPIO header"
- "HiLetgo PN532 NFC RFID Module"
- "SSD1306 OLED Display 0.96 I2C"
- "SanDisk microSD 16GB Class 10"
- "NTAG215 NFC Cards 10 pack"
- "Female to Female Jumper Wires 40 piece"

### User-Provided (Ask DJ Friends)
- **Bluetooth speakers (x2)** - Any portable speaker (JBL, UE Boom, Anker, Bose)
- Must support A2DP audio profile (all modern speakers do)
- They likely already own these!

### 3D Printing (Order by Jan 30!)
- **Local makerspace** or online (Shapeways, Craftcloud, Treatstock)
- **Material:** Black PLA, 20% infill
- **Cost:** $30-50 per cube
- **Lead time:** 7-10 days → order no later than Jan 30
- **Backup:** Cardboard box with cutouts (still looks great!)

---

## 📋 Day-by-Day Timeline (18 Days)

### Week 1: Setup & Individual Components

#### Day 1 (Jan 28 - TODAY) ⏰
```bash
# Order all components above
# Set up development environment
cd ~/Playground/VibeCube
python3 -m venv venv
source venv/bin/activate
pip install websockets pygame pillow

# Create project structure
mkdir -p cube server shared tests hardware cards cube/tracks
touch cube/main.py cube/test_display.py cube/test_nfc.py cube/test_bluetooth.py
touch server/app.py shared/protocol.py
```

#### Day 2 (Jan 29)
**Develop with mocks** (no hardware needed):
- [ ] Create `cube/display_mock.py` - prints to terminal
- [ ] Create `cube/nfc_mock.py` - simulates card insertion
- [ ] Create `cube/audio_player.py` - plays via pygame
- [ ] Test: `python cube/test_audio.py` (should hear test sound)

#### Day 3 (Jan 30)
**Components arrive!**
- [ ] Download Raspberry Pi Imager: https://www.raspberrypi.com/software/
- [ ] Flash "Raspberry Pi OS Lite (64-bit)" to microSD
- [ ] **CRITICAL:** Enable SSH and WiFi in imager settings (gear icon)
- [ ] Set username: `pi`, password: `raspberry`
- [ ] **Order 3D prints today** (last day for Feb 11 delivery!)

#### Day 4 (Jan 31)
**First Pi boot:**
```bash
# Power on Pi, wait 2 minutes for boot
ssh pi@raspberrypi.local
# If .local doesn't work, use IP scanner app to find Pi's IP

# On Pi:
sudo raspi-config
  # → System Options → Hostname → "cube1"
  # → Interface Options → I2C → Enable
  # → Interface Options → SPI → Enable
  # → Update
sudo reboot

# Install dependencies
ssh pi@cube1.local
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-pip python3-venv git i2c-tools
sudo apt install -y python3-pygame python3-pil
sudo apt install -y pulseaudio pulseaudio-module-bluetooth bluez

# Clone your repo (or create if first time)
git clone https://github.com/YOUR_USERNAME/vibe-cube.git ~/vibecube
cd ~/vibecube
python3 -m venv venv
source venv/bin/activate
```

#### Day 5 (Feb 1)
**Connect OLED display:**
```
Component Wiring (Use Female-Female Jumper Wires)
───────────────────────────────────────────────────
OLED VCC  →  Pi Pin 1  (3.3V Power - RED wire)
OLED GND  →  Pi Pin 6  (Ground - BLACK wire)
OLED SDA  →  Pi Pin 3  (GPIO 2 / SDA - BLUE wire)
OLED SCL  →  Pi Pin 5  (GPIO 3 / SCL - YELLOW wire)
```

**Test display:**
```bash
ssh pi@cube1.local
i2cdetect -y 1
# Should show "3c" in the grid → OLED detected

cd ~/vibecube
pip install luma.oled
python cube/test_display.py
# OLED should light up with "VibeCube Ready"
```

**If nothing shows:** Check 3.3V (NOT 5V!), verify I2C enabled, reseat wires

#### Day 6 (Feb 2)
**Connect NFC reader:**
```
PN532 Wiring (SPI Mode - Check PN532 Board Settings!)
──────────────────────────────────────────────────
PN532 VCC   →  Pi Pin 17  (3.3V Power)
PN532 GND   →  Pi Pin 20  (Ground)
PN532 MOSI  →  Pi Pin 19  (GPIO 10 / MOSI)
PN532 MISO  →  Pi Pin 21  (GPIO 9 / MISO)
PN532 SCK   →  Pi Pin 23  (GPIO 11 / SCLK)
PN532 SS    →  Pi Pin 24  (GPIO 8 / CE0)
```

**CRITICAL: Set PN532 to SPI mode:**
- Check board for switches/jumpers (refer to PN532 manual)
- Common: Switch 1=OFF, Switch 2=ON for SPI
- Or specific jumper configuration (see board markings)

**Test NFC:**
```bash
ssh pi@cube1.local
cd ~/vibecube
pip install py532lib
python cube/test_nfc.py
# Hold NTAG215 card 1-2cm from reader
# Should print: "Card detected! UID: XX:XX:XX:XX"
```

**If not detected:** Verify SPI enabled, check PN532 mode switches, try different card

#### Day 7 (Feb 3)
**Pair Bluetooth speaker:**
```bash
ssh pi@cube1.local

# Ensure Bluetooth service running
sudo systemctl enable bluetooth
sudo systemctl start bluetooth

# Put speaker in pairing mode (hold Bluetooth button)
bluetoothctl
> power on
> agent on
> default-agent
> scan on
# Wait 10 seconds - look for your speaker name
# Example output: [NEW] Device 12:34:56:78:9A:BC JBL Flip 5
> pair 12:34:56:78:9A:BC  # Use your speaker's MAC
> trust 12:34:56:78:9A:BC
> connect 12:34:56:78:9A:BC
> exit

# Test audio
paplay /usr/share/sounds/alsa/Front_Center.wav
# Should hear from Bluetooth speaker!

# Set volume
pactl set-sink-volume @DEFAULT_SINK@ 80%
```

---

### Week 2: Integration & Testing

#### Day 8 (Feb 4)
**Test all hardware together:**
```bash
ssh pi@cube1.local
cd ~/vibecube
python cube/main.py --cube-id cube1

# Insert NFC card:
# → OLED shows track name
# → Connects to server
# → Ready to vibe!
```

#### Day 9 (Feb 5)
**Build sync server:**
```bash
# On your Mac
cd ~/Playground/VibeCube/server
pip install fastapi uvicorn websockets

# Create server/app.py (see Technical Details section below)
python app.py
# Server listening on ws://0.0.0.0:8000
```

#### Day 10 (Feb 6)
**Connect cube to server:**
```bash
# Find your Mac's local IP
# Mac: System Settings → Network → WiFi → Details
# Or: ifconfig | grep "inet " | grep -v 127.0.0.1
# Example: 192.168.1.100

# On Pi
ssh pi@cube1.local
cd ~/vibecube
# Edit cube/config.json:
{
  "cube_id": "cube1",
  "server_url": "ws://192.168.1.100:8000"
}

python cube/main.py
# Should connect to server!
```

#### Day 11 (Feb 7)
**Set up second Pi (cube2):**
- Repeat Days 4-7 for second Pi
- Change hostname to "cube2"
- Same software installation
- Different cube_id in config

#### Day 12 (Feb 8)
**Test two-cube sync:**
```bash
# Terminal 1 (Mac): Start server
cd ~/Playground/VibeCube/server
python app.py

# Terminal 2: Cube 1
ssh pi@cube1.local
cd ~/vibecube && python cube/main.py

# Terminal 3: Cube 2
ssh pi@cube2.local
cd ~/vibecube && python cube/main.py

# Insert SAME NFC card in both cubes within 10 seconds
# Both OLEDs show: "SYNCED! Starting in 5s..."
# Both speakers play simultaneously! 🎉
```

**Measure sync accuracy:**
- Record both speakers with phone camera
- Import to video editor
- Check audio waveforms align (<100ms = success)

#### Day 13 (Feb 9)
**Write NFC cards with track data:**
```bash
# Create script: scripts/write_nfc_card.py
cd ~/Playground/VibeCube
python scripts/write_nfc_card.py \
  --track-id track_001 \
  --track-name "Midnight Frequencies" \
  --artist "DJ_Alex"

# Write 5-10 cards with different tracks
```

#### Day 14 (Feb 10)
**Refinement & testing:**
- [ ] Adjust sync `buffer_duration` (3-10 seconds based on network)
- [ ] Add OLED animations (startup, waiting, playing)
- [ ] Implement card removal detection
- [ ] Test network disconnection recovery
- [ ] Add status LED indicators (optional)

**Backup to GitHub:**
```bash
git add .
git commit -m "Working two-cube prototype"
git push origin main
```

---

### Week 3: Assembly & Polish

#### Day 15 (Feb 11)
**3D printed enclosures arrive!**
- [ ] Dry fit all components (don't glue yet)
- [ ] Check NFC card slides smoothly
- [ ] Verify OLED visible through window
- [ ] Test cable routing (power + tight bends)
- [ ] Mark screw hole positions

#### Day 16 (Feb 12)
**Final assembly:**
1. Mount Pi to enclosure base (standoffs or hot glue)
2. Mount OLED in front window (small screws or careful hot glue)
3. Mount NFC reader under card slot (double-sided tape or glue)
4. Route power cable through back opening
5. Close enclosure (screws or snap-fit)
6. **Test everything still works before sealing!**
7. Repeat for cube 2

**Assembly tips:**
- Test before gluing anything permanent
- Leave access to microSD card for updates
- Keep NFC reader within 5mm of slot surface
- Strain relief on power cable

#### Day 17 (Feb 13)
**Final testing & bug fixes:**
- [ ] Power cycle test (unplug, replug, auto-reconnect)
- [ ] NFC card insertion/removal 20 times each cube
- [ ] Test with 5+ different tracks
- [ ] Verify sync accuracy (<500ms acceptable)
- [ ] Check Bluetooth reconnection after speaker off/on
- [ ] Stress test: Leave running 2 hours

**Common bugs to fix:**
- WebSocket reconnection logic
- NFC reader polling timeout
- OLED screen burn-in prevention
- Audio sync drift over time

#### Day 18 (Feb 14)
**Polish & presentation:**
- [ ] Design NFC card artwork (Canva, Photoshop)
- [ ] Print card labels or use sticker paper
- [ ] Clean cube exteriors (damp cloth, remove support material)
- [ ] Write 1-page user guide:
  - How to power on
  - How to insert card
  - What each OLED message means
  - Troubleshooting WiFi issues
- [ ] Package in nice box with cards

#### Day 19 (Feb 15) 🎁
**GIFT DAY!**
- Demo for your friends
- Watch their faces light up! 🎵✨
- Capture their reaction on video
- Enjoy the vibe session together

---

## 🏗️ Technical Architecture

### System Overview
```
┌─────────────────┐         ┌─────────────────┐
│   Cube A (DJ1)  │         │   Cube B (DJ2)  │
│                 │         │                 │
│  ┌───────────┐  │         │  ┌───────────┐  │
│  │NFC Reader │  │         │  │NFC Reader │  │
│  │   OLED    │  │         │  │   OLED    │  │
│  │  Pi Zero  │◄─┼─────────┼─►│  Pi Zero  │  │
│  │ Bluetooth │  │  WiFi   │  │ Bluetooth │  │
│  └─────┬─────┘  │         │  └─────┬─────┘  │
└────────┼────────┘         └────────┼────────┘
         │                           │
      [BT Speaker]              [BT Speaker]
         │                           │
         └──────────┬────────────────┘
                    │
              ┌─────▼─────┐
              │  Server   │
              │ (Mac/Pi4) │
              │ FastAPI + │
              │ WebSockets│
              └───────────┘
```

### Software Stack

**Cube Client (`/cube/`):**
- **Language:** Python 3.11+
- **Dependencies:**
  - `py532lib` - NFC card reading via SPI
  - `luma.oled` - SSD1306 OLED display via I2C
  - `pygame` - Audio playback
  - `websockets` - Real-time server connection
  - `pulseaudio` - Bluetooth audio routing

**Sync Server (`/server/`):**
- **Language:** Python 3.11+ (keep consistent)
- **Framework:** FastAPI + WebSockets
- **State:** In-memory (dict) for MVP
- **Deployment:** Mac for testing, Railway/Render for production

### File Structure
```
VibeCube/
├── cube/
│   ├── main.py              # Main event loop
│   ├── nfc_reader.py        # NFC detection with py532lib
│   ├── audio_player.py      # Synchronized playback
│   ├── display_manager.py   # OLED status updates
│   ├── server_client.py     # WebSocket connection
│   ├── config.json          # Cube ID, server URL
│   ├── tracks/              # Local MP3 storage
│   └── test_*.py            # Hardware test scripts
├── server/
│   ├── app.py               # FastAPI + WebSocket server
│   ├── sync_manager.py      # Playback coordination
│   └── requirements.txt
├── shared/
│   └── protocol.py          # Message format definitions
├── scripts/
│   └── write_nfc_card.py    # NFC card writer
├── hardware/
│   ├── cube_model.stl       # 3D printable enclosure
│   └── wiring_diagram.png
└── cards/
    └── card_template.psd    # NFC card design
```

### Synchronization Protocol

**Message Format:**
```python
# Cube → Server: Card inserted
{
  "event": "card_inserted",
  "cube_id": "cube1",
  "card": {
    "track_id": "track_001",
    "track_name": "Midnight Frequencies",
    "artist": "DJ_Alex"
  }
}

# Server → All Cubes: Ready to sync
{
  "event": "play_synchronized",
  "track_id": "track_001",
  "track_name": "Midnight Frequencies",
  "start_timestamp": 1706400005.500,  # Unix time (5s from now)
  "buffer_duration": 5.0
}

# Cube playback logic:
wait_time = start_timestamp - time.time()
audio.load_track(track_id)
time.sleep(wait_time)
audio.play()  # Synchronized start!
```

**Sync Accuracy:**
- **Target:** <100ms between cubes
- **Achievable:** 50-200ms on typical home WiFi
- **Method:** NTP time sync + server-coordinated timestamps
- **Drift correction:** Resync every 30 seconds if playing long tracks

### NFC Card Data Format

**NDEF Text Record (JSON):**
```json
{
  "track_id": "track_001",
  "track_name": "Midnight Frequencies",
  "artist": "DJ_Alex",
  "duration_sec": 245,
  "created_at": "2026-01-15"
}
```

**Writing cards:**
```python
import ndef
from py532lib.py532 import Py532

# Write to NTAG215
record = ndef.TextRecord(json.dumps(track_data))
nfc.write_ndef([record])
```

---

## 🔧 Hardware Setup Details

### GPIO Pin Assignments
```python
# Raspberry Pi Zero 2 W Pin Layout
#
#    3V3  [1]  [2]  5V
#  GPIO2  [3]  [4]  5V      ← OLED SDA to Pin 3
#  GPIO3  [5]  [6]  GND     ← OLED SCL to Pin 5, GND to Pin 6
#  GPIO4  [7]  [8]  GPIO14
#    GND  [9]  [10] GPIO15
# GPIO17  [11] [12] GPIO18
# GPIO27  [13] [14] GND
# GPIO22  [15] [16] GPIO23
#    3V3  [17] [18] GPIO24  ← PN532 VCC to Pin 17
# GPIO10  [19] [20] GND     ← PN532 MOSI to Pin 19, GND to Pin 20
#  GPIO9  [21] [22] GPIO25  ← PN532 MISO to Pin 21
# GPIO11  [23] [24] GPIO8   ← PN532 SCK to Pin 23, SS to Pin 24
#    GND  [25] [26] GPIO7

# In code:
OLED_I2C_ADDR = 0x3C
NFC_SPI_CS = 8  # GPIO 8 (Pin 24)
```

### Power Requirements
- **Raspberry Pi Zero 2 W:** 5V @ 1.2A typical (2.5A max)
- **OLED Display:** 3.3V @ 20mA
- **PN532 NFC:** 3.3V @ 50mA (during read)
- **Total:** 5V @ 2.5A power supply recommended

### Enclosure Dimensions
```
Outer cube: 120mm x 120mm x 120mm
Wall thickness: 3mm
NFC card slot: 90mm (W) x 60mm (H) x 2mm (D)
OLED window: 30mm (W) x 15mm (H)
Ventilation holes: 3mm diameter, spaced 10mm apart
Power cable opening: 10mm diameter
```

---

## 🐛 Troubleshooting Guide

### Pi Won't Boot
- ✓ Try different power supply (2.5A minimum)
- ✓ Re-flash SD card with Raspberry Pi Imager
- ✓ Test SD card in computer (may be corrupted)
- ✓ Check green LED blinks (activity indicator)

### Can't SSH to Pi
- ✓ Verify SSH enabled in imager settings before flashing
- ✓ Check WiFi credentials match your network
- ✓ Use IP scanner app (Fing, Angry IP Scanner)
- ✓ Try direct Ethernet connection with adapter
- ✓ Connect monitor + keyboard (HDMI + USB OTG)

### OLED Shows Nothing
- ✓ Run `i2cdetect -y 1` - should show `3c`
- ✓ Check wiring: 3.3V NOT 5V (5V damages OLED!)
- ✓ Enable I2C: `sudo raspi-config → Interface Options`
- ✓ Swap SDA/SCL wires (sometimes labeled wrong)
- ✓ Try different OLED (1-2% arrive DOA)

### NFC Reader Not Working
- ✓ Verify PN532 in SPI mode (check switches/jumpers)
- ✓ Enable SPI: `sudo raspi-config → Interface Options`
- ✓ Check 3.3V power connection
- ✓ Hold card flat, 1-2cm from reader center
- ✓ Try different card (verify it's NTAG215)
- ✓ Test with different reader (PN532 can be finicky)

### No Bluetooth Audio
- ✓ Check speaker is paired: `bluetoothctl devices`
- ✓ Reconnect: `bluetoothctl connect XX:XX:XX:XX:XX:XX`
- ✓ List sinks: `pactl list sinks short`
- ✓ Set default: `pactl set-default-sink bluez_sink...`
- ✓ Test: `paplay /usr/share/sounds/alsa/Front_Center.wav`
- ✓ Check volume: `pactl set-sink-volume @DEFAULT_SINK@ 80%`
- ✓ Restart PulseAudio: `pulseaudio -k && pulseaudio --start`

### WebSocket Won't Connect
- ✓ Verify server running: `curl http://localhost:8000/health`
- ✓ Check firewall allows port 8000
- ✓ Use Mac's LAN IP (192.168.x.x), not localhost
- ✓ Test from Pi: `ping YOUR_MAC_IP`
- ✓ Check both on same WiFi network

### Cubes Out of Sync
- ✓ Verify NTP: `timedatectl status` (should show "synchronized: yes")
- ✓ Increase `buffer_duration` to 10 seconds
- ✓ Test latency: `ping YOUR_SERVER_IP` (<50ms ideal)
- ✓ Use wired Ethernet if WiFi is weak
- ✓ Close bandwidth-heavy apps during testing

### Audio Pops/Clicks
- ✓ Increase PulseAudio buffer: Edit `/etc/pulse/daemon.conf`
  ```
  default-fragment-size-msec = 25
  default-fragments = 8
  ```
- ✓ Use higher quality MP3s (320kbps)
- ✓ Move Pi closer to Bluetooth speaker
- ✓ Restart both Pi and speaker

---

## 🎨 3D Design Guide

### Design Software Options

**Option 1: Tinkercad (Easiest)**
- Browser-based, free
- 30-minute learning curve
- Perfect for simple boxes
- Export STL directly

**Option 2: Fusion 360 (Best)**
- Free for hobbyists/students
- Parametric design (easy modifications)
- Professional results
- 2-hour learning curve

**Option 3: Use Template**
- Search Thingiverse: "NFC card holder box 120mm"
- Modify existing design in Tinkercad
- Fastest path to printing

### Design Steps (Tinkercad)
1. Create 120mm cube
2. Hollow interior (117mm inner cube)
3. Cut NFC slot: 90mm x 60mm x 10mm deep, angled 45°
4. Cut OLED window: 30mm x 15mm x 2mm deep
5. Cut ventilation holes: 3mm diameter, grid of 4x4
6. Cut power cable hole: 10mm diameter
7. Add screw posts: 3mm diameter cylinders, 10mm height
8. Export as STL

### Printing Settings
```
Material: PLA (black)
Layer height: 0.2mm
Wall thickness: 3 perimeters (1.2mm)
Infill: 20% gyroid
Supports: Yes (for card slot overhang)
Brim/Raft: Brim recommended
Print time: ~15 hours
```

---

## 🚀 Future Enhancements (Post-Feb 15)

### Phase 2 Ideas
1. **LED Ambient Lighting**
   - WS2812B LED strip (5 LEDs, $3)
   - Pulses with music beats (FFT analysis)
   - Different colors per DJ

2. **E-ink Display**
   - 2.13" e-ink (Waveshare, $15)
   - Shows album art from SoundCloud
   - Lower power, always-on display

3. **Battery Power**
   - 18650 cells + TP4056 charger ($10)
   - 6-8 hours playback
   - Truly portable cubes

4. **Mobile App**
   - React Native app
   - Upload new tracks remotely
   - View listening history
   - Pair new cubes

5. **Multi-Cube Support**
   - Server supports 3+ cubes
   - Group listening sessions
   - "Join any vibe in progress"

6. **Voice Messages**
   - Record 5-second message with card
   - Plays before track starts
   - "Yo, check this out!"

---

## 📚 Resources

### Official Documentation
- Raspberry Pi: https://www.raspberrypi.com/documentation/
- PN532: https://www.nxp.com/docs/en/user-guide/141520.pdf
- SSD1306: https://cdn-shop.adafruit.com/datasheets/SSD1306.pdf

### Libraries
- py532lib: https://github.com/HubertD/py532lib
- luma.oled: https://luma-oled.readthedocs.io/
- pygame: https://www.pygame.org/docs/
- FastAPI: https://fastapi.tiangolo.com/

### Communities
- r/raspberry_pi - Hardware help
- r/Python - Code assistance
- Adafruit Forums - NFC/OLED troubleshooting

---

## ✅ Pre-Flight Checklist

**Before ordering components:**
- [ ] Confirmed DJ friends have Bluetooth speakers
- [ ] Verified they're excited about the gift
- [ ] Have Amazon Prime for 2-day shipping
- [ ] Know where to get 3D printing locally

**Day 1 (Today):**
- [ ] All components ordered on Amazon
- [ ] 3D print service contacted
- [ ] Development environment set up
- [ ] GitHub repo created

**Day 3:**
- [ ] Components arrived
- [ ] 3D prints ordered
- [ ] First Pi flashed and booting

**Day 7:**
- [ ] All hardware tested individually
- [ ] OLED displaying correctly
- [ ] NFC reading cards
- [ ] Bluetooth audio working

**Day 14:**
- [ ] Two cubes syncing perfectly
- [ ] All bugs fixed
- [ ] Code backed up to GitHub

**Day 15:**
- [ ] 3D enclosures arrived
- [ ] Ready for final assembly

**Day 19:**
- [ ] 🎁 GIFT READY!

---

## 🎵 Final Thoughts

This project is ambitious but absolutely achievable in 18 days. The key success factors:

1. **Order components TODAY** - The timeline has zero slack
2. **Test incrementally** - Don't wait to combine everything
3. **Use what works** - Local MP3s > SoundCloud API for MVP
4. **Embrace imperfection** - A working prototype beats a perfect concept
5. **Ask for help** - Reddit, Discord, forums are full of helpful people

The emotional payoff of holding a physical card representing 100 hours of creative work, then sharing that moment with a friend in real-time across any distance, will make all the effort worth it.

**You've got this! Start ordering components NOW! 🚀**
