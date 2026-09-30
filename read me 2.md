Skip to content
IoT-practise
Flask-IoT
Repository navigation
Code
Issues
Pull requests
Agents
Actions
Projects
Security and quality
Insights
Settings
IoT-practise
Flask-IoT
Public
Go to file
t
T
Willxxx7
Willxxx7
2nd commit
23079d6
 · 
2 weeks ago
Name		
.gitignore
1st commit of this IoT demo
2 weeks ago
ReadME.md
added a readme
2 weeks ago
Readme2.md
2nd commit
2 weeks ago
app.py
1st commit of this IoT demo
2 weeks ago
fake_device.py
1st commit of this IoT demo
2 weeks ago
requirements.txt
1st commit of this IoT demo
2 weeks ago
Repository files navigation
README
🌐 Flask-IoT
A live IoT dashboard built with Flask. Devices (or a fake device) send sensor data over HTTP; a web page shows it live and lets you toggle an LED.

You can open the dashboard on your phone while your laptop runs the server.

📦 What's in here
File	Purpose
app.py	The Flask server — dashboard + REST API
fake_device.py	Simulates a device (no hardware needed)
requirements.txt	Python dependencies
.gitignore	Files Git should ignore
🚀 Quick Start
1. Install dependencies
pip install -r requirements.txt
2. Find your laptop's IP
You need this so your phone (or a Pi) can reach the server.

Windows: ipconfig → look for IPv4 Address (e.g. 10.101.5.130)
Mac/Linux: ifconfig | grep inet
3. Start the server
python app.py
You should see:

 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:5000
 * Running on http://10.101.5.130:5000
That 0.0.0.0 is doing the magic — the server is now reachable from any device on the same WiFi.

4. Open the dashboard
On your laptop: http://127.0.0.1:5000
On your phone: http://<YOUR-IP>:5000 (e.g. http://10.101.5.130:5000)
You'll see "device offline" and -- for temperature. That's expected. No device is sending data yet.

5. Simulate a device
Open fake_device.py and set SERVER to your laptop's IP:

SERVER = "http://10.101.5.130:5000"   # <-- change this
Then, in a second terminal:

python fake_device.py
Refresh your phone. 🌡️ Live temperature, updating every 2 seconds.

Tap ON or OFF — the server logs each tap in the terminal.

🔌 API Reference
Method	Endpoint	Purpose
GET	/	The dashboard (HTML)
POST	/api/sensor	Device sends{"temp": 23.4}
GET	/api/latest	Latest reading + online status
GET	/api/led	Device reads current LED state
GET	/api/led/on	Turn LED ON
GET	/api/led/off	Turn LED OFF
Try them from a terminal:

curl http://127.0.0.1:5000/api/latest
curl -X POST http://127.0.0.1:5000/api/sensor -H "Content-Type: application/json" -d "{\"temp\": 42}"
curl http://127.0.0.1:5000/api/led/on
⚠️ Working in this repo — READ THIS
Never push directly to main. It's protected. Always work on your own branch:

git clone https://github.com/IoT-practise/Flask-IoT.git
cd Flask-IoT
git checkout -b yourname-experiments     # e.g. alice-experiments

# ... edit, break, fix ...

git add .
git commit -m "what you did"
git push origin yourname-experiments
Then open a Pull Request on GitHub and merge it yourself (0 approvals required).

This keeps everyone's experiments from stepping on each other.

🍓 Running on a Raspberry Pi
The Pi is a full Linux computer, so the same code runs. Replace the fake device with this script (device_pi.py) to use real GPIO:

import requests, time, random
from gpiozero import LED

SERVER = "http://10.101.5.130:5000"   # your server's IP
LED_PIN = 17

led = LED(LED_PIN)

print(f"Connected to {SERVER}, LED on GPIO {LED_PIN}")

