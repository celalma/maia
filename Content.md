Nice, this is going to be fun. We’ll build **Maia** piece by piece so you can literally copy–paste files and upload them.

I’ll start with:

- Folder structure  
- Core PHP pages  
- Then CSS + JS  
- Then we refine

---

### 📁 1. Folder structure

On your computer, create a main folder, for example:

`maia/`

Inside it, create this structure:

- `index.php`
- `login.php`
- `register.php`
- `logout.php`
- `dashboard.php`
- `api.php`
- `db_init.php`
- `config.php`
- `assets/`
  - `css/`
    - `style.css`
  - `js/`
    - `app.js`
    - `webrtc.js`
  - `img/`
    - `logo.svg`

You’ll fill each file with the code I give you.

---

## ⚙️ 2. `config.php` (shared settings)

Create `config.php` in the root:

```php
<?php
// config.php
// Shared configuration for Maia

session_start();

// Path to SQLite database file
$db_file = __DIR__ . '/db.sqlite';

try {
    $db = new PDO('sqlite:' . $db_file);
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
} catch (Exception $e) {
    die('Database connection error: ' . htmlspecialchars($e->getMessage()));
}

// Simple helper: require login
function require_login() {
    if (!isset($_SESSION['user_id'])) {
        header('Location: login.php');
        exit;
    }
}
```

---

## 🧱 3. `db_init.php` (create database)

Create `db_init.php`:

```php
<?php
// db_init.php
// Run this ONCE in your browser to create the database

$db_file = __DIR__ . '/db.sqlite';

if (file_exists($db_file)) {
    echo "Database already exists at db.sqlite<br>";
    echo "If you want to reset it, delete db.sqlite and reload this page.";
    exit;
}

try {
    $db = new PDO('sqlite:' . $db_file);
    $db->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

    // Users
    $db->exec("
        CREATE TABLE users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            username TEXT NOT NULL,
            display_name TEXT NOT NULL,
            password_hash TEXT NOT NULL,
            friend_code TEXT NOT NULL UNIQUE,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
    ");

    // Friend requests
    $db->exec("
        CREATE TABLE friend_requests (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            from_user_id INTEGER NOT NULL,
            to_user_id INTEGER NOT NULL,
            status TEXT NOT NULL DEFAULT 'pending',
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
    ");

    // Friendships
    $db->exec("
        CREATE TABLE friends (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER NOT NULL,
            friend_id INTEGER NOT NULL
        );
    ");

    // Groups
    $db->exec("
        CREATE TABLE groups (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            owner_id INTEGER NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
    ");

    // Group members
    $db->exec("
        CREATE TABLE group_members (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            group_id INTEGER NOT NULL,
            user_id INTEGER NOT NULL
        );
    ");

    // Messages (direct + group)
    $db->exec("
        CREATE TABLE messages (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            from_user_id INTEGER NOT NULL,
            to_user_id INTEGER,
            group_id INTEGER,
            content TEXT NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
    ");

    // Presence
    $db->exec("
        CREATE TABLE presence (
            user_id INTEGER PRIMARY KEY,
            last_active DATETIME NOT NULL
        );
    ");

    // WebRTC signaling
    $db->exec("
        CREATE TABLE signaling (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            from_user_id INTEGER NOT NULL,
            to_user_id INTEGER NOT NULL,
            payload TEXT NOT NULL,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        );
    ");

    echo "Database created successfully!<br>";
    echo "You can now delete db_init.php (optional) and go to index.php.";
} catch (Exception $e) {
    echo "Error: " . htmlspecialchars($e->getMessage());
}
```

---

## 🏠 4. `index.php` (landing → redirect)

```php
<?php
// index.php
require_once 'config.php';

if (isset($_SESSION['user_id'])) {
    header('Location: dashboard.php');
    exit;
} else {
    header('Location: login.php');
    exit;
}
```

---

## 🔐 5. `register.php`

