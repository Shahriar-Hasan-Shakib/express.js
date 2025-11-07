# Template Engines - EJS, Pug, Handlebars

## সংক্ষিপ্ত পরিচিতি

Template engines আপনাকে dynamic HTML pages তৈরি করতে দেয়। Express.js এর সাথে বিভিন্ন template engine ব্যবহার করা যায়।

## EJS (Embedded JavaScript)

### Installation:
```bash
npm install ejs
```

### Setup:
```javascript
const express = require('express');
const app = express();

app.set('view engine', 'ejs');
app.set('views', './views'); // optional, default is './views'

app.get('/', (req, res) => {
  res.render('index', {
    title: 'হোম পেজ',
    user: { name: 'রহিম', age: 25 }
  });
});

app.listen(3000);
```

### EJS Syntax:

**views/index.ejs:**
```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <h1>স্বাগতম, <%= user.name %>!</h1>
  
  <% if (user.age >= 18) { %>
    <p>আপনি প্রাপ্তবয়স্ক</p>
  <% } else { %>
    <p>আপনি নাবালক</p>
  <% } %>
  
  <ul>
    <% ['আম', 'কলা', 'আপেল'].forEach(fruit => { %>
      <li><%= fruit %></li>
    <% }) %>
  </ul>
</body>
</html>
```

### EJS Tags:
- `<%= %>` - Output escaped (HTML safe)
- `<%- %>` - Output unescaped (raw HTML)
- `<% %>` - JavaScript code
- `<%# %>` - Comment

### Partials (Reusable Components):

**views/partials/header.ejs:**
```ejs
<header>
  <h1>My Website</h1>
  <nav>
    <a href="/">হোম</a>
    <a href="/about">About</a>
  </nav>
</header>
```

**views/index.ejs:**
```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
</head>
<body>
  <%- include('partials/header') %>
  
  <main>
    <p>Main content</p>
  </main>
  
  <%- include('partials/footer') %>
</body>
</html>
```

## Pug (formerly Jade)

### Installation:
```bash
npm install pug
```

### Setup:
```javascript
const express = require('express');
const app = express();

app.set('view engine', 'pug');

app.get('/', (req, res) => {
  res.render('index', {
    title: 'হোম পেজ',
    users: ['রহিম', 'করিম', 'সালমা']
  });
});

app.listen(3000);
```

### Pug Syntax:

**views/index.pug:**
```pug
doctype html
html
  head
    title= title
  body
    h1 স্বাগতম!
    
    if users.length > 0
      ul
        each user in users
          li= user
    else
      p কোনো user নেই
    
    //- Comment
    p This is a paragraph with #[strong bold text]
```

## Handlebars

### Installation:
```bash
npm install express-handlebars
```

### Setup:
```javascript
const express = require('express');
const { engine } = require('express-handlebars');
const app = express();

app.engine('handlebars', engine());
app.set('view engine', 'handlebars');
app.set('views', './views');

app.get('/', (req, res) => {
  res.render('home', {
    title: 'হোম পেজ',
    users: [
      { name: 'রহিম', age: 25 },
      { name: 'করিম', age: 30 }
    ]
  });
});

app.listen(3000);
```

### Handlebars Syntax:

**views/home.handlebars:**
```handlebars
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  <h1>Users List</h1>
  
  {{#if users}}
    <ul>
      {{#each users}}
        <li>{{this.name}} - {{this.age}} বছর</li>
      {{/each}}
    </ul>
  {{else}}
    <p>কোনো users নেই</p>
  {{/if}}
</body>
</html>
```

### Layouts:

**views/layouts/main.handlebars:**
```handlebars
<!DOCTYPE html>
<html>
<head>
  <title>{{title}}</title>
</head>
<body>
  {{{body}}}
</body>
</html>
```

## সম্পূর্ণ উদাহরণ: Blog with EJS

```javascript
const express = require('express');
const app = express();

app.set('view engine', 'ejs');
app.use(express.static('public'));
app.use(express.urlencoded({ extended: true }));

const posts = [
  { id: 1, title: 'Express শিখুন', content: 'Express অসাধারণ...', author: 'রহিম' },
  { id: 2, title: 'Node.js টিউটোরিয়াল', content: 'Node.js একটি...', author: 'করিম' }
];

// Home page
app.get('/', (req, res) => {
  res.render('index', {
    title: 'Blog হোম',
    posts: posts
  });
});

// Single post
app.get('/post/:id', (req, res) => {
  const post = posts.find(p => p.id === parseInt(req.params.id));
  
  if (!post) {
    return res.status(404).render('404', { title: 'Not Found' });
  }
  
  res.render('post', {
    title: post.title,
    post: post
  });
});

// New post form
app.get('/new', (req, res) => {
  res.render('new-post', { title: 'নতুন পোস্ট' });
});

// Create post
app.post('/posts', (req, res) => {
  const newPost = {
    id: posts.length + 1,
    title: req.body.title,
    content: req.body.content,
    author: req.body.author
  };
  
  posts.push(newPost);
  res.redirect('/');
});

app.listen(3000);
```

**views/index.ejs:**
```ejs
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <%- include('partials/header') %>
  
  <main>
    <h1>সব পোস্ট</h1>
    
    <a href="/new">নতুন পোস্ট লিখুন</a>
    
    <% posts.forEach(post => { %>
      <article>
        <h2><a href="/post/<%= post.id %>"><%= post.title %></a></h2>
        <p><%= post.content.substring(0, 100) %>...</p>
        <small>লেখক: <%= post.author %></small>
      </article>
    <% }) %>
  </main>
  
  <%- include('partials/footer') %>
</body>
</html>
```

## Best Practices

### ✅ ১. Use Layouts
```javascript
// Avoid repeating HTML structure
```

### ✅ ২. Partials for Reusable Components
```ejs
<%- include('partials/navbar') %>
```

### ✅ ৩. Pass Data Consistently
```javascript
res.render('page', {
  title: 'Page Title',
  user: req.user,
  csrfToken: req.csrfToken()
});
```

## রেফারেন্স

- **EJS**: https://ejs.co/
- **Pug**: https://pugjs.org/
- **Handlebars**: https://handlebarsjs.com/

---

**পরবর্তী Chapter**: Error Handling
