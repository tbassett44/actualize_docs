---
title: 📡 Actualize Peer BLE Link
excerpt: Cross-platform P2P data negotiation over Bluetooth Low Energy
deprecated: false
hidden: false
metadata:
  title: ''
  description: ''
  robots: index
next:
  description: ''
---
# BTLE Peer Module Documentation

The `btle_peer.js` module provides Bluetooth Low Energy (BLE) peer-to-peer connection functionality for device discovery, pairing, and communication within the connection discovery system.

## 🔍 Overview

The **Actualize Peer BLE Link** module (`modules.btle_peer`) enables two mobile devices (Android or iOS) to discover, connect, and exchange small messages directly over **Bluetooth Low Energy (BLE)** — without relying on internet connectivity or servers.

Each device acts simultaneously as both a **Central (scanner)** and a **Peripheral (advertiser)**, allowing any two nearby devices to detect each other and negotiate a connection dynamically.

This dual-role architecture mirrors how systems like **Apple AirDrop**, **Google Nearby Share**, and **GAEN** proximity frameworks use BLE: for **discovery, handshake, and short data exchange**, with optional escalation to higher-bandwidth channels later.

This module enables devices to:

- Discover nearby BLE-enabled devices
- Establish peer-to-peer connections
- Exchange data between connected devices
- Manage connection states and handle disconnections
- Implement secure pairing protocols

## Module Structure

### Core Components

The BTLE Peer module is typically structured as follows:

```javascript
window.modules.btle_peer = {
    // Core functionality
    init: function() { /* Initialize BLE adapter */ },
    scan: function() { /* Scan for nearby devices */ },
    connect: function(deviceId) { /* Connect to specific device */ },
    disconnect: function(deviceId) { /* Disconnect from device */ },
    
    // Data exchange
    send: function(deviceId, data) { /* Send data to peer */ },
    broadcast: function(data) { /* Broadcast to all connected peers */ },
    
    // Event handlers
    onDeviceDiscovered: function(device) { /* Handle new device */ },
    onConnectionEstablished: function(device) { /* Handle connection */ },
    onDataReceived: function(device, data) { /* Handle incoming data */ },
    onConnectionLost: function(device) { /* Handle disconnection */ },
    
    // State management
    getConnectedDevices: function() { /* Return connected devices */ },
    getDeviceInfo: function(deviceId) { /* Get device details */ },
    isConnected: function(deviceId) { /* Check connection status */ }
};
```

## API Reference

### Initialization

#### `init(options)`

Initializes the BLE adapter and sets up the peer connection system.

**Parameters:**

- `options` (Object): Configuration options
  - `scanInterval` (Number): Scan interval in milliseconds (default: 5000)
  - `connectionTimeout` (Number): Connection timeout in milliseconds (default: 10000)
  - `maxConnections` (Number): Maximum simultaneous connections (default: 5)
  - `serviceUUID` (String): BLE service UUID for the application
  - `characteristicUUID` (String): BLE characteristic UUID for data exchange

**Example:**

```javascript
modules.btle_peer.init({
    scanInterval: 3000,
    connectionTimeout: 15000,
    maxConnections: 8,
    serviceUUID: '12345678-1234-1234-1234-123456789abc',
    characteristicUUID: '87654321-4321-4321-4321-cba987654321'
});
```

### Device Discovery

#### `startScanning()`

Begins scanning for nearby BLE devices.

**Returns:** `Promise<void>`

**Example:**

```javascript
modules.btle_peer.startScanning()
    .then(() => console.log('Scanning started'))
    .catch(error => console.error('Scan failed:', error));
```

#### `stopScanning()`

Stops the current scanning process.

**Returns:** `Promise<void>`

#### `getDiscoveredDevices()`

Returns an array of discovered devices.

**Returns:** `Array<Device>`

**Device Object Structure:**

```javascript
{
    id: 'device-uuid',
    name: 'Device Name',
    rssi: -45,           // Signal strength
    distance: 2.5,       // Estimated distance in meters
    lastSeen: Date,      // Last discovery timestamp
    services: [],        // Available BLE services
    isConnectable: true, // Whether device accepts connections
    metadata: {}         // Additional device information
}
```

### Connection Management

#### `connect(deviceId, options)`

Establishes a connection to a specific device.

**Parameters:**

- `deviceId` (String): Target device identifier
- `options` (Object): Connection options
  - `timeout` (Number): Connection timeout override
  - `retries` (Number): Number of retry attempts (default: 3)
  - `secure` (Boolean): Use secure pairing (default: true)

**Returns:** `Promise<Connection>`

**Example:**

```javascript
modules.btle_peer.connect('device-123', {
    timeout: 20000,
    retries: 5,
    secure: true
}).then(connection => {
    console.log('Connected to:', connection.device.name);
}).catch(error => {
    console.error('Connection failed:', error);
});
```

