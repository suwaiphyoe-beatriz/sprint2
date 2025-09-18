#### authController.js — Secure Login with Password Check
Before: 

```javascript
const login = async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  res.json({ user });
};
```

After: 

```javascript
const user = await User.findOne({ email });
if (!user || !(await user.comparePassword(password))) {
  return res.status(401).json({ message: 'Invalid email or password' });
}
```

Improvement:
- Validates credentials securely.
- Prevents unauthorized access

#### authController.js — Adding Input Validation

Before:

```javascript
const register = async (req, res) => {
  const { username, email, password } = req.body;
  const user = await User.create({ username, email, password });
  res.status(201).json({ user });
};
```

After:

```javascript
const { validationResult } = require('express-validator');

const register = async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ message: 'Validation failed', errors: errors.array() });
  }
  // ...create user
};
```
Improvement:
- Prevents invalid input and improves API reliability.
- Returns clear error messages to the client.

#### Routes/receipes.js — Fixing Route Order

Before:

```javascript
router.get('/:id', getRecipeById);
router.get('/search', searchRecipes);
```

After:

```javascript
router.get('/search', searchRecipes); // static route first
router.get('/:id', getRecipeById);    // dynamic route last
```

Improvement:
- Ensures /search is correctly handled.
- Prevents dynamic route from intercepting static route

#### server.js — Session Security with Cookies

Before:

```javascript
app.use(session({ secret: 'key' }));
```

After:

```javascript
app.use(session({
  secret: process.env.SESSION_SECRET,
  cookie: { httpOnly: true, secure: process.env.NODE_ENV === 'production', sameSite: 'lax' }
}));
```

Improvement:
- Cookies are secure and HTTP-only.
- Reduces risk of XSS and CSRF attacks.

#### authController.js — Check Existing User Before Registration

Before:

```javascript
const user = await User.create({ username, email, password });
```

After:

```javascript
const existingUser = await User.findOne({ $or: [{ email }, { username }] });
if (existingUser) {
  return res.status(409).json({ message: 'User already exists' });
}
```

Improvement:
- Prevents duplicate usernames or emails.
- Returns meaningful HTTP status (409 Conflict).

#### recipeController.js — Search Recipes by Title

Before:

```javascript
const recipes = await Recipe.find({ title: req.query.q });
```

After:

```javascript
const query = req.query.q || '';
const recipes = await Recipe.find({ title: { $regex: query, $options: 'i' } });
```

Improvement:
- Supports partial and case-insensitive search.
- Returns all recipes if the query is empty

#### recipeRoutes.js — Protected Bookmark Endpoint

Before:

```javascript
// No authentication
router.patch('/:id/bookmark', toggleBookmark);
```

After:
```javascript
router.patch('/:id/bookmark', authenticateToken, toggleBookmark);
```

Improvement:
- Only logged-in users can bookmark recipes.
- Prevents anonymous or malicious bookmark updates