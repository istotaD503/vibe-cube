# VibeCube Quick Start - Prototype Today (Jan 28)

## What You Have
- ✅ Raspberry Pi 3 B+ (perfect for prototyping!)
- ✅ LCD screen (can use instead of OLED for now)
- ✅ Your Mac for development

## What You Can Test TODAY (Without Ordering Anything)

### Phase 1: Software Development on Mac (2 hours)
Test everything with mocks - no hardware needed

### Phase 2: Deploy to Pi 3 B+ (2 hours)  
Test real display, real Bluetooth, simulated NFC

### Phase 3: Two-Cube Sync Test (1 hour)
Use Pi + Mac to simulate two cubes

**Total: ~5 hours to validate core concept**

---

## Phase 1: Mock Everything on Mac (Start Here!)

### Step 1: Set Up Project (15 min)
```bash
cd ~/Playground/VibeCube

# Create project structure
mkdir -p cube server shared tests
mkdir -p cube/tracks  # For local MP3s

# Set up Python environment
python3 -m venv venv
source venv/bin/activate
pip install pygame pillow websockets
```

### Step 2: Create Mock NFC Reader (15 min)
Create `cube/nfc_mock.py`:
```python
"""Mock NFC reader for testing without hardware"""
import json

class MockNFCReader:
    def __init__(self):
        self.current_card = None
    
    def simulate_card_insert(self, track_id, track_name, artist):
        """Simulate inserting an NFC card"""
        self.current_card = {
            "track_id": track_id,
            "track_name": track_name,
            "artist": artist
        }
        print(f"[MOCK NFC] Card inserted: {track_name} by {artist}")
        return self.current_card
    
    def simulate_card_remove(self):
        """Simulate removing card"""
        if self.current_card:
            print(f"[MOCK NFC] Card removed: {self.current_card['track_name']}")
            self.current_card = None
    
    def get_current_card(self):
        return self.current_card

if __name__ == "__main__":
    # Test the mock
    reader = MockNFCReader()
    reader.simulate_card_insert("track_001", "Midnight Frequencies", "DJ_Alex")
    print(f"Current card: {reader.get_current_card()}")
    reader.simulate_card_remove()
```

**Test it:**
```bash
python cube/nfc_mock.py
# Should print card insertion/removal messages
```

### Step 3: Create Mock Display (15 min)
Create `cube/display_mock.py`:
```python
"""Mock OLED display - prints to terminal"""

class MockDisplay:
    def __init__(self):
        self.width = 128
        self.height = 64
        print(f"[MOCK DISPLAY] Initialized {self.width}x{self.height}")
    
    def clear(self):
        print("\n" + "="*50)
        print("[MOCK DISPLAY] Screen cleared")
    
    def show_text(self, lines):
        """Show text lines on display"""
        self.clear()
        for line in lines:
            print(f"[MOCK DISPLAY] {line}")
        print("="*50 + "\n")
    
    def show_status(self, status):
        """Show status message"""
        self.show_text([
            "VibeCube",
            "",
            status
        ])

if __name__ == "__main__":
    # Test the mock
    display = MockDisplay()
    display.show_status("Ready")
    display.show_text(["Insert card", "to start vibing"])
```

**Test it:**
```bash
python cube/display_mock.py
# Should show formatted output in terminal
```

