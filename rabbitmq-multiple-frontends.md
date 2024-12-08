
# **How to Handle Multiple Frontends with a Single Backend: RabbitMQ vs REST API Approach**  

When building modern web applications, it's common to have **multiple frontend apps** (like admin panels, user dashboards, or mobile apps) connected to **a single backend**. But how do you keep them **in sync** when users log in, log out, or when their session changes?  

This is where two approaches come in:  
1. **REST API (Traditional Approach)**  
2. **Message Broker (RabbitMQ) Approach**  

In this article, you'll learn the key differences between these two approaches, how to implement both, and which method is **better for your project**.  

---

## 🔥 **What Problem Are We Solving?**  
Imagine you have 3 frontends:  
- **Frontend A**: A public website (React)  
- **Frontend B**: A mobile app (React Native)  
- **Frontend C**: An admin dashboard (React)  

They all share a common backend. You want to:  
1. **Log in once and stay logged in on all apps** (Single Sign-On, SSO)  
2. **Log out from one app and log out on all other apps** automatically.  
3. Keep all frontends updated in **real-time** (like session changes, notifications, etc.)  

---

## ⚔️ **Comparing REST API vs RabbitMQ for This Problem**  

| **Feature**         | **REST API**                | **RabbitMQ**                   |
|---------------------|----------------------------|---------------------------------|
| **Single Sign-On**   | ✅ Possible, but manual setup | ✅ Built-in via message queue   |
| **Auto Logout**      | ❌ Manual requests required  | ✅ Automatic via event listener |
| **Real-time Sync**   | ❌ Requires polling         | ✅ Real-time with events        |
| **Scalability**      | ⚠️ Harder with many users  | ✅ Handles thousands of users  |
| **Complexity**       | ✅ Simple, just REST calls  | ⚠️ Needs RabbitMQ setup        |
| **Speed**            | ❌ Slow (polling)           | ✅ Real-time sync               |

**Key Insight**:  
- If you want **real-time synchronization**, use **RabbitMQ**.  
- If you only need a simple setup, **REST API** can work but with limitations.  

---

## **How Does REST API Work?**  

**Login Process (REST API)**  
1. **User logs in** on Frontend A.  
2. The backend issues a **JWT** token and stores it in a **cookie**.  
3. Each frontend (Frontend A, B, and C) has to **poll the backend** to check if the user is authenticated.  
4. If the user logs out on Frontend A, it must send a request to the backend to log out.  
5. The backend informs other frontends of the logout, but these apps must check on their own (via polling).  

**Why It’s Not Ideal**:  
- **Polling is slow**: Frontends ask, "Am I still logged in?" every 5-10 seconds.  
- If the user logs out, other frontends won’t know instantly unless they request it.  

---

## **How Does RabbitMQ Work?**  

**Login Process (RabbitMQ Approach)**  
1. **User logs in** on Frontend A.  
2. The backend issues a **JWT** token and stores it in a **cookie**.  
3. The backend sends a **RabbitMQ event (USER_LOGGED_IN)** to all frontends.  
4. Frontend B and C immediately receive the event and sync their UI to show the user as logged in.  

**Logout Process**  
1. **User logs out** on Frontend A.  
2. The backend sends a **RabbitMQ event (USER_LOGGED_OUT)** to all frontends.  
3. Frontends B and C see the logout event and **immediately log the user out**.  

**Why It’s Better**:  
- No polling. **Real-time sync**.  
- RabbitMQ sends **instant notifications** to all frontends, so no need for them to keep asking the backend for the user's session state.  

---

## **Step-by-Step Implementation**  

### **1️⃣ REST API Approach (Simple, But Limited)**  

**Backend (Node.js with JWT & Cookies)**  

```javascript
const express = require('express');
const jwt = require('jsonwebtoken');
const cookieParser = require('cookie-parser');

const app = express();
app.use(cookieParser());
app.use(express.json());

const SECRET_KEY = 'supersecret';

app.post('/login', (req, res) => {
  const { username } = req.body;
  const token = jwt.sign({ username }, SECRET_KEY, { expiresIn: '1h' });
  res.cookie('authToken', token, { httpOnly: true });
  res.json({ message: 'Logged in' });
});
```

**Frontend (React)**  

```javascript
import axios from 'axios';
import { useEffect, useState } from 'react';

function App() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    axios.get('http://localhost:5000/verify-token', { withCredentials: true })
      .then(response => setUser(response.data.user))
      .catch(() => console.log('Not authenticated'));
  }, []);
}
```

---

### **2️⃣ RabbitMQ Approach (Real-Time Sync)**  

**Backend (Node.js with RabbitMQ)**  

```javascript
const express = require('express');
const amqp = require('amqplib/callback_api');

let channel;
amqp.connect('amqp://localhost', (err, connection) => {
  connection.createChannel((err, ch) => {
    channel = ch;
  });
});
```

**Frontend (React with Event Listener)**  

```javascript
import axios from 'axios';
import { useEffect, useState } from 'react';

function App() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    const ws = new WebSocket('ws://localhost:15674/ws');
    ws.onmessage = (message) => {
      const { event } = JSON.parse(message.data);
      if (event === 'USER_LOGGED_OUT') setUser(null);
    };
  }, []);
}
```

---

## **Which Should You Choose?**
- Use **REST API** if you want a simple, low-maintenance solution.  
- Use **RabbitMQ** if you need **real-time sync** or have **many frontends**.  

If you'd like a **full project repo** or help setting up **RabbitMQ**, just ask! 😊
