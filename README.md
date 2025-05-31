// server.js const express = require('express'); const fs = require('fs'); const app = express(); const PORT = process.env.PORT || 3000;

app.set('view engine', 'ejs'); app.use(express.urlencoded({ extended: true })); app.use(express.static('public'));

let db = JSON.parse(fs.readFileSync('./db.json'));

app.use((req, res, next) => { req.user = db.users[0]; next(); });

app.get('/', (req, res) => { res.render('index', { user: req.user, result: db.latest_result }); });

app.post('/bet', (req, res) => { const bet = req.body.color; if (['big', 'small', 'violet'].includes(bet)) { db.bets.push({ user: req.user.username, bet, time: Date.now() }); fs.writeFileSync('./db.json', JSON.stringify(db, null, 2)); } res.redirect('/'); });

app.get('/admin', (req, res) => { res.render('admin', { bets: db.bets, result: db.latest_result }); });

app.post('/admin/set-result', (req, res) => { const { result } = req.body; if (['big', 'small', 'violet'].includes(result)) { db.latest_result = result; fs.writeFileSync('./db.json', JSON.stringify(db, null, 2)); } res.redirect('/admin'); });

app.listen(PORT, () => console.log(Server running on http://localhost:${PORT}));

// package.json { "name": "wingo-clone", "version": "1.0.0", "description": "Wingo-style color prediction game", "main": "server.js", "scripts": { "start": "node server.js" }, "dependencies": { "ejs": "^3.1.6", "express": "^4.17.1" } }

// db.json { "users": [ { "username": "demo_user", "balance": 1000 } ], "bets": [], "latest_result": "big" }

// views/index.ejs

<!DOCTYPE html><html>
<head>
  <title>Wingo Clone</title>
  <link rel="stylesheet" href="/style.css">
</head>
<body>
  <h1>Welcome, <%= user.username %></h1>
  <p>Balance: <%= user.balance %></p>
  <p>Latest Result: <%= result %></p>
  <form action="/bet" method="POST">
    <button name="color" value="big">Big</button>
    <button name="color" value="small">Small</button>
    <button name="color" value="violet">Violet</button>
  </form>
</body>
</html>// views/admin.ejs

<!DOCTYPE html><html>
<head>
  <title>Admin Panel</title>
  <link rel="stylesheet" href="/style.css">
</head>
<body>
  <h1>Admin Panel</h1>
  <p>Current Result: <%= result %></p>
  <form action="/admin/set-result" method="POST">
    <select name="result">
      <option value="big">Big</option>
      <option value="small">Small</option>
      <option value="violet">Violet</option>
    </select>
    <button type="submit">Set Result</button>
  </form>
  <h2>Bet History</h2>
  <ul>
    <% bets.forEach(function(bet) { %>
      <li><%= bet.user %> bet on <%= bet.bet %> at <%= new Date(bet.time).toLocaleString() %></li>
    <% }); %>
  </ul>
</body>
</html>// public/style.css body { font-family: Arial, sans-serif; margin: 20px; } button { margin: 5px; padding: 10px 20px; }

![frame_5](https://github.com/user-attachments/assets/65737334-876e-4cd5-9632-975a8669d185)
