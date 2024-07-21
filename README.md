sales-app/
│
├── backend/
│   ├── models/
│   ├── routes/
│   ├── controllers/
│   └── app.js
│
├── frontend/
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   ├── post-item.html
│   ├── view-items.html
│   └── styles.css
│
└── package.jsonconst express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');

const app = express();
const port = 3000;

app.use(cors());
app.use(express.json());

mongoose.connect('mongodb://localhost:27017/sales-app', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

const userRoutes = require('./routes/userRoutes');
const itemRoutes = require('./routes/itemRoutes');

app.use('/api/users', userRoutes);
app.use('/api/items', itemRoutes);

app.listen(port, () => {
    console.log(`Server is running on http://localhost:${port}`);
});const express = require('express');
const router = express.Router();
const Item = require('../models/Item');
const jwt = require('jsonwebtoken');

const authMiddleware = (req, res, next) => {
    const token = req.header('Authorization').replace('Bearer ', '');
    try {
        const decoded = jwt.verify(token, 'secretkey');
        req.userId = decoded.userId;
        next();
    } catch (error) {
        res.status(401).send('Unauthorized');
    }
};

router.post('/', authMiddleware, async (req, res) => {
    try {
        const { title, description, price } = req.body;
        const item = new Item({ title, description, price, userId: req.userId });
        await item.save();
        res.status(201).send('Item posted successfully');
    } catch (error) {
        res.status(400).send(error.message);
    }
});

router.get('/', async (req, res) => {
    try {
        const items = await Item.find().populate('userId', 'username');
        res.status(200).json(items);
    } catch (error) {
        res.status(400).send(error.message);
    }
});

module.exports = router;<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <title>Sales App</title>
</head>
<body class="bg-gray-100">
    <div class="container mx-auto p-4">
        <h1 class="text-2xl font-bold mb-4">Welcome to the Sales App</h1>
        <a href="register.html" class="block bg-blue-500 text-white py-2 px-4 rounded">Register</a>
        <a href="login.html" class="block bg-green-500 text-white py-2 px-4 rounded mt-2">Login</a>
        <a href="post-item.html" class="block bg-yellow-500 text-white py-2 px-4 rounded mt-2">Post Item</a>
        <a href="view-items.html" class="block bg-purple-500 text-white py-2 px-4 rounded mt-2">View Items</a>
    </div>
</body>
</html><!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
    <title>Register</title>
</head>
<body class="bg-gray-100">
    <div class="container mx-auto p-4">
        <h1 class="text-2xl font-bold mb-4">Register</h1>
        <form id="registerForm">
            <input type="text" id="username" placeholder="Username" class="block w-full p-2 border rounded mb-2">
            <input type="password" id="password" placeholder="Password" class="block w-full p-2 border rounded mb-2">
            <button type="submit" class="bg-blue-500 text-white py-2 px-4 rounded">Register</button>
        </form>
    </div>
    <script>
        document.getElementById('registerForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const username = document.getElementById('username').value;
            const password = document.getElementById('password').value;
            const response = await fetch('http://localhost:3000/api/users/register', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ username, password })
            });
            const result = await response.text();
            alert(result);
        });
    </script>
</body>
</html>
