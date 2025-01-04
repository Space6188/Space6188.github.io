
## RESTfull сервіс для управління даними

const express = require('express');
const mysql = require('mysql2');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

app.use(bodyParser.json());

const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: 'password',
    database: 'testdb',
});

db.connect((err) => {
    if (err) {
        console.error('Error connecting to MySQL:', err);
        return;
    }
    console.log('Connected to MySQL');
});

const createUsersTable = `CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    age INT NOT NULL
)`;

db.query(createUsersTable, (err) => {
    if (err) {
        console.error('Error creating users table:', err);
    } else {
        console.log('Users table created or already exists');
    }
});

const Routes = Object.freeze({
    CREATE: '/users',
    READ_ALL: '/users',
    READ_ONE: '/users/:id',
    UPDATE: '/users/:id',
    DELETE: '/users/:id',
});

app.post(Routes.CREATE, (req, res) => {
    const { name, email, age } = req.body;
    const query = 'INSERT INTO users (name, email, age) VALUES (?, ?, ?)';
    db.query(query, [name, email, age], (err, result) => {
        if (err) {
            console.error('Error creating user:', err);
            res.status(500).send('Error creating user');
        } else {
            res.status(200).json({ id: result.insertId, name, email, age });
        }
    });
});

app.get(Routes.READ_ALL, (req, res) => {
    const query = 'SELECT * FROM users';
    db.query(query, (err, results) => {
        if (err) {
            console.error('Error fetching users:', err);
            res.status(500).send('Error fetching users');
        } else {
            res.json(results);
        }
    });
});

app.get(Routes.READ_ONE, (req, res) => {
    const { id } = req.params;
    const query = 'SELECT * FROM users WHERE id = ?';
    db.query(query, [id], (err, results) => {
        if (err) {
            console.error('Error fetching user:', err);
            res.status(500).send('Error fetching user');
        } else if (results.length === 0) {
            res.status(404).send('User not found');
        } else {
            res.json(results[0]);
        }
    });
});

app.put(Routes.UPDATE, (req, res) => {
    const { id } = req.params;
    const { name, email, age } = req.body;
    const query = 'UPDATE users SET name = ?, email = ?, age = ? WHERE id = ?';
    db.query(query, [name, email, age, id], (err, result) => {
        if (err) {
            console.error('Error updating user:', err);
            res.status(500).send('Error updating user');
        } else if (result.affectedRows === 0) {
            res.status(404).send('User not found');
        } else {
            res.json({ id, name, email, age });
        }
    });
});

app.delete(Routes.DELETE, (req, res) => {
    const { id } = req.params;
    const query = 'DELETE FROM users WHERE id = ?';
    db.query(query, [id], (err, result) => {
        if (err) {
            console.error('Error deleting user:', err);
            res.status(500).send('Error deleting user');
        } else if (result.affectedRows === 0) {
            res.status(404).send('User not found');
        } else {
            res.status(200).send();
        }
    });
});

app.listen(port, () => {
    console.log(`Server running on http://localhost:${port}`);
});

