# InstaChat: Real-Time Chat Application

## Overview
This is a real-time chat application built using the MERN stack (MongoDB, Express.js, React.js, Node.js) and Socket.IO. The application supports real-time messaging, user authentication, and group chat functionality.

## Setup and Environment

### Prerequisites
- Operating System: Ubuntu 24.04
- Node.js
- MongoDB
- Docker
- Kubernetes with Minikube (optional for local Kubernetes setup)

### Installation Steps

#### 1. Install Node.js and npm
```bash
sudo apt update
sudo apt install -y nodejs npm
```

#### 2. Install MongoDB
```bash
sudo apt install -y mongodb
```

#### 3. Install Docker
```bash
sudo apt install -y docker.io
sudo systemctl start docker
sudo systemctl enable docker
```

#### 4. Install Kubernetes and Minikube
```bash
sudo apt install -y apt-transport-https
sudo curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo bash -c 'cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF'
sudo apt update
sudo apt install -y kubectl
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube /usr/local/bin/
```

## Backend Development

### 1. Set up the backend server
```bash
mkdir backend
cd backend
npm init -y
npm install express mongoose dotenv bcryptjs jsonwebtoken socket.io
```

### 2. Create server.js
```javascript
const express = require('express');
const mongoose = require('mongoose');
const dotenv = require('dotenv');
const socketIO = require('socket.io');
const http = require('http');
const authRoutes = require('./routes/auth');
const messageRoutes = require('./routes/messages');
const chatRoomRoutes = require('./routes/chatRooms');

dotenv.config();

const app = express();
const server = http.createServer(app);
const io = socketIO(server);

mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('MongoDB connected'))
    .catch(err => console.log(err));

app.use(express.json());
app.use('/api/auth', authRoutes);
app.use('/api/messages', messageRoutes);
app.use('/api/chatrooms', chatRoomRoutes);

io.on('connection', socket => {
    console.log('New client connected');
    socket.on('disconnect', () => {
        console.log('Client disconnected');
    });
});

const PORT = process.env.PORT || 5000;
server.listen(PORT, () => console.log(`Server running on port ${PORT}));
```

### 3. Create RESTful API endpoints

#### User Authentication Routes
```javascript
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const User = require('../models/User');
const router = express.Router();

router.post('/register', async (req, res) => {
    // Registration logic
});

router.post('/login', async (req, res) => {
    // Login logic
});

module.exports = router;
```

#### Message Handling Routes
```javascript
const express = require('express');
const Message = require('../models/Message');
const router = express.Router();

router.post('/', async (req, res) => {
    // Message handling logic
});

module.exports = router;
```

#### Chat Room Management Routes
```javascript
const express = require('express');
const ChatRoom = require('../models/ChatRoom');
const router = express.Router();

router.post('/', async (req, res) => {
    // Chat room management logic
});

module.exports = router;
```

### 4. Create Mongoose models

#### User Model
```javascript
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
    username: { type: String, required: true },
    email: { type: String, required: true },
    password: { type: String, required: true },
});

module.exports = mongoose.model('User', UserSchema);
```

#### Message Model
```javascript
const mongoose = require('mongoose');

const MessageSchema = new mongoose.Schema({
    chatRoomId: { type: String, required: true },
    userId: { type: String, required: true },
    message: { type: String, required: true },
    timestamp: { type: Date, default: Date.now },
});

module.exports = mongoose.model('Message', MessageSchema);
```

#### ChatRoom Model
```javascript
const mongoose = require('mongoose');

const ChatRoomSchema = new mongoose.Schema({
    name: { type: String, required: true },
    users: [{ type: mongoose.Schema.Types.ObjectId, ref: 'User' }],
});

module.exports = mongoose.model('ChatRoom', ChatRoomSchema);
```

## Frontend Development

### 1. Set up the React.js project
```bash
npx create-react-app frontend
cd frontend
npm install axios socket.io-client jwt-decode
```

### 2. Create authentication components and pages

#### Login Component
```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Login = () => {
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        const response = await axios.post('/api/auth/login', { email, password });
        localStorage.setItem('token', response.data.token);
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
            <button type="submit">Login</button>
        </form>
    );
};

