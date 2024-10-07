
# Building a WebRTC Application for Real-Time Video and Voice Calls

**WebRTC (Web Real-Time Communication)** enables peer-to-peer communication directly between browsers, allowing for real-time video, voice, and data exchange. This tutorial demonstrates how to build a WebRTC-based application with video and voice call functionality, using **Socket.IO** as the signaling server for the connection setup.

In this guide, we'll walk through how WebRTC handles media streams, establishes peer connections, exchanges ICE candidates, and manages both video and voice calls, complete with visualizations for audio streams.

## 1. WebRTC Configuration and ICE Servers

WebRTC relies on **ICE (Interactive Connectivity Establishment)** to discover the best path between peers for establishing a connection. This can involve direct communication between devices, or relaying media through a **TURN server** if direct communication isn’t possible due to network restrictions like NAT or firewalls.

```javascript
const config = {
    iceServers: [
        {
            urls: ["turn:203.161.53.228:3478?transport=udp"],
            username: "cloudturn",
            credential: "cloudturn",
        },
        {
            urls: ["stun:stun.l.google.com:19302"], // Google STUN server
        },
    ],
    iceTransportPolicy: "relay",
};
```

- **STUN (Session Traversal Utilities for NAT)** helps clients discover their public IP addresses when behind a NAT.
- **TURN (Traversal Using Relays around NAT)** relays media traffic between peers if they cannot establish a direct connection.
- The `iceTransportPolicy: "relay"` forces WebRTC to always rely on TURN for connection, ensuring media is relayed if direct peer-to-peer communication isn't possible.

## 2. Establishing a Signaling Server with Socket.IO

A **signaling server** is essential in WebRTC to exchange connection details between peers. This is done through Socket.IO, which facilitates the transfer of information such as offers, answers, and **ICE candidates**. Once the peers have all necessary details, they can establish a direct connection.

```javascript
const socket = io("https://test-api.cloudmateria.com");

let clientId = null;

socket.on("connect", () => {
    console.log("Connected to Socket.IO signaling server");
});

socket.on("client-id", (data) => {
    clientId = data.id;
    console.log(`Assigned Client ID: ${clientId}`);
});

socket.on("message", (data) => {
    handleSignalingMessage(data);
});
```

The signaling server assigns a unique `clientId` to each peer and facilitates message exchange for setting up the WebRTC connection.

## 3. Handling Signaling Messages

WebRTC connections are initiated and maintained through the exchange of **signaling messages**. These messages include offers, answers, and ICE candidates.

```javascript
function handleSignalingMessage(data) {
    switch (data.type) {
        case "offer":
            handleOffer(data);
            break;
        case "answer":
            handleAnswer(data);
            break;
        case "candidate":
            handleCandidate(data);
            break;
        case "ready":
            makeCall();
            break;
        case "bye":
            hangup();
            break;
        default:
            console.log("Unhandled signaling message:", data);
    }
}
```

Signaling messages enable peers to establish connections:
- **Offer**: Sent by the caller to initiate a connection.
- **Answer**: Sent by the callee in response to the offer.
- **ICE Candidate**: Represents potential network paths between peers.
- **Ready**: Signals that the peers are ready to make a connection.
- **Bye**: Ends the call.

## 4. Button Handlers for Video and Voice Calls

The UI includes buttons to start a video or voice call, and a button to hang up. Clicking these buttons triggers the appropriate WebRTC setup.

```javascript
const startVideoButton = document.getElementById("startVideoButton");
const startVoiceButton = document.getElementById("startVoiceButton");
const hangupButton = document.getElementById("hangupButton");

startVideoButton.onclick = async () => {
    await startCall({ video: true, audio: true }, true); // Video call
};

startVoiceButton.onclick = async () => {
    await startCall({ audio: true }, false); // Voice call
};

hangupButton.onclick = async () => {
    hangup();
    socket.emit("message", { type: "bye", clientId });
};
```

The `startCall` function requests media using `navigator.mediaDevices.getUserMedia`, acquiring either video and audio or just audio, depending on the call type. The call setup is then handled by WebRTC.

## 5. Media Stream Acquisition

When a user starts a call, the browser requests access to the user's camera and microphone to obtain media streams. These streams are then attached to the WebRTC connection.