### Step 4: Create Audio Player (30 min)
Create `cube/audio_player.py`:
```python
"""Audio player with sync support"""
import pygame
import time
from pathlib import Path

class AudioPlayer:
    def __init__(self, tracks_dir="cube/tracks"):
        pygame.mixer.init()
        self.tracks_dir = Path(tracks_dir)
        self.current_track = None
        print(f"[AUDIO] Initialized, tracks dir: {self.tracks_dir}")
    
    def load_track(self, track_id):
        """Load track by ID (looks for track_id.mp3 or .wav)"""
        for ext in ['.mp3', '.wav', '.ogg']:
            track_path = self.tracks_dir / f"{track_id}{ext}"
            if track_path.exists():
                pygame.mixer.music.load(str(track_path))
                self.current_track = track_id
                print(f"[AUDIO] Loaded: {track_path.name}")
                return True
        
        print(f"[AUDIO] Track not found: {track_id}")
        return False
    
    def play_at_timestamp(self, start_timestamp, buffer_duration=3.0):
        """Play at specific Unix timestamp"""
        current_time = time.time()
        wait_time = start_timestamp - current_time
        
        if wait_time > 0:
            print(f"[AUDIO] Waiting {wait_time:.3f}s until sync time...")
            time.sleep(wait_time)
        else:
            print(f"[AUDIO] Warning: Start time in past by {abs(wait_time):.3f}s")
        
        pygame.mixer.music.play()
        print(f"[AUDIO] Playing at {time.time():.3f}")
    
    def play_now(self):
        """Play immediately (for testing)"""
        pygame.mixer.music.play()
        print("[AUDIO] Playing now")
    
    def stop(self):
        pygame.mixer.music.stop()
        print("[AUDIO] Stopped")
    
    def is_playing(self):
        return pygame.mixer.music.get_busy()

if __name__ == "__main__":
    # Test with a sound file
    player = AudioPlayer()
    
    # Generate test tone if no tracks exist
    import os
    os.makedirs("cube/tracks", exist_ok=True)
    
    print("\nPlace MP3/WAV files in cube/tracks/ directory")
    print("Name them like: track_001.mp3, track_002.mp3")
    print("\nFor now, we'll play the system test sound...")
    
    # Play system sound as test
    pygame.mixer.music.load("/System/Library/Sounds/Hero.aiff")
    pygame.mixer.music.play()
    
    # Wait for it to finish
    while pygame.mixer.music.get_busy():
        time.sleep(0.1)
    
    print("Audio test complete!")
```

**Test it:**
```bash
python cube/audio_player.py
# Should hear system sound
```

### Step 5: Create Simple Server (30 min)
Create `server/app.py`:
```python
"""VibeCube sync server"""
import asyncio
import websockets
import json
import time
from datetime import datetime

# Connected cubes: {cube_id: websocket}
cubes = {}

# Current state: {cube_id: card_data}
cube_states = {}

async def handle_cube(websocket, path):
    cube_id = None
    try:
        async for message in websocket:
            data = json.loads(message)
            event = data.get("event")
            
            if event == "register":
                cube_id = data.get("cube_id")
                cubes[cube_id] = websocket
                cube_states[cube_id] = None
                print(f"[SERVER] Cube {cube_id} connected")
                
                await websocket.send(json.dumps({
                    "event": "registered",
                    "cube_id": cube_id,
                    "message": "Connected to VibeCube server"
                }))
            
            elif event == "card_inserted":
                cube_id = data.get("cube_id")
                card = data.get("card")
                cube_states[cube_id] = card
                
                print(f"[SERVER] {cube_id} inserted: {card['track_name']}")
                
                # Check if both cubes have same card
                await check_sync()
            
            elif event == "card_removed":
                cube_id = data.get("cube_id")
                cube_states[cube_id] = None
                print(f"[SERVER] {cube_id} removed card")
                
                # Notify all cubes
                await broadcast({
                    "event": "partner_disconnected",
                    "cube_id": cube_id
                })
    
    except websockets.exceptions.ConnectionClosed:
        if cube_id:
            print(f"[SERVER] Cube {cube_id} disconnected")
            cubes.pop(cube_id, None)
            cube_states.pop(cube_id, None)

async def check_sync():
    """Check if cubes are ready to sync"""
    # Get all cards
    cards = [card for card in cube_states.values() if card is not None]
    
    if len(cards) < 2:
        return  # Need at least 2 cubes
    
    # Check if all cards match
    first_track = cards[0]["track_id"]
    if all(card["track_id"] == first_track for card in cards):
        print(f"[SERVER] SYNC! All cubes have {cards[0]['track_name']}")
        
        # Calculate sync timestamp (5 seconds from now)
        sync_time = time.time() + 5.0
        
        await broadcast({
            "event": "play_synchronized",
            "track_id": first_track,
            "track_name": cards[0]["track_name"],
            "start_timestamp": sync_time,
            "buffer_duration": 5.0
        })

async def broadcast(message):
    """Send message to all connected cubes"""
    if cubes:
        await asyncio.gather(
            *[ws.send(json.dumps(message)) for ws in cubes.values()],
            return_exceptions=True
        )

async def main():
    print("="*60)
    print("VibeCube Sync Server")
    print("="*60)
    print("Listening on ws://localhost:8000")
    print("Waiting for cubes to connect...")
    print()
    
    async with websockets.serve(handle_cube, "0.0.0.0", 8000):
        await asyncio.Future()  # Run forever

if __name__ == "__main__":
    asyncio.run(main())
```

