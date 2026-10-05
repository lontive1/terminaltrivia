# Game Engine FInite State Machine Specification

'''mermaid
stateDiagram-v2
    [*] --> INIT

    INIT --> WAITING_FOR_PLAYERS : Server starts and begins listening for clients

    WAITING_FOR_PLAYERS --> WAITING_FOR_PLAYERS : Player 1 sends CONNECT / Send LOBBY_WAIT
    WAITING_FOR_PLAYERS --> WAITING_FOR_PLAYERS : Player 2 CONNECT / Assign Player 1 and Player 2 roles
    WAITING_FOR_PLAYERS --> CLEANUP : Connected client disconnects

    GAME_START --> PLAYER_TURN : Send GAME_START and first STATE_UPDATE

    PLAYER_TURN --> EVALUATE_MOVE : Valid MOVE receieved from active player
    PLAYER_TURN --> PLAYER_TURN : Invalid or out of turn MOVE / Send ERROR
    PLAYER_TURN --> GAME_OVER : Client disconnects / Remaining player wins by forfeit

    EVALUATE_MOVE --> PLAYER_TURN : Questions remain / Send over next STATE_UPDATE
    EVALUATE_MOVE --> GAME_OVER : Final question comepleted / send over GAME_OVER

    GAME_OVER --> CLEANUP : Final result delivered

    CLEANUP --> WAITING_FOR_PLAYERS : Reset game and wait for new players
    CLEANUP --> [*] : Server shutdown

'''

##FSM State Description

* **INIT:** The server initializes and begins listening for client connections
* **WAITING_FOR_PLAYERS:** The server waits until the two clients have connected. The first client receives `LOBBY_WAIT`. When the second client connects the server assigns the `Player_1` and `Player_2` roles
* **GAME_START:** The server sends `GAME_START` to both clients and prepares the first trivia question
* **PLAYER_TURN:** The server waits for a `MOVE` from the active player. Then if invalid or out of turn moves makes an `ERROR` without changing the state
* **EVALUATE_MOVE:** The server evaluates the submitted answer and updates the game state
* **GAME_OVER:** The server determines the final result or declares the remaining player the winner by forfeit after a disconnect
* **CLEANUP:** The server closes the game session clears the current game data, and prepares for another game.

## FSM Edge-Case & Failure Handling
* **OutofTurn Move:** If a player submitted a `MOVE` when it is not their turn, the server then sends an `ERROR` and remains in `PLAYER_TURN`

* **Invalid Move:** If a player submits any invalid answer selection or malformed `MOVE`, the server sends an `ERROR` and remains in `PLAYER_TURN` so the player can submit another move.

* **Abrupt Client Disconnect:** If a client disconnects unexpectedly during an active game, including a 0-byte EOF or socket connection failure, the server transitions to `GAME_OVER`. The remaining connected player wins by forfeit.

* **Post-Game Reset:** After `GAME_OVER`, the server transitions to `CLEANUP`, clears the current game information, and returns to `WAITING_FOR_PLAYERS` for the next game.