```javascript
async function startCall(constraints, isVideoCall) {
    localStream = await navigator.mediaDevices.getUserMedia(constraints);
    if (isVideoCall) {
        localVideo.srcObject = localStream;
    } else {
        setupVoiceGraph(localStream); // For voice calls, set up the voice graph
    }
    socket.emit("message", { type: "ready", clientId });
}
```

- **Video Call**: Both video and audio streams are obtained.
- **Voice Call**: Only the audio stream is obtained and visualized using a voice graph.

## 6. Creating the Peer Connection

The **RTCPeerConnection** object manages the WebRTC connection between two peers, including the negotiation of media streams and ICE candidates.

```javascript
function createPeerConnection() {
    pc = new RTCPeerConnection(config);

    pc.onicecandidate = (e) => {
        if (e.candidate) {
            socket.emit("message", { type: "candidate", candidate: e.candidate.candidate, clientId });
        }
    };

    pc.ontrack = (e) => {
        remoteVideo.srcObject = e.streams[0];
    };

    localStream.getTracks().forEach((track) => {
        pc.addTrack(track, localStream);
    });
}
```

The **onicecandidate** event handler sends gathered ICE candidates to the other peer through the signaling server, while **ontrack** handles receiving remote media streams.

## 7. Offer/Answer Exchange

In WebRTC, connection negotiation follows the **offer/answer** model. The caller creates an offer, and the callee responds with an answer. This negotiation is conducted using **SDP (Session Description Protocol)**.

```javascript
async function makeCall() {
    await createPeerConnection();
    const offer = await pc.createOffer();
    socket.emit("message", { type: "offer", sdp: offer.sdp, clientId });
    await pc.setLocalDescription(offer);
}

async function handleOffer(offer) {
    await createPeerConnection();
    await pc.setRemoteDescription(new RTCSessionDescription(offer));
    const answer = await pc.createAnswer();
    socket.emit("message", { type: "answer", sdp: answer.sdp, clientId });
    await pc.setLocalDescription(answer);
}
```

- **Offer**: Created by the initiating peer and sent to the signaling server.
- **Answer**: Created by the receiving peer and returned to the signaling server.

## 8. ICE Candidate Exchange

Once the offer and answer have been exchanged, the peers start gathering and exchanging **ICE candidates**. These candidates represent potential routes for the connection.

```javascript
pc.onicecandidate = (e) => {
    if (e.candidate) {
        socket.emit("message", { type: "candidate", candidate: e.candidate.candidate, clientId });
    }
};
```

As candidates are found, they are sent to the remote peer, and once the best candidate is identified, the peers can connect and begin transmitting media.

## 9. Visualizing Audio with a Voice Graph

For voice-only calls, we can visualize the audio frequencies using the **Web Audio API**. This is implemented through a frequency visualizer, where an `AnalyserNode` analyzes the audio stream and renders the data on a canvas.

```javascript
function setupVoiceGraph(stream) {
    const audioContext = new AudioContext();
    const source = audioContext.createMediaStreamSource(stream);
    analyser = audioContext.createAnalyser();
    source.connect(analyser);

    analyser.fftSize = 256;
    bufferLength = analyser.frequencyBinCount;
    dataArray = new Uint8Array(bufferLength);

    draw();
}
```

The **draw** function renders a bar graph based on the frequency data of the audio stream in real-time, providing a visual representation of the sound.

## 10. Ending the Call

To terminate a WebRTC connection, the `RTCPeerConnection` is closed, and all media tracks are stopped.

```javascript
async function hangup() {
    if (pc) {
        pc.close();
        pc = null;
    }
    if (localStream) {
        localStream.getTracks().forEach((track) => track.stop());
    }
}
```

The **hangup** function resets the connection state and stops the media stream, effectively ending the call.

## Conclusion

This WebRTC application demonstrates how to set up real-time communication with both video and voice calling functionality. By using WebRTC’s **RTCPeerConnection**, **ICE** candidates, and the **offer/answer** model, we can establish peer-to-peer connections that support media streaming directly between browsers.

With **Socket.IO** handling the signaling process, we are able to connect peers in various network environments, relying on **TURN/STUN** servers for connectivity.