#### `disconnect(deviceId)`

Disconnects from a specific device.

**Parameters:**

- `deviceId` (String): Device to disconnect from

**Returns:** `Promise<void>`

#### `disconnectAll()`

Disconnects from all connected devices.

**Returns:** `Promise<void>`

### Data Exchange

#### `send(deviceId, data, options)`

Sends data to a connected device.

**Parameters:**

- `deviceId` (String): Target device identifier
- `data` (Object|String|ArrayBuffer): Data to send
- `options` (Object): Send options
  - `reliable` (Boolean): Ensure delivery (default: true)
  - `timeout` (Number): Send timeout in milliseconds
  - `priority` (String): Message priority ('high', 'normal', 'low')

**Returns:** `Promise<void>`

**Example:**

```javascript
modules.btle_peer.send('device-123', {
    type: 'message',
    content: 'Hello from peer!',
    timestamp: Date.now()
}, {
    reliable: true,
    priority: 'high'
});
```

#### `broadcast(data, options)`

Broadcasts data to all connected devices.

**Parameters:**

- `data` (Object|String|ArrayBuffer): Data to broadcast
- `options` (Object): Broadcast options
  - `excludeDevices` (Array): Device IDs to exclude
  - `reliable` (Boolean): Ensure delivery to all devices

**Returns:** `Promise<Array<Result>>`

### Event Handling

#### `on(event, callback)`

Registers an event listener.

**Events:**

- `deviceDiscovered`: New device found during scanning
- `deviceLost`: Previously discovered device no longer visible
- `connectionEstablished`: Successfully connected to a device
- `connectionFailed`: Failed to connect to a device
- `connectionLost`: Lost connection to a device
- `dataReceived`: Received data from a peer
- `dataSent`: Successfully sent data to a peer
- `error`: General error occurred

**Example:**

```javascript
modules.btle_peer.on('deviceDiscovered', function(device) {
    console.log('Found device:', device.name, 'at', device.distance + 'm');
    
    // Auto-connect to devices with specific name pattern
    if (device.name.startsWith('MyApp-')) {
        modules.btle_peer.connect(device.id);
    }
});

modules.btle_peer.on('dataReceived', function(device, data) {
    console.log('Received from', device.name + ':', data);
    
    // Handle different message types
    switch (data.type) {
        case 'chat':
            displayChatMessage(device, data.message);
            break;
        case 'file':
            handleFileTransfer(device, data);
            break;
        case 'ping':
            modules.btle_peer.send(device.id, { type: 'pong' });
            break;
    }
});
```

## Connection States

### Device States

- `discovered`: Device found during scanning
- `connecting`: Connection attempt in progress
- `connected`: Successfully connected and ready for data exchange
- `pairing`: Secure pairing process in progress
- `paired`: Secure pairing completed
- `disconnecting`: Disconnection in progress
- `disconnected`: No longer connected
- `error`: Connection error occurred

### Connection Lifecycle

```javascript
// 1. Discovery
modules.btle_peer.startScanning();

// 2. Connection
modules.btle_peer.on('deviceDiscovered', (device) => {
    if (shouldConnectTo(device)) {
        modules.btle_peer.connect(device.id);
    }
});

// 3. Data Exchange
modules.btle_peer.on('connectionEstablished', (device) => {
    // Send initial handshake
    modules.btle_peer.send(device.id, {
        type: 'handshake',
        version: '1.0',
        capabilities: ['chat', 'file-transfer']
    });
});

// 4. Cleanup
modules.btle_peer.on('connectionLost', (device) => {
    console.log('Lost connection to:', device.name);
    // Attempt reconnection if needed
    setTimeout(() => {
        modules.btle_peer.connect(device.id);
    }, 5000);
});
```

## Security Considerations

### Pairing and Authentication

- Uses BLE secure pairing protocols
- Implements device authentication
- Supports encryption for data transmission
- Validates device certificates when available

### Data Protection

- Encrypts sensitive data before transmission
- Implements message integrity checks
- Supports secure key exchange
- Validates incoming data for security threats

## Error Handling

### Common Error Types

- `BLUETOOTH_DISABLED`: Bluetooth is turned off
- `PERMISSION_DENIED`: Missing BLE permissions
- `DEVICE_NOT_FOUND`: Target device not discoverable
- `CONNECTION_TIMEOUT`: Connection attempt timed out
- `PAIRING_FAILED`: Secure pairing unsuccessful
- `DATA_TRANSMISSION_FAILED`: Failed to send/receive data
- `UNSUPPORTED_DEVICE`: Device doesn't support required features

### Error Handling Example

