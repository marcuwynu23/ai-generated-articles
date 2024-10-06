
# Building an Extendable WebRTC and Socket.IO Class Library for Video and Voice Calls

WebRTC is a powerful technology that enables peer-to-peer media (audio, video, and data) communication between browsers. Using WebRTC, you can build real-time video and voice applications, while `Socket.IO` handles signaling for the connection setup. One common challenge when working with WebRTC is managing the connection lifecycle, which involves ICE candidates, signaling offers and answers, and handling media streams.

In this article, we’ll walk through how to create an **abstract WebRTC class** using JavaScript, which can be extended to handle both **video and voice calls**. The goal is to create a reusable library that simplifies WebRTC integration, leaving room for customization.

## What We’ll Build

1. An abstract `WebRTCConnection` class that manages the core WebRTC setup.
2. Extendable classes like `VideoCallConnection` and `VoiceCallConnection` that implement specific behaviors for video and voice calls.
3. Signaling with `Socket.IO` to exchange offers, answers, and ICE candidates between peers.

## Why Use an Abstract Class?

In object-oriented programming, **abstract classes** allow you to define a blueprint for other classes. You define the common methods in the abstract class, while requiring the child classes to implement specific details. This approach helps manage the repetitive logic of WebRTC, leaving room for custom implementations based on the type of call (video or voice).

## Step 1: Abstract WebRTCConnection Class

The `WebRTCConnection` class handles the core functionality for setting up a WebRTC peer connection. It includes methods for establishing connections, sending offers, handling ICE candidates, and managing media streams. Since it’s an abstract class, the child classes will implement specific logic for handling video and audio streams.

```javascript
"use strict";
import { io } from "socket.io-client";

class WebRTCConnection {
    constructor(config, signalingServerUrl) {
        if (new.target === WebRTCConnection) {
            throw new TypeError("Cannot construct WebRTCConnection instances directly");
        }

        this.config = config;
        this.signalingServerUrl = signalingServerUrl;
        this.socket = io(this.signalingServerUrl);
        this.pc = null;
        this.localStream = null;
        this.clientId = null;

        // Establish Socket.IO connection
        this.socket.on("connect", () => {
            console.log("Socket.IO connection established");
            this.onReady(); // Call the abstract onReady method
        });

        // Handle incoming signaling messages
        this.socket.on("signaling", (data) => {
            console.log("Received signaling message:", data);
            switch (data.type) {
                case "offer":
                    this.onOffer(data);
                    break;
                case "answer":
                    this.onAnswer(data);
                    break;
                case "candidate":
                    this.onCandidate(data);
                    break;
                case "bye":
                    this.onClose();
                    break;
                default:
                    console.log("Unhandled signaling type:", data.type);
            }
        });

        this.socket.on("disconnect", () => {
            console.log("Socket.IO disconnected");
            this.onClose();
        });
    }

    // Abstract methods to be implemented by subclasses
    onReady() {
        throw new Error("Method 'onReady' must be implemented.");
    }

    onOffer(offer) {
        throw new Error("Method 'onOffer' must be implemented.");
    }

    onAnswer(answer) {
        throw new Error("Method 'onAnswer' must be implemented.");
    }

    onCandidate(candidate) {
        throw new Error("Method 'onCandidate' must be implemented.");
    }

    onClose() {
        throw new Error("Method 'onClose' must be implemented.");
    }

    // Method to get the local media stream (audio/video)
    async getLocalMedia(constraints = { audio: true, video: true }) {
        try {
            this.localStream = await navigator.mediaDevices.getUserMedia(constraints);
            return this.localStream;
        } catch (error) {
            console.error("Error getting local media:", error);
            throw error;
        }
    }

    // Create a peer connection
    createPeerConnection() {
        if (this.pc) {
            console.warn("PeerConnection already exists");
            return;
        }
        this.pc = new RTCPeerConnection(this.config);

        this.pc.onicecandidate = (event) => {
            if (event.candidate) {
                this.socket.emit("signaling", {
                    type: "candidate",
                    candidate: event.candidate,
                    clientId: this.clientId,
                });
            }
        };

        this.pc.ontrack = (event) => {
            this.onRemoteStream(event.streams[0]);
        };

        // Add local tracks to the connection
        this.localStream.getTracks().forEach((track) => {
            this.pc.addTrack(track, this.localStream);
        });
    }

    onRemoteStream(remoteStream) {
        throw new Error("Method 'onRemoteStream' must be implemented.");
    }

    // Method to initiate a call
    async makeCall() {
        await this.createPeerConnection();

        const offer = await this.pc.createOffer();
        await this.pc.setLocalDescription(offer);

        this.socket.emit("signaling", {
            type: "offer",
            sdp: offer.sdp,
            clientId: this.clientId,
        });
    }

    // Method to handle an incoming offer
    async handleOffer(offer) {
        await this.createPeerConnection();
        await this.pc.setRemoteDescription(new RTCSessionDescription(offer));

        const answer = await this.pc.createAnswer();
        await this.pc.setLocalDescription(answer);

        this.socket.emit("signaling", {
            type: "answer",
            sdp: answer.sdp,
            clientId: this.clientId,
        });
    }

    async handleAnswer(answer) {
        if (!this.pc) {
            console.error("No peer connection to handle answer");
            return;
        }
        await this.pc.setRemoteDescription(new RTCSessionDescription(answer));
    }

    async handleCandidate(candidate) {
        if (!this.pc) {
            console.error("No peer connection to handle candidate");
            return;
        }
        await this.pc.addIceCandidate(new RTCIceCandidate(candidate));
    }

    async hangup() {
        if (this.pc) {
            this.pc.close();
            this.pc = null;
        }

        if (this.localStream) {
            this.localStream.getTracks().forEach((track) => track.stop());
        }

        this.socket.emit("signaling", { type: "bye", clientId: this.clientId });
    }
}

export default WebRTCConnection;
```

