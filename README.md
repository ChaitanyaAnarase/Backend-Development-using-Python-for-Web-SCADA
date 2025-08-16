# Backend-Development-using-Python-for-Web-SCADA
The backend of our Web-Based SCADA system is developed using Python, Flask, industrial communication protocols, and SQL database integration. It acts as the core engine, linking PLCs with the frontend dashboard while ensuring reliable monitoring, control, and data storage.

1. Project Structure
SCADA_Backend/
│
├── app/
│   ├── __init__.py          # Flask app initialization
│   ├── routes.py            # API endpoints for frontend
│   ├── modbus_client.py     # Modbus TCP communication
│   ├── opcua_client.py      # OPC UA communication
│   ├── models.py            # Data models (if required)
│   └── utils.py             # Helper functions
│
├── templates/               # HTML templates (if using Flask rendering)
├── static/                  # CSS, JS, images (if needed)
├── requirements.txt         # Python dependencies
├── config.py                # Configuration (IP addresses, ports)
└── run.py                   # Entry point to start Flask server

2. Python Dependencies (requirements.txt)
Flask==2.3.3
pymodbus==3.2.0
freeopcua==0.99.18
flask-cors==3.1.1

3. Flask App Initialization (__init__.py)
from flask import Flask
from flask_cors import CORS

def create_app():
    app = Flask(__name__)
    CORS(app)  # Enable cross-origin requests for frontend
    
    # Register routes
    from .routes import scada_routes
    app.register_blueprint(scada_routes)
    
    return app

4. API Routes (routes.py)
from flask import Blueprint, jsonify, request
from .modbus_client import read_modbus, write_modbus
from .opcua_client import read_opcua, write_opcua

scada_routes = Blueprint('scada', __name__)

# Modbus API
@scada_routes.route('/modbus/read', methods=['GET'])
def modbus_read():
    address = int(request.args.get('address', 0))
    value = read_modbus(address)
    return jsonify({"address": address, "value": value})

@scada_routes.route('/modbus/write', methods=['POST'])
def modbus_write():
    data = request.json
    address = data['address']
    value = data['value']
    write_modbus(address, value)
    return jsonify({"status": "success"})

# OPC UA API
@scada_routes.route('/opcua/read', methods=['GET'])
def opcua_read():
    node_id = request.args.get('node_id')
    value = read_opcua(node_id)
    return jsonify({"node_id": node_id, "value": value})

@scada_routes.route('/opcua/write', methods=['POST'])
def opcua_write():
    data = request.json
    node_id = data['node_id']
    value = data['value']
    write_opcua(node_id, value)
    return jsonify({"status": "success"})

5. Modbus Client (modbus_client.py)
from pymodbus.client.sync import ModbusTcpClient

MODBUS_IP = "192.168.1.10"
MODBUS_PORT = 502

client = ModbusTcpClient(MODBUS_IP, port=MODBUS_PORT)

def read_modbus(address, count=1):
    rr = client.read_holding_registers(address, count)
    return rr.registers[0] if rr.isError() == False else None

def write_modbus(address, value):
    client.write_register(address, value)

6. OPC UA Client (opcua_client.py)
from opcua import Client

OPCUA_URL = "opc.tcp://192.168.1.20:4840"
client = Client(OPCUA_URL)
client.connect()

def read_opcua(node_id):
    node = client.get_node(node_id)
    return node.get_value()

def write_opcua(node_id, value):
    node = client.get_node(node_id)
    node.set_value(value)

7. Run Flask Server (run.py)
from app import create_app

app = create_app()

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000, debug=True)

8. GitHub Repository Tips

Initialize repo:

git init
git add .
git commit -m "Initial SCADA backend commit"


Add .gitignore for Python:

__pycache__/
*.pyc
.env