**Test it:**
```bash
python server/app.py
# Should start server on port 8000
```

### Step 6: Create Cube Client (30 min)
Create `cube/main.py`:
```python
"""VibeCube client - runs on each cube"""
import asyncio
import websockets
import json
import sys
import time
from audio_player import AudioPlayer
from display_mock import MockDisplay
from nfc_mock import MockNFCReader

class VibeCube:
    def __init__(self, cube_id, server_url, use_mock_nfc=True):
        self.cube_id = cube_id
        self.server_url = server_url
        self.ws = None
        
        self.display = MockDisplay()
        self.audio = AudioPlayer()
        self.nfc = MockNFCReader() if use_mock_nfc else None
        
        self.current_card = None
        self.status = "idle"
    
    async def connect(self):
        """Connect to sync server"""
        self.display.show_status("Connecting...")
        
        self.ws = await websockets.connect(self.server_url)
        
        # Register with server
        await self.ws.send(json.dumps({
            "event": "register",
            "cube_id": self.cube_id
        }))
        
        self.display.show_status("Ready!")
        print(f"[{self.cube_id}] Connected to server")
    
    async def handle_server_messages(self):
        """Listen for messages from server"""
        async for message in self.ws:
            data = json.loads(message)
            event = data.get("event")
            
            if event == "registered":
                print(f"[{self.cube_id}] Registered with server")
            
            elif event == "play_synchronized":
                track_id = data["track_id"]
                track_name = data["track_name"]
                start_timestamp = data["start_timestamp"]
                
                self.display.show_text([
                    "SYNCED!",
                    track_name,
                    f"Starting in {int(start_timestamp - time.time())}s"
                ])
                
                # Load and play at timestamp
                if self.audio.load_track(track_id):
                    self.audio.play_at_timestamp(start_timestamp)
                    self.display.show_text([
                        "♪ VIBING ♪",
                        track_name
                    ])
                    self.status = "playing"
            
            elif event == "partner_disconnected":
                self.display.show_status("Partner left")
    
    async def simulate_card_insert(self, track_id, track_name, artist):
        """Simulate inserting NFC card (for testing)"""
        self.current_card = self.nfc.simulate_card_insert(track_id, track_name, artist)
        
        self.display.show_text([
            "Card detected!",
            track_name,
            f"by {artist}"
        ])
        
        # Tell server
        await self.ws.send(json.dumps({
            "event": "card_inserted",
            "cube_id": self.cube_id,
            "card": self.current_card
        }))
        
        self.status = "waiting"
        self.display.show_status("Waiting for partner...")
    
    async def run(self):
        """Main loop"""
        await self.connect()
        
        # Start listening to server
        server_task = asyncio.create_task(self.handle_server_messages())
        
        # Keep running
        await server_task

async def main():
    if len(sys.argv) < 2:
        print("Usage: python cube/main.py <cube_id> [server_url]")
        print("Example: python cube/main.py cube1 ws://localhost:8000")
        sys.exit(1)
    
    cube_id = sys.argv[1]
    server_url = sys.argv[2] if len(sys.argv) > 2 else "ws://localhost:8000"
    
    cube = VibeCube(cube_id, server_url)
    
    try:
        await cube.run()
    except KeyboardInterrupt:
        print(f"\n[{cube_id}] Shutting down...")

if __name__ == "__main__":
    asyncio.run(main())
```

