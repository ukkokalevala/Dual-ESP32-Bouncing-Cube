Project Description: Dual-ESP32 Bouncing Cube
Overview
A two-board embedded graphics project where a single virtual "cube" bounces around a 256×64 pixel virtual playfield that is physically split across two 128×64 OLED displays, each driven by its own ESP32. The two ESP32s communicate wirelessly via ESP-NOW so that the cube appears to move seamlessly from one screen to the other — as if the two OLEDs were one continuous display.

No motion sensor (MPU) is required. The cube's motion is entirely simulated in software on one board and streamed to the other.

Concept
Picture two small OLED screens sitting side by side:

   ESP32 "A"        ESP32 "B"
   (left half)     (right half)
Together they form a virtual 256×64 canvas:


Virtual coordinate space (0 → 255)
A small square (the "cube") moves diagonally across this space, bouncing off all four virtual walls. When it crosses the boundary at x = 128, it appears to jump from the right edge of screen A to the left edge of screen B — but in reality it's just one continuous animation split into two halves.

Roles of the Two Boards
Board	Role	Responsibility
ESP32-A	Simulator + Left Renderer	Runs the physics, draws the cube when its x is 0–127, broadcasts state every frame
ESP32-B	Right Renderer	Receives state via ESP-NOW, draws the cube when its x is 128–255
This asymmetry is deliberate — it removes any ambiguity about who "owns" the cube at a given moment, eliminating the classic handoff-jitter problem.

Hardware Requirements
Per board (×2):

1× ESP32 development board (any variant with WiFi)

1× SSD1306 OLED display (128×64, I²C)

4× jumper wires (VCC, GND, SDA, SCL)

USB power source

No MPU, no buttons, no extra sensors.

Software Architecture
Communication layer:

ESP-NOW in broadcast mode (FF:FF:FF:FF:FF:FF) — no MAC address discovery needed

A small fixed-size struct is sent every frame (~50 Hz)

Packet format:
c
struct CubePacket {
    float x, y;      // position in virtual 256×64 space
    float vx, vy;    // velocity (for future use / debugging)
    uint8_t magic;   // 0x42, used to reject stray packets
};
Physics loop (Board A only):

Update x and y by velocity each frame

Reverse velocity when hitting any of the four virtual walls

Broadcast the new position

Render the cube if it lies within A's half

Render loop (Board B):

Receive the latest packet

Translate virtual x → local x (subtract 128)

Draw the cube if it falls within B's half

Key Design Decisions
Problem	Solution
Don't want to look up MAC addresses	Use ESP-NOW broadcast FF:FF:FF:FF:FF:FF
Receiver sees its own echo packets	magic byte filter + only A simulates
Cube jitters at screen boundary	A owns physics entirely; B is a passive mirror
Two screens must look like one	Shared virtual coordinate space (0–255)
No motion sensing required	Cube motion is fully software-simulated
What It Demonstrates
ESP-NOW low-latency peer-to-peer wireless on ESP32

Distributed rendering — splitting one logical canvas across two physical devices

Master/slave coordination without MAC discovery or pairing overhead

I²C OLED graphics with the Adafruit GFX library

A foundation for more advanced multi-board effects (Pong, scrolling text, multiplayer games)

Possible Extensions
Trail effects — keep the last N positions and fade them

Multiple cubes — array of CubePacket entries in the broadcast

Spin/rotation animation — swap between a square, diamond, and cross at each bounce

Pong mode — add a paddle on each board, cube becomes the ball

Tilt control — now an MPU6050 would make sense: tilt to nudge gravity

Third screen — expand the virtual canvas to 384 wide with another board

One-Line Summary
Two ESP32 boards, each with an OLED, talk over ESP-NOW broadcast to render one bouncing cube across a shared 256×64 virtual canvas — no MPU, no MAC lookup, no complexit