```javascript
modules.btle_peer.on('error', function(error) {
    switch (error.code) {
        case 'BLUETOOTH_DISABLED':
            showBluetoothEnablePrompt();
            break;
        case 'PERMISSION_DENIED':
            requestBluetoothPermissions();
            break;
        case 'CONNECTION_TIMEOUT':
            retryConnection(error.deviceId);
            break;
        default:
            console.error('BTLE Error:', error);
            showErrorMessage(error.message);
    }
});
```

## Platform Support

### Mobile Platforms

- **iOS**: Requires iOS 10+ with Core Bluetooth framework
- **Android**: Requires Android 5.0+ with BLE support
- **Permissions**: Location and Bluetooth permissions required

### Web Platforms

- **Chrome**: Web Bluetooth API support (Chrome 56+)
- **Edge**: Limited Web Bluetooth support
- **Safari**: No Web Bluetooth support (iOS/macOS)
- **Firefox**: Experimental Web Bluetooth support

## Integration with Connection Discovery

The BTLE Peer module integrates with the connection discovery system:

```javascript
// Initialize within connection discovery view
modules.btle_peer.init({
    serviceUUID: app.config.btle.serviceUUID,
    onDeviceDiscovered: function(device) {
        // Update UI with discovered device
        connectionDiscovery.addDiscoveredDevice(device);
    },
    onConnectionEstablished: function(device) {
        // Update connection status in UI
        connectionDiscovery.updateDeviceStatus(device.id, 'connected');
    }
});

// Start discovery when view loads
modules.btle_peer.startScanning();
```

## Performance Considerations

### Optimization Tips

- Limit scanning frequency to preserve battery
- Use connection pooling for multiple devices
- Implement data compression for large transfers
- Cache device information to reduce discovery overhead
- Use appropriate MTU sizes for data transmission

### Battery Management

- Stop scanning when not needed
- Implement connection keep-alive mechanisms
- Use low-power BLE modes when available
- Optimize data transmission frequency

## Troubleshooting

### Common Issues

1. **Devices not discovered**: Check Bluetooth permissions and range
2. **Connection failures**: Verify device compatibility and signal strength
3. **Data transmission errors**: Check MTU size and connection stability
4. **High battery usage**: Optimize scanning intervals and connection management

### Debug Mode

```javascript
modules.btle_peer.setDebugMode(true);
// Enables detailed logging for troubleshooting
```

The BTLE Peer module provides a robust foundation for Bluetooth Low Energy peer-to-peer communication, enabling seamless device discovery and data exchange in mobile and web applications.

***

## 🧩 Core Concepts

| Role           | BLE Function                                                                                                                                           | Description                                                         |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| **Peripheral** | Advertises presence and exposes a GATT service with three characteristics: `RX` (write), `TX` (notify), `ID` (read).                                   | Other devices connect to this and send data.                        |
| **Central**    | Scans for peers advertising the shared service UUID, connects, discovers the GATT service, subscribes to notifications (`TX`), and writes data (`RX`). | Acts as client when initiating.                                     |
| **Dual Mode**  | Every device is both advertiser and scanner at once.                                                                                                   | This enables symmetric discovery — either device can connect first. |

***

## 🧠 Data Flow Summary

**1️⃣ Discovery**

- Each device advertises a service UUID and optional short name.
- Each device scans continuously for others advertising the same UUID.

**2️⃣ Connection & Negotiation**

- When one device detects another:

  - It connects as a Central.
  - It discovers the GATT service and reads the **ID characteristic**.
  - If the peer’s `nodeId` matches its own → disconnect (self-advert).
  - Otherwise → establish the link.

**3️⃣ Messaging**

- **Central → Peripheral:** Write data to the peer’s `RX` characteristic.
- **Peripheral → Central:** Notify subscribed centrals on the `TX` characteristic.
- Messages are framed with a 2-byte length header and chunked to respect MTU (~180 bytes typical).

**4️⃣ Teardown**

- Either side can unsubscribe or disconnect.
- The peripheral removes unsubscribed addresses from its peer map.
- Both sides may continue advertising/scanning for new peers.

***

## 🧱 GATT Configuration

| Characteristic | UUID Example                           | Direction                     | Purpose                                               |
| -------------- | -------------------------------------- | ----------------------------- | ----------------------------------------------------- |
| **RX**         | `6e400002-b5a3-f393-e0a9-e50e24dcca9e` | Central → Peripheral          | Incoming messages (`write` / `writeWithoutResponse`). |
| **TX**         | `6e400003-b5a3-f393-e0a9-e50e24dcca9e` | Peripheral → Central          | Outgoing messages (`notify`).                         |
| **ID**         | `6e400004-b5a3-f393-e0a9-e50e24dcca9e` | Central → Peripheral (`read`) | Unique node identifier (persistent per install).      |

