# Custom FTC Dashboard Write-up
Write-up/Guide on how to customize the [acmerobotics FTC Dashboard](https://github.com/acmerobotics) without using new libraries or importing code onto the control hub

Concepts covered in this write-up are meant for REV Control Hubs

**DISCLAIMER:** SOME OF THE CONCEPTS SHOWN HERE HAVE NOT BEEN TESTED, THEY ARE THEORETICAL AND SHOULD WORK(I'm too lazy to test them) I ALSO MIGHT'VE MADE MISTAKES WRITING THIS, SO PLEASE LET ME KNOW IF I GOT ANYTHING WRONG

If you need any help, feel free to reach out

# Why Customize Your Dashboard?
Most teams won't need this because the default dashboard is enough, however, sometimes a little customization could go a long way in boosting productivity. Unfortunately(unless I'm stupid), there is no simple way to modify the dashboard and directly compile it onto the bot. This write-up will explain a simple(or complex) workaround to customize the dashboard and do whatever you want with it.

# 1. Explanation
A simple way to do this is to use a simple data relaying method to a self-hosted dashboard. A simple example would go as follows: Bot transmits data to a server/backend on a computer which then relays the data to the self-hosted dashboard. This has been tested using websockets, but there are other ways to go about transmitting the data. 

![Sort of how it goes](https://i.imgur.com/AdF6zWw.png)

# 2. A Faster Way
A faster way to do the same thing done in #1 is to just directly relay all the data from the bot to the dashboard. However, this hasn't been tested yet since it's very hard to debug when it's end to end and hosting the websocket server on either the dashboard or the bot is generally not a good idea.

## Pros and Cons for Each Way
Pros of using a backend server:
- Easier to debug
- Easier to make handler for connections
- Hosted on a more stable platform

Cons of using a backend server:
- Might be a little slower
- More points of failure

Pros of End-End:
- Less latency, more consistent connection
- Less points of failure
- Easier to manage...?

Cons of End-End:
- Very hard to debug due to the nature of each end
- Running a websocket server on either the bot or the dashboard itself isn't a great idea(unstable platform)

# 3. Actually Customizing the Dashboard
To customize the dashboard, it's pretty simple. Download the dashboard source code from [here](https://github.com/acmerobotics/ftc-dashboard)<br> Follow the installation instructions in the README.md to get started. All the code for the actual dashboard is located in the client folder. To customize the site, simply go through the different components of the page and change what you want. You should be able to add/remove components, but I have only tested changing the content due to my limited knowledge of React. You can preview your site by going into the directory and running `yarn dev` After running the command, Vite will create a page on your machine where you can make changes and view them in real time. 

# 4. Deploying and Running
To deploy and run, you will need 3 components(or 2 if you're using end-to-end) <br>
Components: <br>
- Script on the bot that can connect to websockets
- A locally hosted websocket server that serves as a median
- Custom locally hosted dashboard<br>


In order to run the script on the bot, have it embedded or called within a teleop or auto(not tested) class. To run the websocket server, run a simple handler while connected to the bot wifi and bind the websocket to the bots IP `192.168.43.62:PORT` To run the dashboard code, cd into the directory containing the code for the dashboard and run `yarn dev --host` After this, everything should be setup on the bots wifi. 


# 5. Sample Scripts(Mostly tested, still improving reconnection and error handling)

Complete sample script for locally hosted websocket server made in python 3.9.11:
```python
import asyncio
import websockets
import json
import signal

class CodeServer:
    def __init__(self):
        self.clients = set()
        self.should_exit = False

    async def register(self, websocket):
        if websocket not in self.clients:
            self.clients.add(websocket)
            print(f"Added {websocket}")
        
        asyncio.create_task(self.handle_disconnect(websocket)) # await websocket.wait_closed() freezes the program so it's run in the background(Very crucial)
            
    async def handle_disconnect(self, websocket):
        try:
            await websocket.wait_closed()
        finally:
            self.clients.remove(websocket)
            print(f"Removed {websocket}")

    async def broadcast(self, message):
        if self.clients:
            await asyncio.gather(
                *[client.send(message) for client in self.clients]
            )

    async def handle_connection(self, websocket):

        await self.register(websocket) # Register the websocket
        
        try:
            async for message in websocket:
                print(f"Received message {message}")
                await self.broadcast(json.dumps({"code": message}))
        except websockets.exceptions.ConnectionClosedOK:
            print("Disconnected")
        except Exception as e:
            raise e

    def handle_signal(self, signum, frame):
        self.should_exit = True
        print("\nShutting down server...")

async def main():
    server = CodeServer()
    signal.signal(signal.SIGINT, server.handle_signal)
    
    # Create the WebSocket server with handler
    async with websockets.serve(server.handle_connection, "192.168.43.62", 8765):
        print("WebSocket server started on ws://192.168.43.62:8765")
        
        # No idea
        while not server.should_exit:
            
            # Code breaks for me if this isn't there
            await asyncio.sleep(1)

if __name__ == "__main__":
    asyncio.run(main())
```

Incomplete snippet, meant for receiving text from the websocket and updating the text box on the custom dashboard

```javascript
import React, { useEffect, useState, useCallback} from 'react';
const RECONNECT_INTERVAL = 3000;

const [codeContent, setCodeContent] = useState([]);
  const [socket, setSocket] = useState(null);
  const [isConnected, setIsConnected] = useState(false);

  const connectWebSocket = useCallback(() => {
    try {
      const ws = new WebSocket('ws://192.168.43.62:8765');
      // ws://192.168.43.62:8765

      ws.onopen = () => {
        console.log('Connected to WebSocket server');
        setIsConnected(true);
      };

      ws.onmessage = (event) => {
        try {
          const data = JSON.parse(event.data);
          if (data.code) {
            setCodeContent(data.code.split('\n'));
          }
        } catch (error) {
          console.error('Error parsing WebSocket message:', error);
        }
      };

      ws.onerror = (error) => {
        console.error('WebSocket error:', error);
      };

      ws.onclose = () => {
        console.log('WebSocket connection closed');
        setIsConnected(false);
        setSocket(null);
        
        // Reconnect
        setTimeout(connectWebSocket, RECONNECT_INTERVAL);
      };

      setSocket(ws);
    } catch (error) {
      console.error('Error creating WebSocket:', error);
      // Reconnect
      setTimeout(connectWebSocket, RECONNECT_INTERVAL);
    }
  }, []);

  useEffect(() => {
    connectWebSocket();

    // Cleanup
    return () => {
      if (socket) {
        socket.close();
      }
    };
  }, [connectWebSocket]);
```
**WARNING: THE CODE ABOVE IS INCOMPLETE AND MIGHT NOT WORK AS A DIRECT COPY AND PASTE**

Almost complete websocket code for the bot
```java
try {
    webSocket = new WebSocketClient(new URI("ws://192.168.43.62:8765")) {
        @Override
        public void onOpen(ServerHandshake handshakedata) {
            telemetry.addData("WebSocket", "Connected");
            telemetry.update();
        }

        @Override
        public void onMessage(String message) {
            telemetry.addData("WebSocket Received", message);
            telemetry.update();
        }

        @Override
        public void onClose(int code, String reason, boolean remote) {
            telemetry.addData("WebSocket", "Connection closed: " + reason);
            telemetry.update();
        }

        @Override
        public void onError(Exception ex) {
            telemetry.addData("WebSocket Error", ex.getMessage());
            telemetry.update();
        }
    };

    webSocket.connect();
} catch (URISyntaxException e) {
    telemetry.addData("WebSocket Error", "Invalid URI: " + e.getMessage());
    telemetry.update();
}
```
This code was used before `waitForStart();` in teleop. To send data in the teleop loop, use the following code
```java
if (webSocket != null && webSocket.isOpen()) {
    webSocket.send(DATA GOES HERE);
    telemetry.addData("WebSocket", DATA VARIABLE GOES HERE); // This isn't necessary, it's just for debugging
} else {
    telemetry.addData("WebSocket", "Not connected");
}
```



# Some Other Things
Since websockets can send in bytes, you can do a lot more than just sending text. Data from images can be converted into bytes, which can then be sent to the backend where it'll be decoded and written in a easily decodeable format and sent to the frontend to be displayed. On a MacBook M3 Pro using wscat for simulating the bot, 183.9 MB of data was sent to the backend in around 29.1 milliseconds.

## This is still WIP