while True:
    # 1. Send sensor reading
    temp = 20 + random.random() * 10   # replace with a real sensor later
    try:
        requests.post(f"{SERVER}/api/sensor", json={"temp": temp}, timeout=2)
        print(f"sent {temp:.1f}°C")
    except Exception as e:
        print(f"send error: {e}")

    # 2. Read LED command
    try:
        r = requests.get(f"{SERVER}/api/led", timeout=2)
        if r.json()["led"] == "on":
            led.on()
        else:
            led.off()
    except Exception as e:
        pass

    time.sleep(2)
Wiring the LED
GPIO 17 (pin 11) ── 220Ω resistor ── LED long leg (+)
GND     (pin 9)  ─────────────────── LED short leg (−)
gpiozero is pre-installed on Raspberry Pi OS. If not:

sudo apt install python3-gpiozero
Pi 5 note: if you get GPIO errors, install lgpio first:

sudo apt install python3-lgpio
🧪 Challenges
Work through these in order. Each one teaches a different idea.

Live chart. Add a small line chart to the dashboard showing the last 20 readings. (Hint: readings is already a list on the server.)
Second sensor. Add a humidity field to /api/sensor and display it on the dashboard.
Response time. Tap ON. How long until the LED changes? Now measure it properly with time.time() on both ends. Why is there a delay?
Offline detection. Kill fake_device.py. How long until the dashboard says "device offline"? Where in app.py is that decided? Change it to 30 seconds.
Concurrency. Remove threaded=True from app.py. Restart the server. Open the dashboard in two browser tabs at once. What happens? Why?
Multiple devices. Run fake_device.py three times with different temperatures. What does the dashboard show? Should it show all three? Design a fix.
Persistence. Stop the server. Start it again. Are the readings still there? Why not? Modify app.py to save readings to a SQLite file.
🤔 The Big Question
Why does the LED take 1–2 seconds to respond?

This is polling. The server doesn't push anything to the device — the device has to ask "any change?" every second. And the browser asks /api/latest every 2 seconds. Everyone is repeatedly asking.

It works. It's simple. But it's slow, and it wastes bandwidth.

Next class: we'll fix this with MQTT + WebSockets — where the server pushes to devices and browsers instantly. You'll feel the difference the moment you tap the button.

Once you've felt polling's limits, the question "why do people use Node.js for IoT?" answers itself.

🆘 Troubleshooting
"My phone can't reach the dashboard."

Is your phone on the same WiFi as the laptop? (Not cellular.)
Did you use your laptop's IP, not 127.0.0.1?
Is your firewall blocking port 5000? On Windows, allow Python through.
"device offline" won't go away.

Check SERVER inside fake_device.py — it must match your laptop's IP.
Is fake_device.py actually running in another terminal?
"Address already in use."

Something is already on port 5000. Kill it, or change port=5000 in app.py.
"ModuleNotFoundError: No module named 'flask'."

Run pip install -r requirements.txt. If you're in a venv, activate it first.
📚 What you're learning
Flask routing — turning URLs into functions
0.0.0.0 binding — making a server reachable on the network
REST APIs — clean ways to send structured data (POST/GET + JSON)
threaded=True — what happens when two clients hit a server at once
Polling — and why it doesn't scale
GPIO on a Pi — real hardware from Python
Built for an intro IoT class. If you break it, that's the point — open a PR.

About
for IoT

Resources
Readme
Activity
Custom properties
Stars
3 stars
Watchers
1 watching
Forks
0 forks
Report repository
Releases
No releases published
Create a new release
Packages
No packages published
Publish your first package
Contributors
1
 (1)
@Willxxx7
Willxxx7
Languages
Python
100%
Suggested workflows
Based on your tech stack

Python package logo
Python package
Create and test a Python package on multiple Python versions.
By GitHub Actions
Python application logo
Python application
Create and test a Python application.
By GitHub Actions
Python Package using Anaconda logo
Python Package using Anaconda
Create and test a Python package on multiple Python versions using Anaconda for package management.
By GitHub Actions
More workflows
Footer
© 2026 GitHub, Inc.
Footer navigation
Terms
Privacy
Security
Status
Community
Docs
Contact
Manage cookies
Do not share my personal information
 