## Step 2: Extending the WebRTCConnection Class

Now that we have a base class, let’s extend it for both video and voice calls by implementing the required methods like `onReady`, `onOffer`, `onAnswer`, and `onClose`.

### VideoCallConnection Class

The `VideoCallConnection` class will handle both audio and video streams, displaying the local and remote video in `<video>` elements.

```javascript
import WebRTCConnection from "./WebRTCConnection";

class VideoCallConnection extends WebRTCConnection {
    constructor(config, signalingServerUrl, localVideo, remoteVideo) {
        super(config, signalingServerUrl);
        this.localVideo = localVideo;
        this.remoteVideo = remoteVideo;
    }

    async onReady() {
        console.log("VideoCall ready. Obtaining local media...");
        this.localStream = await this.getLocalMedia({ audio: true, video: true });
        this.localVideo.srcObject = this.localStream;
        this.makeCall(); // Start the call when media stream is ready
    }

    async onOffer(offer) {
        console.log("Received offer, creating answer...");
        await this.handleOffer(offer);
    }

    async onAnswer(answer) {
        console.log("Received answer, setting remote description...");
        await this.handleAnswer(answer);
    }

    async onCandidate(candidate) {
        console.log("Received ICE candidate, adding to peer connection...");
        await this.handleCandidate(candidate);
    }

    onRemoteStream(remoteStream) {
        console.log("Displaying remote stream");
        this.remoteVideo.srcObject = remoteStream;
    }

    onClose() {
        console.log("Call ended, cleaning up...");
        this.hangup();
    }
}

export default VideoCallConnection;
```

### VoiceCallConnection Class

The `VoiceCallConnection` will only handle audio streams without a video feed.

```javascript
import WebRTCConnection from "./WebRTCConnection";

class VoiceCallConnection extends WebRTCConnection {
    constructor(config, signalingServerUrl) {
        super(config, signalingServerUrl);
    }

    async onReady() {
        console.log("VoiceCall ready. Obtaining local audio stream...");
        this.localStream = await this.getLocalMedia({ audio: true, video: false });
        this.makeCall(); // Start the call when the audio stream is ready
    }

    async onOffer(offer) {
        console.log("Received offer, creating answer...");
        await this.handleOffer(offer);
    }

    async onAnswer(answer) {
        console.log("Received answer, setting remote description...");
        await this.handleAnswer(answer);
    }

    async onCandidate(candidate) {
        console.log("Received ICE candidate, adding to peer connection...");
        await this.handleCandidate(candidate);
    }

    onRemoteStream(remoteStream) {
        console.log("Playing remote audio stream");
        // Optionally play the remote audio stream here
    }

    onClose() {
        console.log("Call ended, cleaning up...");
        this.hangup();
    }
}

export default VoiceCallConnection;
```

## Step 3: Using the Classes

To use these classes, you’ll need to instantiate them.

## Step 4: Signaling Server Implementation using Socket.IO

To make the WebRTC connections work, we need a signaling server to exchange WebRTC offers, answers, and ICE candidates between peers. Here’s an example of a basic `Node.js` server using `Socket.IO` for signaling:

### Server Code (`server.js`)

```javascript
const express = require("express");
const http = require("http");
const socketIo = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = socketIo(server);

io.on("connection", (socket) => {
    console.log("A user connected:", socket.id);

    // Listen for offer and forward it to the other peer
    socket.on("signaling", (data) => {
        console.log("Signaling data:", data);
        // Broadcast the signaling data to all other connected peers
        socket.broadcast.emit("signaling", data);
    });

    socket.on("disconnect", () => {
        console.log("User disconnected:", socket.id);
    });
});

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

### Explanation

- **Socket.IO**: Handles real-time communication between the peers. When one peer sends a signaling message (offer, answer, or ICE candidate), the server broadcasts it to all other connected peers.
- **Connection and Disconnection**: When a user connects or disconnects, their actions are logged on the server for debugging purposes.
- **Signaling**: The server doesn’t store or process the signaling messages; it simply forwards them to all other connected peers.

### Running the Server

1. **Install Dependencies**: 
   ```bash
   npm init -y
   npm install express socket.io
   ```

2. **Start the Server**:
   ```bash
   node server.js
   ```

This will start a signaling server that your WebRTC clients can use to exchange signaling messages.