### Peripheral setup:

```js
await bluetoothle.addService(res, rej, {
  service: SERVICE,
  characteristics: [
    // RX (write)
    { uuid: RX, permissions:{write:true}, properties:{write:true, writeWithoutResponse:true} },
    // TX (notify)
    { uuid: TX, permissions:{read:true}, properties:{notify:true} },
    // ID (read-only)
    { uuid: ID_UUID, permissions:{read:true}, properties:{read:true} }
  ]
});
```

### ID characteristic read response:

```js
if (evt.status === "readRequested" && evt.characteristicUuid === ID_UUID) {
  bluetoothle.respond(() => {}, () => {}, {
    address: evt.address,
    requestId: evt.requestId,
    status: bluetoothle.OPERATION_SUCCESS,
    value: bluetoothle.bytesToEncodedString(MY_NODE_ID_BYTES)
  });
}
```

***

## 🔑 Node Identity & Self-Detection

Each install generates a persistent 16-byte `nodeId`, stored locally.  
This prevents self-connections and helps uniquely identify peers across sessions.

**Algorithm:**

- On first launch, generate `crypto.getRandomValues(16)` → hex string → store in `localStorage`.
- When connecting to a peer, read its `ID` characteristic.
- If peer’s `nodeId` === your own → disconnect (self-advert).

***

## 🔄 Lifecycle Events

| Event                  | Description                                  | Module Hook                                            |
| ---------------------- | -------------------------------------------- | ------------------------------------------------------ |
| `initializePeripheral` | BLE peripheral ready, service may be added.  | internal boot sequence                                 |
| `writeRequested`       | Central wrote data to RX characteristic.     | triggers `opts.onMessage({role:"central→peripheral"})` |
| `readRequested`        | Central read ID characteristic.              | respond with nodeId                                    |
| `subscribed`           | Central subscribed to TX characteristic.     | add peer to map                                        |
| `unsubscribed`         | Central unsubscribed from TX characteristic. | remove peer + cleanup                                  |
| `connect()`            | Central initiated connection to peer.        | triggers `opts.onConnect()`                            |
| `disconnect()`         | Either side ended session.                   | triggers `opts.onDisconnect()`                         |

***

## 📬 Sending Data

### Central → Peripheral

```js
peer.sendTo(address, { type: "ping", time: Date.now() ));
```

### Peripheral → Central

```js
peer.send({ type: "pong", time: Date.now() });
```

### Broadcast to all

```js
peer.send({ kind: "announcement", msg: "hello neighbors" });
```

The library automatically:

- Encodes the payload (UTF-8)
- Frames it with 2-byte length header
- Splits into MTU-safe chunks (~180 B)
- Reassembles on the receiver side

***

## 🧹 Stop / Cleanup

```js
await peer.stop();
```

This will:

1. Stop scanning and advertising
2. Unsubscribe, disconnect, and close all peer connections
3. Clear the internal `peers` map

Ensures clean restarts without “Device previously connected” errors.

***

## 🧭 Debugging Checklist

✅ Both devices log:

- `Advertising started…`
- `Scan started…`
- `Found peer…`
- `connect OK…`
- `subscribe OK…`
- `📩 From Peripheral…` or `📩 From Central…`

⚠️ If no peers are found:

- Relax UUID filtering in scan
- Ensure `bluetoothle.initialize({ request:true })` completed
- Verify permissions (`BLUETOOTH_SCAN`, `ACCESS_FINE_LOCATION`)
- Ensure BT is enabled and screen is on (iOS background throttles BLE)

***

## 🧠 Example Flow Diagram (text-based)

```
 ┌───────────────────────────────┐
 │  Device A (dual role)         │
 │  Advertise SERVICE            │
 │  Scan for SERVICE             │
 └───────────────┬───────────────┘
                 │
                 ▼
         (BLE Advertisement seen)
                 │
                 ▼
 ┌───────────────────────────────┐
 │  Device B connects as Central │
 │  Discover SERVICE             │
 │  Read ID → compare nodeId     │
 │  Subscribe TX / Write RX      │
 └───────────────┬───────────────┘
                 │
                 ▼
          Bidirectional Messaging
   (Central→RX)   (Peripheral→TX notify)
                 │
                 ▼
       Disconnect / Unsubscribe / Stop
```

***

## 🧩 Next Steps / Roadmap

- [ ] Optional message signing (ECDSA over nodeId)
- [ ] Advertise short public key hash via manufacturer data (requires native plugin)
- [ ] WebRTC or Wi-Fi Direct escalation channel after handshake
- [ ] Multi-peer mesh relaying
- [ ] Integrate into Actualize App social layer for “proximity connection”
  - [ ] Friending
  - [ ] Financial Payments?