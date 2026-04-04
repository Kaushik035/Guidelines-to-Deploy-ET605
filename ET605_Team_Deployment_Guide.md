# ET605 Course Project — Team Deployment Guide

## Server Infrastructure for Mathematics Chapter Websites

**Prepared by:** Merge Team (Kaushik, Khushi, Kabir)
**Server Domain:** kaushik-dev.online
**Date:** April 2026

---

## 1. What We Are Providing You

Each team receives a **fully isolated Linux server** (Ubuntu 22.04) running inside a container on our central server. Think of it as your own private virtual machine where you have full control.

**You will receive:**

- A **.pem file** — this is your private SSH key to log into your server. Guard it carefully. If you lose it or share it publicly, anyone can access your server.
- **Your credentials** — username, SSH port, app port, and your live URL (details in Section 2 below).
- **A live HTTPS URL** — your website will be accessible from anywhere in the world at `https://your-container-name.kaushik-dev.online`.
- **Full root (sudo) access** — you can install any software, database, or framework you need.
- **Isolated environment** — no other team can see or access your files, database, or processes. You cannot affect other teams either.

**You do NOT get:**

- Access to the host machine or any other team's container.
- Ability to change your SSH port or app port — these are fixed by the infrastructure team.

---

## 2. Your Team Credentials

You will receive a credentials sheet with the following information. Here is what each field means:

| Field | Meaning | Example |
|-------|---------|---------|
| Team | Your team name | AMA |
| Topic | The math chapter you are building | Grade 6 Fractions |
| Container | Your server's container name | grade6-fractions |
| Username | Your login username | grade6_fractions |
| SSH Port | The port you use to SSH into your server | 2202 |
| App Port | The port your app MUST listen on | 3002 |
| Key File | Your private key file name | grade6-fractions.pem |
| SSH Command | The full command to connect | ssh -i grade6-fractions.pem grade6_fractions@kaushik-dev.online -p 2202 |
| App URL | Where your website will be live | https://grade6-fractions.kaushik-dev.online |
| Recommendation API | The endpoint to call after chapter completion | POST https://kaushik-dev.online/recommend |

---

## 3. How to Connect to Your Server

### 3.1 — From macOS or Linux

Open your terminal and run:

```bash
chmod 600 your-key-file.pem
ssh -i your-key-file.pem your_username@kaushik-dev.online -p your_ssh_port
```

For example, if you are Team AMA (Grade 6 Fractions):

```bash
chmod 600 grade6-fractions.pem
ssh -i grade6-fractions.pem grade6_fractions@kaushik-dev.online -p 2202
```

The first time you connect, it will ask "Are you sure you want to continue connecting?" — type `yes` and press Enter.

### 3.2 — From Windows (using WSL)

If you have WSL (Windows Subsystem for Linux) installed:

1. Open WSL terminal.
2. Copy your .pem file into WSL (e.g., to your home directory).
3. Run the same SSH command as above.

### 3.3 — From Windows (using PuTTY)

1. Download PuTTY and PuTTYgen from https://www.putty.org.
2. Open PuTTYgen → click "Load" → select your .pem file → click "Save private key" → save as .ppk file.
3. Open PuTTY:
   - Host Name: `kaushik-dev.online`
   - Port: your SSH port (e.g., 2202)
   - Connection → SSH → Auth → Private key file: select your .ppk file
   - Click "Open"

### 3.4 — Verify You Are Connected

Once logged in, you should see a prompt like:

```
grade6_fractions@grade6-fractions:~$
```

You are now inside your own private server. You have full control.

---

## 4. How to Deploy Your Application

### 4.1 — Understand the Key Rule

**YOUR APP MUST LISTEN ON YOUR DESIGNATED APP PORT.** This is the most important rule. If your app port is 3002, your app must start a web server on port 3002. If it listens on any other port, your website will not be accessible from the outside.

When starting your server, always bind to `0.0.0.0` (all interfaces), NOT `localhost` or `127.0.0.1`.

### 4.2 — Step-by-Step Deployment

**Step 1: SSH into your server**

```bash
ssh -i your-key.pem your_username@kaushik-dev.online -p your_ssh_port
```

**Step 2: Install your tech stack**

You have full sudo access. Install whatever you need:

For Node.js:
```bash
sudo apt update
sudo apt install -y nodejs npm
```

For Python:
```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

For Java:
```bash
sudo apt update
sudo apt install -y default-jdk
```

For databases:
```bash
# MongoDB
sudo apt install -y mongodb

# MySQL
sudo apt install -y mysql-server

