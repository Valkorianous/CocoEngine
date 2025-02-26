# Dynamic Networking System – Documentation

A **lightweight** and **dynamic** approach to Roblox networking that uses just **two RemoteEvents** for **client ↔ server** communication and **MessagingService** for **server ↔ server**. This system allows you to **bind** event handlers and **fire** events by name at runtime—no need for predefined remote objects or function tables.

---

## System Overview

- **Single Initialization**:  
  - **Server** calls `Network:Init()` once (e.g., in `ServerScriptService`).  
  - **Client** calls `Network:Init()` once (e.g., in `StarterPlayerScripts`).  
- **Universal Event Names**:  
  - You “bind” callbacks for a string `eventName`.  
  - You “fire” the same `eventName` from the other side (client or server).  
- **Server ↔ Server**:  
  - Publishing and subscribing to cross-server events via Roblox `MessagingService`.  

> **Note**: Always perform **security checks** on the server. Malicious clients can fire any event name with any parameters.

---

## Functions

### 1. `Network:Init()`
**Context**: Server or Client  
**Parameters**: *(none)*  
**Description**:  
- Initializes the networking system.  
- **Server**: Creates (or finds) a folder in `ReplicatedStorage` to store two RemoteEvents (`ClientToServer` & `ServerToClient`) and subscribes to `MessagingService`.  
- **Client**: Waits for these RemoteEvents to appear and sets up a universal handler for server→client messages.  
**Notes**:  
- Call it **once** per environment (server or client). Additional calls are ignored.

---

### 2. `Network:BindEvent(eventName, callback)`
**Context**: Server or Client  
**Parameters**:  
- `eventName` (**string**): The name of the event to handle.  
- `callback` (**function**): The function that runs when this event is received.  

**Behavior**:  
- **On the Server**: The callback signature is `function(player, ...)`, receiving the `Player` who fired the event plus any extra parameters. Triggered when a client calls `FireServer(eventName, ...)`.  
- **On the Client**: The callback signature is `function(...)`, receiving whatever data the server sends. Triggered when the server calls `FireClient` or `FireAllClients` with the same event name.

---

### 3. `Network:UnbindEvent(eventName)`
**Context**: Server or Client  
**Parameters**:  
- `eventName` (**string**): The event name to unbind.  
**Description**:  
- Removes the callback previously registered for `eventName`.  
- Calling it when no callback exists does nothing.

---

### 4. `Network:FireServer(eventName, ...)`
**Context**: **Client-Only**  
**Parameters**:  
- `eventName` (**string**): Name of the event to send to the server.  
- `...`: Additional arguments sent to the server’s callback.  

**Description**:  
- Fires a client→server event.  
- On the **server**, the corresponding `BindEvent` callback runs, receiving `(player, ...)`—where `player` is the sender.

---

### 5. `Network:FireClient(eventName, player, ...)`
**Context**: **Server-Only**  
**Parameters**:  
- `eventName` (**string**): Name of the event for the target client.  
- `player` (**Player**): The specific player to receive the message.  
- `...`: Extra parameters delivered to that client’s callback.  

**Description**:  
- Fires a server→client event for a single `player`.  
- The client’s `BindEvent(eventName, callback)` is called, receiving `(...)`.

---

### 6. `Network:FireAllClients(eventName, ...)`
**Context**: **Server-Only**  
**Parameters**:  
- `eventName` (**string**): Name of the event for all connected clients.  
- `...`: Extra parameters broadcast to every client callback.  

**Description**:  
- Fires a server→client event for **all** players.  
- Each client’s `BindEvent(eventName, callback)` runs with the provided parameters.

---

### 7. `Network:BindGlobalEvent(eventName, callback)`
**Context**: **Server-Only**  
**Parameters**:  
- `eventName` (**string**): Name of the global event to handle.  
- `callback` (**function**): Runs when this server receives a cross-server event.  

**Description**:  
- Subscribes to `MessagingService` for cross-server events.  
- When any server calls `FireGlobalEvent` with `eventName`, the bound callback is invoked on **all** servers that have also bound `eventName`.  
- Callback signature: `function(data)`

---

### 8. `Network:UnbindGlobalEvent(eventName)`
**Context**: **Server-Only**  
**Parameters**:  
- `eventName` (**string**): The global event name to unbind.  

**Description**:  
- Unsubscribes from the specified `eventName` on this server.  
- Future messages for that event name will no longer trigger a callback.

---

### 9. `Network:FireGlobalEvent(eventName, data)`
**Context**: **Server-Only**  
**Parameters**:  
- `eventName` (**string**): The global event name to publish.  
- `data` (**any**): Data sent to all subscribed servers.  

**Description**:  
- Publishes to `MessagingService` on a single “NetworkGlobal” topic.  
- Any other servers that have bound the same `eventName` with `BindGlobalEvent` will receive `data`.

---

## Overview of Typical Usage

1. **Initialize** in **server** and **client**:
   - `Network:Init()` in `ServerScriptService`
   - `Network:Init()` in `StarterPlayerScripts`
2. **On the Server**, use `BindEvent("MyEvent", function(player, ...) ...)` to handle client calls, and `FireClient/FireAllClients("MyEvent", ...)` to notify clients.
3. **On the Client**, use `BindEvent("MyEvent", function(...) ...)` to react to server calls, and `FireServer("MyEvent", ...)` to send data to the server.
4. **For cross-server events**, `BindGlobalEvent("MyGlobalEvent", function(data) ...)` on every server you want listening, and call `FireGlobalEvent("MyGlobalEvent", data)` on any server to broadcast the message.

---

**End of Documentation**
