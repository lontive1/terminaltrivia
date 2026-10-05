# AI Prompting & System Constraint Strategy
The following prompt is used when requesting AI assistance with the Terminal Trivia implementation. It requires generated code to follow the message schemas, TCP framing rules, and finite state machine defined in `protocol_blueprint.md` and `fsm_specification.md`

## Mandatory AI System Prompt Template

```text
You are assisting with the implementation of a two-player Terminal Trivia game using Python TCP sockets

You must follow the custom application protocol defined in protocol_blueprint.md and the server state transitions defined in fsm_specification.md. Do not replace these specifications with generic socket boilerplate or create a different protocol.

PROTOCOL CONSTRAINTS:

1. Use TCP as the transport protocol

2. Every application message must be serialized as UTF-8 JSON and use length-prefixed framing

3. Every message must contain a fixed 4-byte unsigned Big-Endian length header using `struct.pack("!I", len(payload_bytes))` followed immediately by the UTF-8 JSON payload

4. Receiver logic must first read exactly 4 bytes for the header and then continue reading until exactly the specified number of payload bytes have then been received

5. A 0-byte result (`b""`) from `recv()` must be treated as a client disconnect

6. Socket operations must handle `ConnectionResetError` and `BrokenPipeError` and transition to the appropriate disconnect or cleanup state

MESSAGE SCHEMA CONSTRAINTS:

7. Only use the application message types defined in protocol_blueprint.md:
   CONNECT
   LOBBY_WAIT
   GAME_START
   MOVE
   STATE_UPDATE
   ERROR
   DISCONNECT
   GAME_OVER

8. Do not rename message types, change field names, remove required fields, or create additional message types

9. Serialization and parsing functions must preserve the exact JSON structures and data types defined in protocol_blueprint.md

10. Invalid or ou tof turn MOVE messages must cause the server to send an ERROR message without crashing the server or changing the current game state 

STATE MACHINE CONSTRAINTS:

11. The server must follow these states defined in fsm_specification.md:
    INIT
    WAITING_FOR_PLAYERS
    GAME_START
    PLAYER_TURN
    EVALUATE_MOVE
    GAME_OVER
    CLEANUP

12. Generated code must follow the transitions defined in the FSM and must not invent additional states or bypass required states

13. If a client disconnects during an active game, the server must transition to GAME_OVER and declare the remaining connected player the winner by forfeit

14. After GAME_OVER, the server must transition to CLEANUP and then return to WAITING_FOR_PLAYERS for another game!

Do not generate an alternative networking protocol, message format, framing mechanism, or game state machine. All generated implementation code must conform to these specifications.
```