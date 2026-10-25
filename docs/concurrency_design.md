# Server Concurrency Architecture Design

## 1. Server Concurrency Model

For terminal trivia, it was chosen that non-blocking I/O using Pythons selectors module. This will allow the server to handle multiple client connections in a single thread wihtout needing seperate threads for each player.

Server uses an event loop to check which client sockets are ready to send or receive data. This prevents the server from getting stuck waiting for one plater while another plater is trying to communicate.

## 2. Connection Lifecycle

1. **Server Startup:** Ther server creates a TCP listening socket, binds it to an address and port, starts listening, and registers the socket with the selector.