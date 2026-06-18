Great — here comes the **next major file**, the heart of Maia:

# 🌼 `dashboard.php`  
This is the main interface:  
- Friends list  
- Groups list  
- Chat window  
- Message input  
- Call button  
- Clean, warm, Copilot‑like layout  

It’s intentionally simple so you can understand and customize it.

---

# 📄 **dashboard.php**  
Copy this into `dashboard.php` in your Maia folder:

```php
<?php
require_once 'config.php';
require_login();

// Fetch user info
$user_id = $_SESSION['user_id'];
$display_name = $_SESSION['display_name'];
$friend_code = $_SESSION['friend_code'];

// Update presence
$db->prepare("
    INSERT INTO presence (user_id, last_active)
    VALUES (:id, datetime('now'))
    ON CONFLICT(user_id) DO UPDATE SET last_active = excluded.last_active
")->execute([':id' => $user_id]);

?>
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Maia · Dashboard</title>
    <link rel="stylesheet" href="assets/css/style.css">
</head>
<body class="app-body">

    <!-- Sidebar -->
    <aside class="sidebar">
        <div class="sidebar-header">
            <img src="assets/img/logo.svg" class="logo-small">
            <h2>Maia</h2>
        </div>

        <div class="user-info">
            <strong><?= htmlspecialchars($display_name) ?></strong>
            <span class="friend-code"><?= htmlspecialchars($friend_code) ?></span>
            <a href="logout.php" class="logout-btn">Logout</a>
        </div>

        <div class="sidebar-section">
            <h3>Friends</h3>
            <div id="friends-list" class="list"></div>
            <button class="btn-secondary" onclick="openFriendModal()">Add friend</button>
        </div>

        <div class="sidebar-section">
            <h3>Groups</h3>
            <div id="groups-list" class="list"></div>
            <button class="btn-secondary" onclick="openGroupModal()">New group</button>
        </div>
    </aside>

    <!-- Chat area -->
    <main class="chat-area">
        <div class="chat-header">
            <h2 id="chat-title">Select a conversation</h2>
            <button id="call-btn" class="btn-call" onclick="startCall()" disabled>📞 Call</button>
        </div>

        <div id="chat-messages" class="chat-messages">
            <div class="placeholder">Choose a friend or group to start chatting</div>
        </div>

        <div class="chat-input">
            <input id="message-input" type="text" placeholder="Type a message…" disabled>
            <button class="btn-primary" onclick="sendMessage()" disabled>Send</button>
        </div>
    </main>

    <!-- Friend modal -->
    <div id="friend-modal" class="modal hidden">
        <div class="modal-content">
            <h2>Add a friend</h2>
            <input id="friend-code-input" type="text" placeholder="Enter friend code (MAIA-1234)">
            <button class="btn-primary" onclick="sendFriendRequest()">Send request</button>
            <button class="btn-secondary" onclick="closeFriendModal()">Cancel</button>
        </div>
    </div>

    <!-- Group modal -->
    <div id="group-modal" class="modal hidden">
        <div class="modal-content">
            <h2>Create a group</h2>
            <input id="group-name-input" type="text" placeholder="Group name">
            <button class="btn-primary" onclick="createGroup()">Create</button>
            <button class="btn-secondary" onclick="closeGroupModal()">Cancel</button>
        </div>
    </div>

    <script src="assets/js/app.js"></script>
    <script src="assets/js/webrtc.js"></script>
</body>
</html>
```

---

# 🌞 What this page does  
This is the **UI skeleton**:

### Left sidebar  
- Shows your name + friend code  
- Shows friends  
- Shows groups  
- Buttons to add friends / create groups  

### Right side  
- Chat messages  
- Message input  
- Call button (WebRTC)  

### Hidden modals  
- Add friend  
- Create group  

### JS hooks  
Everything interactive is handled by:
- `app.js` (chat, friends, groups, polling)
- `webrtc.js` (voice calls)

---

# 🌼 Next files coming  
Now that the UI exists, the next pieces are:

### **Next file I’ll send:**  
👉 `api.php` — the backend that handles everything (messages, friends, groups, presence, signaling)

After that:  
- `style.css` (warm, clean Copilot‑like UI)  
- `app.js`  
- `webrtc.js`  
- `logo.svg`

---

Just say **“next”** and I’ll send `api.php`.