```php
<?php
require_once 'config.php';

$error = '';
if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $display_name = trim($_POST['display_name'] ?? '');
    $password = $_POST['password'] ?? '';

    if ($username === '' || $display_name === '' || $password === '') {
        $error = 'Please fill in all fields.';
    } else {
        // Generate friend code: MAIA-XXXX
        $friend_code = 'MAIA-' . random_int(1000, 9999);

        $stmt = $db->prepare("SELECT id FROM users WHERE username = :u OR friend_code = :fc");
        $stmt->execute([':u' => $username, ':fc' => $friend_code]);
        if ($stmt->fetch()) {
            $error = 'Username or friend code already exists. Try again.';
        } else {
            $hash = password_hash($password, PASSWORD_DEFAULT);
            $stmt = $db->prepare("
                INSERT INTO users (username, display_name, password_hash, friend_code)
                VALUES (:u, :d, :p, :fc)
            ");
            $stmt->execute([
                ':u' => $username,
                ':d' => $display_name,
                ':p' => $hash,
                ':fc' => $friend_code
            ]);
            header('Location: login.php?registered=1');
            exit;
        }
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Maia · Register</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body class="auth-body">
    <div class="auth-card">
        <div class="auth-logo">
            <img src="assets/img/logo.svg" alt="Maia logo">
            <h1>Maia</h1>
            <p class="subtitle">Built for chatting & gaming</p>
        </div>

        <h2>Create your account</h2>

        <?php if ($error): ?>
            <div class="auth-error"><?= htmlspecialchars($error) ?></div>
        <?php endif; ?>

        <form method="post" class="auth-form">
            <label>
                Username
                <input type="text" name="username" required>
            </label>
            <label>
                Display name
                <input type="text" name="display_name" required>
            </label>
            <label>
                Password
                <input type="password" name="password" required>
            </label>
            <button type="submit" class="btn-primary">Sign up</button>
        </form>

        <p class="auth-switch">
            Already have an account?
            <a href="login.php">Log in</a>
        </p>
    </div>
</body>
</html>
```

---

## 🔑 6. `login.php`

```php
<?php
require_once 'config.php';

$error = '';
$registered = isset($_GET['registered']);

if ($_SERVER['REQUEST_METHOD'] === 'POST') {
    $username = trim($_POST['username'] ?? '');
    $password = $_POST['password'] ?? '';

    if ($username === '' || $password === '') {
        $error = 'Please fill in all fields.';
    } else {
        $stmt = $db->prepare("SELECT * FROM users WHERE username = :u");
        $stmt->execute([':u' => $username]);
        $user = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$user || !password_verify($password, $user['password_hash'])) {
            $error = 'Invalid username or password.';
        } else {
            $_SESSION['user_id'] = $user['id'];
            $_SESSION['display_name'] = $user['display_name'];
            $_SESSION['friend_code'] = $user['friend_code'];

            $db->prepare("
                INSERT INTO presence (user_id, last_active)
                VALUES (:id, datetime('now'))
                ON CONFLICT(user_id) DO UPDATE SET last_active = excluded.last_active
            ")->execute([':id' => $user['id']]);

            header('Location: dashboard.php');
            exit;
        }
    }
}
?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Maia · Login</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body class="auth-body">
    <div class="auth-card">
        <div class="auth-logo">
            <img src="assets/img/logo.svg" alt="Maia logo">
            <h1>Maia</h1>
            <p class="subtitle">Light, warm, and ready to chat</p>
        </div>

        <h2>Welcome back</h2>

        <?php if ($registered): ?>
            <div class="auth-success">Account created! You can log in now.</div>
        <?php endif; ?>

        <?php if ($error): ?>
            <div class="auth-error"><?= htmlspecialchars($error) ?></div>
        <?php endif; ?>

        <form method="post" class="auth-form">
            <label>
                Username
                <input type="text" name="username" required>
            </label>
            <label>
                Password
                <input type="password" name="password" required>
            </label>
            <button type="submit" class="btn-primary">Log in</button>
        </form>

        <p class="auth-switch">
            New here?
            <a href="register.php">Create an account</a>
        </p>
    </div>
</body>
</html>
```

---

## 🚪 7. `logout.php`

```php
<?php
require_once 'config.php';
session_destroy();
header('Location: login.php');
exit;
```

---

Next step I’ll send:

- `dashboard.php` (main UI: friends, groups, chat, call button)  
- `api.php` (simple endpoints for messages, friends, presence, signaling)  
- `assets/css/style.css` (light, warm, Copilot-like)  
- `assets/js/app.js` + `webrtc.js` (polling + basic voice call flow)

If you’re okay so far, tell me and I’ll drop **dashboard.php + CSS** next so you can start seeing the UI.