# PostgreSQL
sudo apt install -y postgresql
```

You can install multiple things. It is your server — install whatever your project needs.

**Step 3: Clone your project**

```bash
git clone https://github.com/your-team/your-project.git
cd your-project
```

If your repo is private, you can use a personal access token:

```bash
git clone https://your-token@github.com/your-team/your-project.git
```

**Step 4: Install dependencies**

For Node.js:
```bash
npm install
```

For Python:
```bash
pip3 install -r requirements.txt
```

**Step 5: Configure your app to listen on the correct port**

This is critical. Your app MUST listen on your designated app port and on 0.0.0.0.

Node.js (Express) example — if your app port is 3002:
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('<h1>Grade 6 Fractions</h1>');
});

// IMPORTANT: Use your designated port and 0.0.0.0
const PORT = 3002;
app.listen(PORT, '0.0.0.0', () => {
  console.log(`Server running on port ${PORT}`);
});
```

Python (Flask) example — if your app port is 3002:
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def home():
    return '<h1>Grade 6 Fractions</h1>'

# IMPORTANT: Use your designated port and 0.0.0.0
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=3002)
```

Python (Django) example:
```bash
python3 manage.py runserver 0.0.0.0:3002
```

Java (Spring Boot) — in application.properties:
```
server.port=3002
server.address=0.0.0.0
```

**Step 6: Run your app**

```bash
node server.js
# or
python3 app.py
# or
java -jar your-app.jar
```

**Step 7: Verify your website is live**

Open a browser on your phone or laptop and visit your App URL:

```
https://your-container-name.kaushik-dev.online
```

You should see your website.

---

## 5. Keeping Your App Running After Disconnecting

When you close the SSH session, your app will stop. To keep it running permanently, use one of these methods:

### Option A — Using pm2 (Recommended for Node.js)

```bash
sudo npm install -g pm2

# Start your app
pm2 start server.js --name "my-app"

# Make it survive reboots
pm2 startup
pm2 save

# Useful pm2 commands
pm2 status          # Check if app is running
pm2 logs            # View app logs
pm2 restart my-app  # Restart app
pm2 stop my-app     # Stop app
```

### Option B — Using screen (Works for any stack)

```bash
# Start a screen session
screen -S myapp

# Run your app inside the screen
node server.js
# or python3 app.py, etc.

# Detach from screen (app keeps running): press Ctrl+A, then D

# Re-attach later to see logs
screen -r myapp
```

### Option C — Using systemd (Most robust)

```bash
sudo nano /etc/systemd/system/myapp.service
```

Paste (adjust paths and commands for your stack):

```
[Unit]
Description=My Chapter App
After=network.target

[Service]
User=your_username
WorkingDirectory=/home/your_username/your-project
ExecStart=/usr/bin/node server.js
Restart=always
RestartSec=5
Environment=PORT=3002

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myapp
sudo systemctl start myapp

# Check status
sudo systemctl status myapp

# View logs
sudo journalctl -u myapp -f
```

---

## 6. Frontend + Backend on a Single Port

Your container has only ONE port exposed to the outside world (your app port). If you have both a frontend and a backend, you must serve them through that single port. Here are two common approaches:

### Approach A — Single Server Serves Everything (Simplest)

Your backend server also serves the frontend files:

Node.js (Express) example:
```javascript
const express = require('express');
const app = express();

// Serve frontend (React build, HTML files, etc.)
app.use(express.static('public'));
// or for React: app.use(express.static('client/build'));

// Backend API routes
app.get('/api/quiz', (req, res) => {
  res.json({ question: "What is 2+2?", options: [3, 4, 5, 6] });
});

app.post('/api/submit', (req, res) => {
  res.json({ score: 85 });
});

// IMPORTANT: Your designated port
app.listen(3002, '0.0.0.0');
```

Python (Flask) example:
```python
from flask import Flask, send_from_directory, jsonify
app = Flask(__name__, static_folder='static')

@app.route('/')
def home():
    return send_from_directory('static', 'index.html')

@app.route('/api/quiz')
def quiz():
    return jsonify({"question": "What is 2+2?"})

app.run(host='0.0.0.0', port=3002)
```

### Approach B — Backend Proxies to Frontend (For separate dev servers)

If you run React dev server on port 3010 and backend on port 3002 during development, for deployment just build your React app and serve the static files from your backend (Approach A).

```bash
# Build your React app
cd client
npm run build

# The build folder now contains static files
# Your Express server serves them with express.static('client/build')
```

---

## 7. Calling the Recommendation API

After a student completes your chapter, your app MUST call our Recommendation API. This is how the unified system knows the student's progress and can recommend the next chapter.

### What to Send

**Endpoint:** `POST https://kaushik-dev.online/recommend`
**Content-Type:** `application/json`

**Example payload:**

```json
{
  "student_id": "student_123",
  "chapter": "grade6_fractions",
  "score": 85,
  "time_spent_minutes": 25,
  "completed": true,
  "timestamp": "2026-04-04T10:30:00Z"
}
```

