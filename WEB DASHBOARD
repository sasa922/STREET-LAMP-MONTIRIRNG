from flask import Flask, render_template_string, request, redirect
import paho.mqtt.client as mqtt
from threading import Lock
from flask_httpauth import HTTPBasicAuth
from werkzeug.security import generate_password_hash, check_password_hash
import secrets
import time
import socket

app = Flask(_name_)
app.secret_key = secrets.token_hex(16)

auth = HTTPBasicAuth()
users = {
    "admin": generate_password_hash("ChangeThisPassword123!")
}

@auth.verify_password
def verify_password(username, password):
    if username in users and check_password_hash(users.get(username), password):
        return username

# MQTT Setup
broker = "102.46.132.3"
port = 1883
topic_control = "esp32/bruh/control"
topic_status = "esp32/bruh/status"

received_messages = []
message_lock = Lock()
lamp_status = {
    "lamp1": "ok",
    "lamp2": "ok",
    "lamp3": "ok"
}

client = mqtt.Client(mqtt.CallbackAPIVersion.VERSION2)

def on_connect(client, userdata, flags, rc, properties=None):
    if rc == 0:
        print("✅ Connected to MQTT Broker!")
        client.publish(topic_control, "Server connected")
        client.subscribe(topic_status)
    else:
        print(f"❌ Connection failed with code {rc}")

def on_message(client, userdata, msg):
    global received_messages
    message = msg.payload.decode()

    # Update lamp status based on message content
    for i in range(1, 4):
        if f"Lamp {i} broken" in message:
            lamp_status[f"lamp{i}"] = "broken"
        elif f"Lamp {i} working" in message:  # <-- THIS LINE HANDLES IT!
            lamp_status[f"lamp{i}"] = "ok"

    # Add timestamped message to log
    timestamp = time.strftime('%H:%M:%S')
    formatted_msg = f"{timestamp} - {message}"

    with message_lock:
        received_messages.append(formatted_msg)
        if len(received_messages) > 10:
            received_messages.pop(0)


client.on_connect = on_connect
client.on_message = on_message
client.connect(broker, port, 60)
client.loop_start()

@app.route('/')
@auth.login_required
def index():
    with message_lock:
        messages = received_messages.copy()

    local_ip = socket.gethostbyname(socket.gethostname())

    return f"""
    <!DOCTYPE html>
    <html>
    <head>
        <title>Street Lamp Control</title>
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <script>
    setInterval(() => {{
        window.location.reload();
    }}, 5000); // refresh every 5 seconds
</script>

        <style>
            body {{ font-family: Arial, sans-serif; max-width: 800px; margin: 0 auto; padding: 20px; }}
            h1 {{ color: #2c3e50; text-align: center; }}
            .lamp-status {{
                display: flex;
                justify-content: space-around;
                margin: 20px 0;
            }}
            .lamp {{
                padding: 15px;
                text-align: center;
                border-radius: 10px;
                width: 150px;
            }}
            .ok {{ background-color: #3498db; color: white; }}
            .broken {{ background-color: #e74c3c; color: white; }}
            .reset-btn {{ margin-top: 10px; padding: 5px 15px; border: none; border-radius: 5px; cursor: pointer; }}
            .message-log {{
                background: #f0f0f0;
                padding: 15px;
                border-radius: 5px;
                margin-top: 20px;
                height: 300px;
                overflow-y: auto;
            }}
            .message {{ margin: 8px 0; padding: 8px; border-bottom: 1px solid #ddd; }}
            .timestamp {{ color: #7f8c8d; font-size: 0.9em; }}
        </style>
    </head>
    <body>
        <h1>🌐 Street Lamp Monitor</h1>

        <div class="lamp-status">
            <div class="lamp {lamp_status['lamp1']}">
                Lamp 1<br>{lamp_status['lamp1'].upper()}
                <form action="/reset/1" method="post">
                    <button class="reset-btn" type="submit">Reset</button>
                </form>
            </div>
            <div class="lamp {lamp_status['lamp2']}">
                Lamp 2<br>{lamp_status['lamp2'].upper()}
                <form action="/reset/2" method="post">
                    <button class="reset-btn" type="submit">Reset</button>
                </form>
            </div>
            <div class="lamp {lamp_status['lamp3']}">
                Lamp 3<br>{lamp_status['lamp3'].upper()}
                <form action="/reset/3" method="post">
                    <button class="reset-btn" type="submit">Reset</button>
                </form>
            </div>
        </div>

        <div class="message-log">
            <h3>📡 Device Messages:</h3>
            {''.join(f'<div class="message"><span class="timestamp">[{msg.split(" - ")[0]}]</span> {msg.split(" - ")[1]}</div>'
                     for msg in messages) if messages else '<div class="message">No messages yet...</div>'}
        </div>

        <div style="margin-top: 20px; text-align: center;">
            <small>Local Access: http://{local_ip}:5000</small>
        </div>
    </body>
    </html>
    """

@app.route('/reset/<int:lamp_id>', methods=['POST'])
@auth.login_required
def reset_lamp(lamp_id):
    if 1 <= lamp_id <= 3:
        lamp_status[f"lamp{lamp_id}"] = "ok"
    return redirect('/')

if _name_ == '_main_':
    print("""
    🌟 Street Lamp Monitor Started!
    Access:
    1. Local: http://localhost:5000
    2. LAN: http://<your-local-ip>:5000
    3. Username: admin
    4. Password: ChangeThisPassword123!
    """)
    print(f"📡 Your local IP: {socket.gethostbyname(socket.gethostname())}")
    app.run(host="0.0.0.0", port=5000, debug=True)