### Step 7: Test Complete System (15 min)

**Terminal 1 - Start Server:**
```bash
cd ~/Playground/VibeCube
source venv/bin/activate
python server/app.py
```

**Terminal 2 - Start Cube 1:**
```bash
cd ~/Playground/VibeCube
source venv/bin/activate
python cube/main.py cube1
```

**Terminal 3 - Start Cube 2:**
```bash
cd ~/Playground/VibeCube
source venv/bin/activate
python cube/main.py cube2
```

**Terminal 4 - Simulate Card Insertion:**
```bash
cd ~/Playground/VibeCube
source venv/bin/activate
python3 << EOF
import asyncio
import websockets
import json

async def insert_card():
    # Connect as cube1
    ws1 = await websockets.connect("ws://localhost:8000")
    await ws1.send(json.dumps({"event": "register", "cube_id": "cube1"}))
    await ws1.recv()
    
    # Insert card on cube1
    await ws1.send(json.dumps({
        "event": "card_inserted",
        "cube_id": "cube1",
        "card": {"track_id": "track_001", "track_name": "Midnight Frequencies", "artist": "DJ_Alex"}
    }))
    
    await asyncio.sleep(2)
    
    # Connect as cube2
    ws2 = await websockets.connect("ws://localhost:8000")
    await ws2.send(json.dumps({"event": "register", "cube_id": "cube2"}))
    await ws2.recv()
    
    # Insert same card on cube2
    await ws2.send(json.dumps({
        "event": "card_inserted",
        "cube_id": "cube2",
        "card": {"track_id": "track_001", "track_name": "Midnight Frequencies", "artist": "DJ_Alex"}
    }))
    
    print("Both cards inserted! Check server output for sync message.")
    await asyncio.sleep(10)

asyncio.run(insert_card())
EOF
```

**✅ If you see "SYNC!" in server output, core functionality works!**

---

## Phase 2: Test on Raspberry Pi 3 B+ (2 hours)

### Step 1: Prepare Pi (30 min)

**Flash Raspberry Pi OS** (if not already):
- Download Raspberry Pi Imager
- Flash "Raspberry Pi OS Lite (64-bit)"
- Enable SSH and WiFi in settings

**SSH to Pi:**
```bash
ssh pi@raspberrypi.local
# Default password: raspberry
```

**Update and install:**
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y python3-pip python3-venv git
sudo apt install -y python3-pygame python3-pil
sudo apt install -y pulseaudio pulseaudio-module-bluetooth bluez

# Enable I2C (for future OLED, harmless now)
sudo raspi-config nonint do_i2c 0
sudo reboot
```

### Step 2: Deploy Code to Pi (15 min)

**On your Mac:**
```bash
cd ~/Playground/VibeCube

# Push to GitHub (if not already)
git add .
git commit -m "Initial prototype"
git push

# Or use rsync
rsync -avz --exclude venv --exclude __pycache__ . pi@raspberrypi.local:~/vibecube/
```

**On Pi:**
```bash
ssh pi@raspberrypi.local
cd ~/vibecube
python3 -m venv venv
source venv/bin/activate
pip install pygame pillow websockets
```

### Step 3: Test Display (30 min)

Your LCD screen - connect it and test. If it's HDMI, you'll see terminal output. If it's GPIO LCD, you need specific drivers.

**For HDMI LCD (easiest):**
Just connect and use terminal output - the mock display will work!

**For GPIO LCD (like 3.5" TFT):**
You'll need to install drivers - but skip this for now, terminal output is fine.

### Step 4: Test Bluetooth Audio (30 min)

```bash
ssh pi@raspberrypi.local

