// server.js
const express = require('express');
const fs = require('fs');
const app = express();
const PORT = process.env.PORT || 3000;

app.set('view engine', 'ejs');
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

let db = JSON.parse(fs.readFileSync('./db.json'));

app.use((req, res, next) => {
  req.user = db.users[0];
  next();
});

app.get('/', (req, res) => {
  res.render('index', { user: req.user, result: db.latest_result });
});

app.post('/bet', (req, res) => {
  const bet = req.body.color;
  if (['big', 'small', 'violet'].includes(bet)) {
    db.bets.push({ user: req.user.username, bet, time: Date.now() });
    fs.writeFileSync('./db.json', JSON.stringify(db, null, 2));
  }
  res.redirect('/');
});

app.get('/admin', (req, res) => {
  res.render('admin', { bets: db.bets, result: db.latest_result });
});

app.post('/admin/set-result', (req, res) => {
  const { result } = req.body;
  if (['big', 'small', 'violet'].includes(result)) {
    db.latest_result = result;
    fs.writeFileSync('./db.json', JSON.stringify(db, null, 2));
  }
  res.redirect('/admin');
});

app.listen(PORT, () => console.log(`Server running on http://localhost:${PORT}`));
