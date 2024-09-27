 we’ll build a real-time chat application using Socket.io for bi-directional communication, Express.js for the server, React.js for the frontend, and Chakra UI for styling. We'll set up the project using Vite for fast development.

Key Features:

User ID based chat: A random userId is generated and stored in session storage.
Message display: Our messages appear on the left, and others' appear on the right, each with an icon.
Here is the demo of this project :

Socket demo

chat demo

Server socket logs

Let’s get started!

1. Setting Up the Project
Create A wrapper folder chat-app where our front-end and back-end resides:

mkdir chat-app
cd chat-app
Backend Setup: Express.js with Socket.io
First, create a folder for your backend and initialize it with npm init -y:

mkdir chat-backend
cd chat-backend
npm init -y
Now install the necessary packages:

npm install express socket.io
Create the following file structure for the backend:

chat-backend/
│
├── server.js
└── package.json

server.js - Express and Socket.io
Here’s how to set up the Express server with Socket.io:

const express = require('express');
const http = require('http');
const { Server } = require('socket.io');

const app = express();
const server = http.createServer(app);
const io = new Server(server, {
  cors: {
    origin: 'http://localhost:5173',
    methods: ['GET', 'POST']
  }
});

io.on('connection', (socket) => {
  console.log('A user connected:', socket.id);

  socket.on('sendMessage', (message) => {
    io.emit('receiveMessage', message);
  });

  socket.on('disconnect', () => {
    console.log('A user disconnected');
  });
});

server.listen(4000, () => {
  console.log('Server listening on port 4000');
});

This code creates an Express server that listens on port 4000 and sets up Socket.io to handle real-time communication. When a user sends a message (sendMessage event), it's broadcast to all connected clients.

Frontend Setup: Vite with React and Chakra UI

Create a Vite React project and follow below screenshots (create project in chat-app and not in chat-backend):

npm create vite@latest
Creating project

Selecting Reactjs

Selecting Js

Navigate into the project folder and install the necessary dependencies:

cd chat-frontend
npm install socket.io-client @chakra-ui/react @emotion/react @emotion/styled framer-motion
Now, let's create a basic structure for our chat application.

2. Implementing the Chat Frontend
The following will be the structure for the frontend:

chat-frontend/
│
├── src/
│   ├── components/
│   │   └── ChatBox.jsx
│   ├── App.jsx
│   └── main.jsx
├── index.html
└── package.json
App.jsx
In App.jsx, set up Chakra UI and render the ChatBox component:

import { ChakraProvider, Box, Heading } from "@chakra-ui/react";
import ChatBox from "./components/ChatBox";

function App() {
  return (
    <ChakraProvider>
      <Box p={5}>
        <Heading as="h1" mb={6}>
          Real-Time Chat
        </Heading>
        <ChatBox />
      </Box>
    </ChakraProvider>
  );
}

export default App;

ChatBox.jsx
This is where the main chat logic will go. We’ll use Socket.io to listen for messages and handle real-time updates. A random userId is generated and stored in session storage, and the chat UI is built using Chakra UI.

import React, { useState, useEffect } from "react";
import { Box, Input, Button, HStack, VStack, Text, Avatar } from "@chakra-ui/react";
import { io } from "socket.io-client";

const socket = io("http://localhost:4000");

const ChatBox = () => {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const [userId, setUserId] = useState(null);

  useEffect(() => {
    // Generate or get userId from session storage
    let storedUserId = sessionStorage.getItem("userId");
    if (!storedUserId) {
      storedUserId = Math.random().toString(36).substring(7);
      sessionStorage.setItem("userId", storedUserId);
    }
    setUserId(storedUserId);

    // Listen for messages
    socket.on("receiveMessage", (message) => {
      setMessages((prevMessages) => [...prevMessages, message]);
    });

    return () => {
      socket.off("receiveMessage");
    };
  }, []);

  const sendMessage = () => {
    if (input.trim()) {
      const message = {
        userId,
        text: input,
      };
      socket.emit("sendMessage", message);
    //   setMessages((prevMessages) => [...prevMessages, message]);
      setInput("");
    }
  };

  return (
    <VStack spacing={4} align="stretch">
      <Box h="400px" p={4} borderWidth={1} borderRadius="lg" overflowY="auto">
        {messages.map((msg, index) => (
          <HStack key={index} justify={msg.userId === userId ? "flex-start" : "flex-end"}>
            {msg.userId === userId && <Avatar name="Me" />}
            <Box
              bg={msg.userId === userId ? "blue.100" : "green.100"}
              p={3}
              borderRadius="lg"
              maxW="70%"
            >
              <Text>{msg.text}</Text>
            </Box>
            {msg.userId !== userId && <Avatar name="Other" />}
          </HStack>
        ))}
      </Box>

      <HStack>
        <Input
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type a message"
        />
        <Button onClick={sendMessage} colorScheme="teal">
          Send
        </Button>
      </HStack>
    </VStack>
  );
};

export default ChatBox;
How It Works:

Random userId: We generate a random string as userId and store it in sessionStorage. This ensures that even if you reload the page, you'll retain the same userId for the session.

Socket.io for messaging: The app listens for receiveMessage events from the server and displays incoming messages. When a user sends a message, it's emitted via Socket.io, and the UI updates in real-time.

Chakra UI styling: Messages are displayed in a box, with our messages aligned to the left (with a blue background) and others’ to the right (green background). Each message also has an avatar for a simple, personalised chat interface.

3. Running the Application
Start the backend:

cd chat-backend
node server.js
Start the frontend:

cd chat-frontend
npm run dev
Now, open http://localhost:5173 in multiple browser windows to test real-time chat functionality. You’ll see each user's messages displayed with their unique userId.

You’ve successfully built a real-time chat application using Socket.io, Express, React.js, and Chakra UI, with the project set up via Vite! 🎉

This app demonstrates the power of Socket.io for real-time communication 💬 and Chakra UI for a clean, responsive interface 📱.

You can expand this project by adding features like chat rooms 🏠, message persistence 💾, and user authentication 🔒.

That's all for this blog! Stay tuned for more updates and keep building amazing apps! 💻✨
Happy coding! 😊
