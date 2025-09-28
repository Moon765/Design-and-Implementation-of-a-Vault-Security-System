<!-- README for: Design and Implementation of a Vault Security System -->
<!-- Tip: rename this file to README.md if you prefer Markdown, but GitHub also renders HTML. -->

<h1 align="center">🔐 Design & Implementation of a Vault Security System</h1>
<p align="center">
  <em>Multi-factor vault security with Face Recognition (OpenCV), Fingerprint, Password (4×4 Keypad) & RFID on Raspberry Pi</em><br/>
  <strong>Authors:</strong> Md. Mahadi Hasan Moon · Md. Mahidul Islam · Debashish Kumar Ghosh · Md. Aurongo Jeb · Nafiz Ahmed Chisty
</p>

<p align="center">
  <a href="https://github.com/Moon765/Design-and-Implementation-of-a-Vault-Security-System/blob/main/Project%20book.pdf">📄 View the full paper (PDF)</a> ·
  <a href="#features">Features</a> ·
  <a href="#architecture">Architecture</a> ·
  <a href="#hardware">Hardware</a> ·
  <a href="#wiring">Wiring</a> ·
  <a href="#install">Install</a> ·
  <a href="#run">Run</a> ·
  <a href="#how-it-works">How it works</a> ·
  <a href="#results--limitations">Results & Limitations</a> ·
  <a href="#references">References</a>
</p>

<hr/>

<h2 id="features">✨ Features</h2>
<ul>
  <li><strong>Four-step authentication</strong> (must pass all, no skipping): Face → Fingerprint → Password → RFID.</li>
  <li><strong>Real-time face detection/recognition</strong> with OpenCV (Haar cascades), multi-face capable.</li>
  <li><strong>Two-process design</strong>: one process dedicated to continuous video/recognition; another governs sensor/auth flows.</li>
  <li><strong>Remote monitoring</strong> over LAN (e.g., VNC) with live overlay of recognized names.</li>
  <li><strong>Actuation</strong>: relay-driven electronic door lock; buzzer & LEDs for user feedback.</li>
</ul>

<h2 id="architecture">🧭 System Architecture</h2>
<pre>
┌─────────────┐              ┌────────────────────────────────────┐
│  Camera     │──► frames ──►│  Process A: Face (OpenCV, Python)  │
└─────────────┘              │  • Live detection/recognition      │
                             │  • Publishes identity token        │
                             └────────────┬───────────────────────┘
                                          │ (i2c/IPC)
                                          ▼
                             ┌────────────────────────────────────┐
                             │  Process B: Auth Orchestrator      │
                             │  • Reads identity token            │
                             │  • Fingerprint → Password → RFID   │
                             │  • Drives LCD, buzzer, LEDs        │
                             │  • Unlock via relay on success     │
                             └────────────────────────────────────┘
</pre>

<h2 id="hardware">🧰 Bill of Materials (Core)</h2>
<ul>
  <li>Raspberry Pi 3 Model B (onboard Wi-Fi/Bluetooth, 40-pin GPIO)</li>
  <li>Raspberry Pi Camera Module V2 (Sony IMX219)</li>
  <li>R30X Fingerprint Sensor (via USB-TTL/UART bridge)</li>
  <li>MFRC522 RFID Reader (13.56 MHz, SPI)</li>
  <li>4×4 Matrix Keypad (8-wire row/column)</li>
  <li>20×4 Character LCD with I²C backpack</li>
  <li>Relay module + 12 V electronic door lock, buzzer, LEDs, jumper wires, 12 V Li-Po for lock</li>
</ul>

<h2 id="wiring">🧵 Wiring Overview (example)</h2>
<table>
  <thead><tr><th>Module</th><th>Raspberry Pi</th><th>Notes</th></tr></thead>
  <tbody>
    <tr><td>Camera V2</td><td>CSI connector</td><td>Enable camera in raspi-config</td></tr>
    <tr><td>MFRC522 RFID</td><td>SPI (SCK/MOSI/MISO/CS), 3.3 V, GND</td><td>Use <code>spidev</code>, enable SPI</td></tr>
    <tr><td>Fingerprint (R30X)</td><td>USB → PL2303 → UART (TX/RX), 5 V</td><td>Vendor libraries/serial</td></tr>
    <tr><td>4×4 Keypad</td><td>8× GPIO</td><td>Rows as outputs (low), columns as inputs (pull-up)</td></tr>
    <tr><td>20×4 LCD (I²C)</td><td>SDA/SCL, 5 V, GND</td><td>PCF8574 backpack</td></tr>
    <tr><td>Relay → Door lock</td><td>GPIO (IN), 5 V, GND</td><td>Flyback protection; separate 12 V supply for lock</td></tr>
    <tr><td>Buzzer/LEDs</td><td>GPIO + resistor</td><td>Simple status/alerts</td></tr>
  </tbody>