# Start Bluetooth
sudo systemctl start bluetooth
sudo systemctl enable bluetooth

# Pair your Bluetooth speaker (put it in pairing mode first)
bluetoothctl
> power on
> agent on
> default-agent
> scan on
# Wait for your speaker to appear
> pair XX:XX:XX:XX:XX:XX
> trust XX:XX:XX:XX:XX:XX
> connect XX:XX:XX:XX:XX:XX
> exit

# Test audio
paplay /usr/share/sounds/alsa/Front_Center.wav
# Should hear from Bluetooth speaker!
```

### Step 5: Run Cube on Pi (15 min)

**Terminal 1 (Mac) - Server:**
```bash
cd ~/Playground/VibeCube
source venv/bin/activate
python server/app.py
```

**Terminal 2 (Pi) - Cube:**
```bash
ssh pi@raspberrypi.local
cd ~/vibecube
source venv/bin/activate

# Find your Mac's IP
# On Mac: ifconfig | grep "inet " | grep -v 127.0.0.1

python cube/main.py cube1 ws://YOUR_MAC_IP:8000
```

**✅ If cube connects to server, your Pi setup works!**

---

## Phase 3: Two-Cube Simulation (1 hour)

### Test Sync Between Mac and Pi

**Terminal 1 (Mac) - Server:**
```bash
python server/app.py
```

**Terminal 2 (Pi) - Cube 1:**
```bash
python cube/main.py cube1 ws://YOUR_MAC_IP:8000
```

**Terminal 3 (Mac) - Cube 2:**
```bash
python cube/main.py cube2 ws://localhost:8000
```

**Simulate both inserting cards:**
```python
# Use the script from Phase 1, Step 7
```

**✅ If both "cubes" sync and play simultaneously, IT WORKS!**

---

## Success Criteria for Today

By end of day, you should have:
- ✅ Mock NFC, display, and audio working on Mac
- ✅ Server coordinating two mock cubes
- ✅ Code deployed to Pi 3 B+
- ✅ Bluetooth audio working on Pi
- ✅ Pi connecting to server and syncing with Mac

**If all above work: COMMIT TO THE PROJECT!** 🎉

The core sync mechanism is proven. Remaining work is just:
- Order real NFC reader (PN532)
- Order small OLED display
- Order two Pi Zero 2 W (or use Pi 3 B+ if you have two)
- Design and print enclosure

---

## If You Get Stuck

**Can't hear audio on Mac:**
```bash
# Check pygame installation
python3 -c "import pygame; pygame.mixer.init(); print('OK')"
```

**Server won't start:**
```bash
# Check if port 8000 is in use
lsof -i :8000
# Kill process: kill -9 <PID>
```

**Pi won't connect to server:**
```bash
# Test network connectivity
ping YOUR_MAC_IP
# Check firewall on Mac - allow port 8000
```

**Bluetooth won't pair:**
```bash
# Restart Bluetooth service
sudo systemctl restart bluetooth
# Check status
sudo systemctl status bluetooth
```

---

## Quick Reference

**Start everything:**
```bash
# Terminal 1: Server
cd ~/Playground/VibeCube && source venv/bin/activate && python server/app.py

# Terminal 2: Cube 1 (Mac)
cd ~/Playground/VibeCube && source venv/bin/activate && python cube/main.py cube1

# Terminal 3: Cube 2 (Pi)
ssh pi@raspberrypi.local "cd ~/vibecube && source venv/bin/activate && python cube/main.py cube2 ws://YOUR_MAC_IP:8000"
```

**Add test track:**
```bash
# Download any MP3 and name it track_001.mp3
cp ~/Music/some_song.mp3 cube/tracks/track_001.mp3
```

---

## Next Steps After Today

If prototype works:
1. ✅ Order components from TIMELINE.md (~$100)
2. ✅ Follow full setup guide with real NFC hardware
3. ✅ Design cube enclosure
4. ✅ Complete by Feb 15

**You've got a working prototype - the hardest part is done!** 🚀