export default Login;
```

#### Register Component
```javascript
import React, { useState } from 'react';
import axios from 'axios';

const Register = () => {
    const [username, setUsername] = useState('');
    const [email, setEmail] = useState('');
    const [password, setPassword] = useState('');

    const handleSubmit = async (e) => {
        e.preventDefault();
        await axios.post('/api/auth/register', { username, email, password });
    };

    return (
        <form onSubmit={handleSubmit}>
            <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} />
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
            <button type="submit">Register</button>
        </form>
    );
};

export default Register;
```

### 3. Set up Socket.IO on the client side
```javascript
import io from 'socket.io-client';

const socket = io('http://localhost:5000');

socket.on('connect', () => {
    console.log('Connected to server');
});

socket.on('disconnect', () => {
    console.log('Disconnected from server');
});

export default socket;
```

### 4. Create chat components

#### ChatRoom Component
```javascript
import React, { useState, useEffect } from 'react';
import socket from './socket';

const ChatRoom = ({ chatRoomId }) => {
    const [messages, setMessages] = useState([]);
    const [message, setMessage] = useState('');

    useEffect(() => {
        socket.on('message', (newMessage) => {
            setMessages((prevMessages) => [...prevMessages, newMessage]);
        });

        return () => {
            socket.off('message');
        };
    }, []);

    const handleSendMessage = () => {
        socket.emit('message', { chatRoomId, message });
        setMessage('');
    };

    return (
        <div>
            <div>
                {messages.map((msg, index) => (
                    <div key={index}>{msg}</div>
                ))}
            </div>
            <input
                type="text"
                value={message}
                onChange={(e) => setMessage(e.target.value)}
            />
            <button onClick={handleSendMessage}>Send</button>
        </div>
    );
};

export default ChatRoom;
```

## Deployment

### 1. Containerize the application using Docker

#### Dockerfile for Backend
```dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
```

#### Dockerfile for Frontend
```dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

RUN npm run build

EXPOSE 3000

CMD ["npx", "serve", "-s", "build"]
```

### 2. Create Kubernetes configuration files

#### Backend Deployment and Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
      - name: backend
        image: your-backend-image
        ports:
        - containerPort: 5000
        env:
        - name: MONGO_URI
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: uri

---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
  - protocol: TCP
    port: 5000
    targetPort: 5000
```

#### Frontend Deployment and Service
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
      - name: frontend
        image: your-frontend-image
        ports:
        - containerPort: 3000

---
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 3000
    targetPort: 3000
  type: LoadBalancer
```

## Documentation

### Create a README.md file
```markdown
# Real-Time Chat Application

## Overview
This is a real-time chat application built using the MERN stack (MongoDB, Express.js, React.js, Node.js) and Socket.IO. The application supports real-time messaging, user authentication, and group chat functionality.

## Setup and Environment
- Operating System: Ubuntu 24.04
- Dependencies: Node.js, MongoDB, Docker, Kubernetes

## Installation

### Backend
```
cd backend
npm install
node server.js
```

### Frontend
```
cd frontend
npm install
npm start
```

## Deployment

### Docker
```
docker build -t your-backend-image -f backend/Dockerfile .
docker build -t your-frontend-image -f frontend/Dockerfile .
```

### Kubernetes
```
kubectl apply -f k8s/backend-deployment.yaml
kubectl apply -f k8s/frontend-deployment.yaml
```

## Features
- Real-time messaging
- User authentication (JWT and Google Auth)
- Group chat functionality
- Notifications

## Additional Resources
- [Building a Real-Time Chat Application Using MERN Stack and Socket.IO](https://example.com)
- [Build and Deploy a Complete Chat App with MERN Stack - YouTube](https://youtube.com)
- [MERN Stack Real Time Chat App With Express, React, MongoDB - Udemy](https://udemy.com)
```
