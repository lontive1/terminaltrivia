#App proptocol blueprint

##Transport later and packet framing mech

**Transport Protocol** TCP
**Data Serialization Format:** Structured JSON
* **Framing Rule:** Length-Prefixed Framing. Every serialized message payload is preceded by a fixed 4-byte unsigned integer in Network Byte Order (Big-Endian `!I`) that specifies the exact byte length of the payload.


### On wire Byte Stream COninous Layout Ex

```text
[4-Byte Length][{"msg_type":"CONNECT","player_id":"Player_1","timestamp":1728000000}][4-Byte Length][{"msg_type":"LOBBY_WAIT","message":"Waiting for Player 2","timestamp":1728000005}]
```

###Receiver Extraction Logic

1. The receiver reads exactly 4 bytes from the TCP stream for the header message
2. If the header returns empty the connection is treated as closed
3. The header is unpacked using `struct.unpack("!I", header_bytes)[0]` to be able to determine payload length N
4. The receiver is continuing to read untill exactly N bytes have been collected
5. The payload is decoded from UTP8 and parsed as JSON


## 2. App message Types and Explicit JSON Schemas

### 1. CONNECT Client to Server


**Purpose:** Client requests to join the game with a player alias

```json
{
  "msg_type": "CONNECT",
  "player_id": "Player_1",
  "timestamp": 1728000000
}
```

* `msg_type`: String
* `player_id`: String
* `timestamp`: Integer

### 2. LOBBY_WAIT Server to Client

**Purpose:** Server notifies the frist client that its. waiting for Player 2 to connect

```json
{
  "msg_type": "LOBBY_WAIT",
  "message": "Waiting for Player 2",
  "timestamp": 1728000005
}
```

* `msg_type`: String
* `message`: String
* `timestamp`: Integer

### 3. GAME_START Server to Clients

**Purpose:** Server notifies both clients that the game has satared and assigns player 1 and player 2 their respective roles

```json
{
  "msg_type": "GAME_START",
  "assigned_role": "Player_1",
  "total_players": 2,
  "timestamp": 1728000010
}
```

* `msg_type`: String
* `assigned_role`: String
* `total_players`: Integer
* `timestamp`: Integer

### 4. MOVE Client to Server

**Purpose** A player submits an asnwer selection for the trivia question that is showed

```json
{
  "msg_type": "MOVE",
  "player_id": "Player_1",
  "payload": {
    "question_id": 1,
    "choice": "B"
  },
  "timestamp": 1728000020
}
```

* `msg_type`: String
* `player_id`: String
* `payload`: Object
* `payload.question_id`: Integer
* `payload.choice`: String
* `timestamp`: Integer

###5. STATE_UPDATE Server to Clients

**Purpose** SEvrer broadcasts the current question, asnwer options, scores, and the active player.

```json
{
  "msg_type": "STATE_UPDATE",
  "payload": {
    "question_id": 2,
    "question_text": "What protocol operates at the Transport Layer?",
    "options": {
      "A": "IP",
      "B": "TCP",
      "C": "HTTP",
      "D": "Ethernet"
    },
    "scores": {
      "Player_1": 10,
      "Player_2": 0
    },
    "active_player": "Player_2"
  },
  "timestamp": 1728000025
}
```

* `msg_type`: String
* `payload`: Object
* `payload.question_id`: Integer
* `payload.question_text`: String
* `payload.options`: Object
* `payload.scores`: Object
* `payload.active_player`: String
* `timestamp`: Integer

### 6. ERROR Server to Client

**Purpose** Server notifies a client of an invalid move, out of turn move, or malformed message

```json
{
  "msg_type": "ERROR",
  "error_code": "INVALID_SELECTION",
  "details": "Selection must be A, B, C, or D.",
  "timestamp": 1728000030
}
```

* `msg_type`: String
* `error_code`: String
* `details`: String
* `timestamp`: Integer


### 7. DISCONNECT CLient to Server

**Purpose** Client notifies the sever of an intentantional departure from the game

```json
{
  "msg_type": "DISCONNECT",
  "player_id": "Player_1",
  "reason": "User quit application.",
  "timestamp": 1728000040
}
```

* `msg_type`: String
* `player_id`: String
* `reason`: String
* `timestamp`: Integer

### 8. GAME_OVER Server to Clients

**Purpose** Server Broadcasts the final game outcome and final scores

```json
{
  "msg_type": "GAME_OVER",
  "outcome": {
    "winner": "Player_1",
    "reason": "Final question completed",
    "final_scores": {
      "Player_1": 40,
      "Player_2": 20
    }
  },
  "timestamp": 1728000050
}
```

* `msg_type`: String
* `outcome`: Object
* `outcome.winner`: String
* `outcome.reason`: String
* `outcome.final_scores`: Object
* `timestamp`: Integer

### 3. Connection termination and scoket lifecucle managemnet

### Applicatino Disconnectiion

When a player intentionally exits, the client sends a structured `DISCONNECT` message before closing its socket. The server can then notify the remaining player, declare a win by forfeit if the game is active, and close the socket

A normal socket closure causes TCP to perform its FIN teardown.

### TCP EOF (0-Byte) Handling

When a remote peer closes its connection cleanly, `recv()` returns zero bytes (`b""`). The receive loop checks for this condition:

```python
if not data:
    break
```

The server treats this as a client disconnection and performs the appropriate game state transition and socket cleanup

### Abrupt Connection Termination

If a client crashes, is forcefully terminated, or experiences a network failure, the connection may terminate without a normal TCP FIN teardown.

Socket operations handle the following exceptions:

* **`ConnectionResetError`:** The remote peer forcibly closed or reset the connection.
* **`BrokenPipeError`:** The application attempted to send data to a socket whose remote endpoint was already closed.

When either condition occurs, the server handles the client as disconnected, then transitions to the appropriate game state, and performs socket cleanup