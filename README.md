# dragon-ball-remote-play
Dragon Ball Remote Play is a C++ project enabling remote 2-player co-op for the latest Dragon Ball game. Players can share controller input online, simulating local multiplayer gameplay from different locations. The system leverages networking and input handling to create a smooth co-op experience over the internet.


To build the project your friend envisions, the goal is to create a system where two players can share controller input remotely, enabling them to play a co-op game together. Here’s a high-level approach to implement the remote play feature using C++:

### Key Components:
1. **Networking Layer (Socket Communication):**
   - To send and receive controller inputs between two players remotely, you'll need a client-server architecture using TCP or UDP sockets. 
   - TCP ensures reliable communication but may introduce some latency. UDP can be faster but may drop packets, so you'll need to choose depending on your latency vs. reliability needs.

2. **Controller Input Capture:**
   - Capture player input from the local machine using input libraries like SDL2 (Simple DirectMedia Layer) or platform-specific APIs (e.g., `XInput` for Xbox controllers or `DirectInput`).
   - Send this input over the network to the remote machine.

3. **Synchronization:**
   - Ensure inputs are synchronized between the two players by sending input data (controller state) in real time. 
   - Handle network latency by potentially introducing predictive algorithms or lag compensation.

4. **Integration with the Game:**
   - For the remote player’s input to control the game, you need to simulate the controller input on the host machine. This could be done using libraries like SDL2 or system-level hooks to inject input.
   
### High-Level Architecture:
- **Client-Side:**
  - Capture local controller input.
  - Send captured input to the host using the networking layer.
  - Receive updates from the host (e.g., the game state) to keep both sides in sync.

- **Host-Side:**
  - Capture and process both local and remote inputs.
  - Simulate remote player’s input on the local machine.
  - Send updates back to the remote player to synchronize game state (if needed).

### Libraries and Tools:
1. **Networking:** 
   - Use `Boost.Asio` for handling socket communication in C++ (supports TCP and UDP).
   
2. **Controller Input:**
   - `SDL2` is a powerful cross-platform library that can handle controller input.
   - Alternatively, on Windows, you can use `XInput` for Xbox controllers or `DirectInput` for a broader range of controllers.

3. **Game Input Simulation:**
   - Use `SDL2` or a platform-specific API to inject simulated input based on remote player commands.

4. **Multithreading:**
   - You will likely need multithreading to handle the real-time nature of controller input capture and network communication simultaneously. C++'s `std::thread` or `Boost.Thread` can help with this.

### Sample Code Snippet:

1. **Networking Setup:**
```cpp
#include <iostream>
#include <boost/asio.hpp>

using boost::asio::ip::tcp;

void start_server() {
    try {
        boost::asio::io_context io_context;
        tcp::acceptor acceptor(io_context, tcp::endpoint(tcp::v4(), 12345));

        std::cout << "Server running on port 12345" << std::endl;

        for (;;) {
            tcp::socket socket(io_context);
            acceptor.accept(socket);

            std::string message = "Welcome to Remote Play";
            boost::asio::write(socket, boost::asio::buffer(message));

            // Here you would read and send controller inputs back and forth
        }
    } catch (std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }
}
```

2. **Controller Input Capture Using SDL2:**
```cpp
#include <SDL2/SDL.h>
#include <iostream>

void capture_input() {
    if (SDL_Init(SDL_INIT_GAMECONTROLLER) < 0) {
        std::cerr << "SDL could not initialize! SDL_Error: " << SDL_GetError() << std::endl;
        return;
    }

    SDL_GameController* controller = SDL_GameControllerOpen(0);
    if (controller == nullptr) {
        std::cerr << "Could not open game controller! SDL_Error: " << SDL_GetError() << std::endl;
        return;
    }

    SDL_Event e;
    while (true) {
        while (SDL_PollEvent(&e)) {
            if (e.type == SDL_CONTROLLERBUTTONDOWN) {
                std::cout << "Button pressed: " << (int)e.cbutton.button << std::endl;
            }
        }
    }

    SDL_GameControllerClose(controller);
    SDL_Quit();
}
```

3. **Simulating Input:**
   - If you're on Windows, you can use `SendInput()` from the WinAPI to simulate game controller inputs for the remote player.

### Development Steps:
1. **Set up Networking:**
   - Use `Boost.Asio` to establish a connection between two machines for transmitting input data.

2. **Implement Controller Input Capture:**
   - Use SDL2 or platform-specific APIs to capture the local controller’s state and send it to the remote player.

3. **Input Simulation on the Host:**
   - Simulate the remote player’s input on the local machine using either SDL2 or a system-specific API to control the game.

4. **Handle Latency:**
   - You’ll need to measure and handle network latency. Depending on the quality of the network, input lag might occur, so consider adding buffer mechanisms or compensating for lag.

5. **Test and Debug:**
   - Run the system and test it with different networks to ensure smooth remote play.

This should give your friend a solid foundation to build a remote play system for 2-player co-op. Would you like any further details or refinements on any part of this project?