</table>

<h2 id="install">💻 Install & Setup</h2>
<ol>
  <li>Update OS & enable interfaces:
    <pre><code>sudo apt update &amp;&amp; sudo apt upgrade -y
sudo raspi-config    # enable Camera, SPI, I2C</code></pre>
  </li>
  <li>Python environment & dependencies (examples):
    <pre><code>sudo apt install -y python3-venv python3-pip libatlas-base-dev
python3 -m venv .venv
source .venv/bin/activate
pip install opencv-python numpy pillow imutils RPi.GPIO spidev smbus2
# plus your fingerprint and MFRC522 libraries</code></pre>
  </li>
  <li>Clone repository & place the paper:
    <pre><code>git clone &lt;this-repo&gt;
cp "Design and Implementation of a Vault Security.pdf" ./</code></pre>
  </li>
</ol>

<h2 id="run">▶️ Running (two-process model)</h2>
<ol>
  <li><strong>Process A – Face service:</strong> starts camera, recognizes faces, emits identity tokens (e.g., <code>alpha</code>=unknown, <code>beta</code>=David).
    <pre><code>python face_service.py  # publishes tokens via i2c/IPC</code></pre>
  </li>
  <li><strong>Process B – Auth controller:</strong> waits for token; if known → Fingerprint → Password → RFID; drives LCD/buzzer/relay.
    <pre><code>python auth_controller.py</code></pre>
  </li>
</ol>

<h2 id="how-it-works">🔄 How It Works</h2>
<ol>
  <li><strong>Face Recognition (OpenCV):</strong> Haar Cascade classifier detects faces; recognizer maps to enrolled identities. If unknown, buzzer alerts and flow resets; if known, proceed. (Multi-face capable.)</li>
  <li><strong>Fingerprint:</strong> R30X matches minutiae patterns against the user’s enrolled template. On success, proceed.</li>
  <li><strong>Password:</strong> User enters PIN via 4×4 keypad; controller debounces and verifies.</li>
  <li><strong>RFID:</strong> MFRC522 reads tag UID via SPI; compares against the user’s assigned card.</li>
  <li><strong>Actuation:</strong> If <em>all</em> checks pass, relay energizes the 12 V electronic lock to open; otherwise system resets to Step 1.</li>
</ol>

<h3>Why two processes?</h3>
<p>Continuous, low-latency video/recognition mustn’t be interrupted by sensor logic. The face service streams frames and publishes a compact token. The auth controller remains responsive to user I/O and ensures deterministic multi-factor sequencing.</p>

<h2 id="results--limitations">📊 Results & Limitations</h2>
<ul>
  <li><strong>Results:</strong> Real-time recognition with live overlay; successful sequential verification and door control; remote viewing over VNC.</li>
  <li><strong>Known limitations:</strong> Night-mode face detection not implemented; entry counting not enforced; surveillance stream may exhibit real-time delay under load.</li>
</ul>

<h2 id="security-notes">🔒 Security Notes</h2>
<ul>
  <li>Use secure storage for templates, PINs, and UID mappings; avoid plaintext where possible.</li>
  <li>Isolate power domains (Pi vs. lock 12 V). Add hardware interlocks and flyback diodes.</li>
  <li>For production, consider liveness detection, tamper sensors, encrypted comms, and audit logs.</li>
</ul>

<h2 id="repo-structure">📁 Suggested Repo Structure</h2>
<pre><code>.
├─ face_service.py
├─ auth_controller.py
├─ hardware/
│  ├─ wiring-diagrams/
│  └─ mfrc522/, r30x/, keypad/, lcd/
├─ models/
│  ├─ haarcascades/
│  └─ trained_faces/
├─ data/
│  └─ logs/
├─ Design and Implementation of a Vault Security.pdf
└─ README.html
</code></pre>

<h2 id="references">📚 References</h2>
<p>For full methodology, figures, and citations, see the PDF: <a href="https://github.com/Moon765/Design-and-Implementation-of-a-Vault-Security-System/blob/main/Project%20book.pdf">Design and Implementation of a Vault Security System</a>.</p>

<hr/>

<h2 id="citation">How to Cite</h2>
<p>
Moon, M. M. H., Islam, M. M., Ghosh, D. K., Jeb, M. A., &amp; Chisty, N. A.
<strong>Design and Implementation of a Vault Security System.</strong>
(See repository PDF for full details and venue.)
</p>

<p align="center">— Built for education, prototyping, and research —</p>
