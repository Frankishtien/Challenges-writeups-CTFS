## js code

<details>
  <summary>js code</summary>


```javascript
const express = require('express');
const session = require('express-session');
const bodyParser = require('body-parser');
const fs = require('fs');
const path = require('path');
const app = express();
const puppeteer = require('puppeteer');


// --------------------------------------------------Auxiliary Code
app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.urlencoded({ extended: true }));
app.use(session({
    secret: process.env.SECRET,
    resave: false,
    saveUninitialized: true
}));

app.use((req, res, next) => {
    if (!req.session.csrfToken) {
        req.session.csrfToken = generateCSRFToken();
    }
    next();
});

const crypto = require('crypto');

function generateCSRFToken() {
    return crypto.randomBytes(32).toString('hex');
}

function verifyCSRF(req, res) {
    const tokenFromBody = req.body._csrf;
    const tokenFromSession = req.session.csrfToken;
    return tokenFromBody && tokenFromSession && tokenFromBody === tokenFromSession;
}

function escapeHTML(str) {
    return String(str).replace(/[&<>'"`=\/]/g, function (char) {
        return {
            '&': '&amp;',
            '<': '&lt;',    
            '>': '&gt;',
            "'": '&#x27;',
            '"': '&quot;',
            '`': '&#x60;',
            '=': '&#x3D;',
            '/': '&#x2F;'
        }[char];
    });
}

// Insecure in-memory users database, only for XSS challenge purposes
const users = {
    alice: { username: 'alice', password: 'pass', bio: 'Hi, I am Alice!' },
    bob: { username: 'bob', password: process.env.PASSWORD, bio: 'Hello from Bob!' }
};

let sharedComments = [];


function isAuthenticated(req) {
    return req.session && req.session.username && users[req.session.username];
}

function renderHTML(filePath, replacements = {}) {
    let content = fs.readFileSync(path.join(__dirname, 'public', filePath), 'utf-8');
    for (let key in replacements) {
        const regex = new RegExp(`{{${key}}}`, 'g');
        content = content.replace(regex, replacements[key]);
    }
    return content;
}

// -------------------------------------------- Routes

app.get('/', (req, res) => {
    if (isAuthenticated(req)) return res.redirect('/profile');
    res.send(renderHTML('login.html', { error: '' }));
});

// Insecure login flow, only for XSS challenge purposes
app.post('/login', (req, res) => {
    const { username, password } = req.body;

    if (users[username] && users[username].password === password) {
        req.session.username = username;

        // Set masterkey cookie
        let masterKeyValue = '';

        if (username === 'alice') {
            masterKeyValue = 'DEMOMASTERKEY';
        } else if (username === 'bob') {
            try {
                masterKeyValue = fs.readFileSync('/tmp/masterkey.txt', 'utf8').trim();
            } catch (err) {
                console.error('[!] Failed to read /tmp/masterkey.txt:', err);
                masterKeyValue = 'ERROR';
            }
        }

        res.cookie('masterkey', masterKeyValue, {
            httpOnly: true, // NOT Accessible by javascript code
        });

        return res.redirect('/profile');
    }

    res.send(renderHTML('login.html', { error: '<p class="error">Invalid credentials</p>', csrfToken: req.session.csrfToken }));
});

app.get('/profile', (req, res) => {
    if (!isAuthenticated(req)) return res.redirect('/');
    const user = users[req.session.username];
    res.send(renderHTML('profile.html', {
        username: escapeHTML(user.username),
        bio: escapeHTML(user.bio),
        csrfToken: req.session.csrfToken}));
});

app.post('/profile', (req, res) => {
    if (!isAuthenticated(req)) return res.redirect('/');
    if (!verifyCSRF(req, res)) return res.status(403).send('Invalid CSRF token');
    const { bio, password } = req.body;
    const user = users[req.session.username];
    user.bio = bio;
    if (password && password.trim()) user.password = password;
    res.redirect('/profile');
});

app.get('/shared', (req, res) => {
    if (!isAuthenticated(req)) return res.redirect('/');
    const commentHTML = sharedComments.map(c =>
        `<li><strong>${escapeHTML(c.author)}:</strong> ${escapeHTML(c.text)}</li>`
    ).join('');
    res.send(renderHTML('shared.html', { comments: commentHTML, csrfToken: req.session.csrfToken }));
});

app.post('/shared', (req, res) => {
    if (!isAuthenticated(req)) return res.redirect('/');
    if (!verifyCSRF(req, res)) return res.status(403).send('Invalid CSRF token');
    const comment = {
        author: req.session.username,
        text: req.body.comment
    };
    sharedComments.push(comment);
    res.redirect('/shared');
});

app.get('/search', (req, res) => {
    if (!isAuthenticated(req)) return res.status(401).send('Unauthorized');

    const query = req.query.query || '';
    if (!query) return res.status(400).send('Query parameter is required');

    const results = sharedComments.filter(c => 
        c.text.includes(query)
    );

    let resultsHTML = '';
    if (results.length === 0) {
        resultsHTML = `<p>No matches found.</p>`;
    } else {
        resultsHTML = '<ul>';
        results.forEach(c => {
            resultsHTML += `<li><strong>${escapeHTML(c.author)}:</strong> ${escapeHTML(c.text)}</li>`;
        });
        resultsHTML += '</ul>';
    }

    const responseHTML = `
        <html>
        <head><title>Search Results</title></head>
        <body>
            <div id="results-header"></div>
            <script>
                var searchQuery = "${query}";
                document.getElementById('results-header').innerHTML = '<h2>Search Results for "' + searchQuery + '"</h2>';
            </script>
            ${resultsHTML}
            <br><a href="/shared">Back to Shared Area</a>
        </body>
        </html>
    `;

    res.send(responseHTML);
});



app.get('/logout', (req, res) => {
    req.session.destroy(() => {
        res.redirect('/');
    });
});



app.get('/report', async (req, res) => {
    res.send('Bob is checking it out...');

    // Bob's logic
});



app.listen(80, () => {
    console.log('Running on http://localhost:80');
});

```


  
</details>



---
























































