### How to Call It

Node.js example:
```javascript
const axios = require('axios');

async function sendToRecommendation(studentData) {
  try {
    const response = await axios.post('https://kaushik-dev.online/recommend', {
      student_id: studentData.id,
      chapter: 'grade6_fractions',
      score: studentData.score,
      time_spent_minutes: studentData.timeSpent,
      completed: true,
      timestamp: new Date().toISOString()
    });
    
    console.log('Recommendation:', response.data);
    // Show the recommendation to the student
    return response.data;
  } catch (error) {
    console.error('Error calling recommendation API:', error.message);
  }
}
```

Python example:
```python
import requests
from datetime import datetime

def send_to_recommendation(student_data):
    url = "https://kaushik-dev.online/recommend"
    payload = {
        "student_id": student_data["id"],
        "chapter": "grade6_fractions",
        "score": student_data["score"],
        "time_spent_minutes": student_data["time_spent"],
        "completed": True,
        "timestamp": datetime.now().isoformat()
    }
    
    response = requests.post(url, json=payload)
    recommendation = response.json()
    print("Recommendation:", recommendation)
    return recommendation
```

### When to Call It

Call the Recommendation API when a student finishes your chapter — for example, after they complete the final quiz or exercise. Display the recommendation response to the student so they know which chapter to study next.

---

## 8. Important Rules and Reminders

### DO

- Listen on your designated app port (check your credentials sheet).
- Bind to `0.0.0.0`, NOT `localhost` or `127.0.0.1`.
- Use pm2, screen, or systemd to keep your app running after disconnecting SSH.
- Call the Recommendation API after chapter completion.
- Keep your .pem file safe and private.
- Use `git clone` to deploy your code (easiest way to update later).
- Test your website by visiting your App URL from a browser.

### DO NOT

- Do NOT change your app port — it is fixed and mapped to your URL.
- Do NOT try to access other team's containers or the host machine.
- Do NOT share your .pem key file publicly (e.g., on GitHub).
- Do NOT run your app on localhost/127.0.0.1 — it must be 0.0.0.0.
- Do NOT use ports other than your designated one for external-facing services.
- Do NOT delete system files or modify SSH configuration.

### Common Mistakes

| Mistake | Symptom | Fix |
|---------|---------|-----|
| App listening on wrong port | Website shows 502 Bad Gateway | Change your app to listen on your designated port |
| App bound to localhost | Website shows 502 Bad Gateway | Change to 0.0.0.0 |
| App not running | Website shows 502 Bad Gateway | SSH in and start your app, use pm2 to keep it alive |
| Wrong .pem file permissions | SSH says "permission denied" | Run `chmod 600 your-key.pem` |
| Forgot to install dependencies | App crashes on start | Run `npm install` or `pip install -r requirements.txt` |

---

## 9. Useful Commands Inside Your Container

```bash
# Check if your app is running
ps aux | grep node    # for Node.js
ps aux | grep python  # for Python

# Check what is listening on your port
ss -tlnp | grep YOUR_PORT

# Check disk usage
df -h

# Check memory usage
free -h

# View your app logs (if using pm2)
pm2 logs

# Restart your app (if using pm2)
pm2 restart all

# Update your code from GitHub
cd ~/your-project
git pull origin main
pm2 restart all   # restart after pulling new code
```

---

## 10. How to Update Your Code After Deployment

When you push new code to GitHub and want to update your live website:

```bash
# SSH into your container
ssh -i your-key.pem your_username@kaushik-dev.online -p your_ssh_port

# Go to your project directory
cd ~/your-project

# Pull latest code
git pull origin main

# Install any new dependencies
npm install   # for Node.js
# or
pip3 install -r requirements.txt  # for Python

# Restart your app
pm2 restart all   # if using pm2
# or
sudo systemctl restart myapp   # if using systemd
```

Your website will be updated within seconds.

---

## 11. Getting Help

If you face issues with your server (SSH not connecting, app not accessible, internet not working inside container), contact the Infrastructure Team. Please provide:

1. Your team name and container name
2. The exact error message you see
3. What command you ran
4. Screenshot if possible

**DO NOT** attempt to fix infrastructure-level issues yourself (network, ports, proxy). Contact us and we will fix it.

---

## Quick Reference Card

```
SSH into your server:
  ssh -i YOUR_KEY.pem YOUR_USERNAME@kaushik-dev.online -p YOUR_SSH_PORT

Your website URL:
  https://YOUR_CONTAINER_NAME.kaushik-dev.online

Recommendation API:
  POST https://kaushik-dev.online/recommend

Your app MUST listen on:
  Port: YOUR_APP_PORT
  Host: 0.0.0.0
```
