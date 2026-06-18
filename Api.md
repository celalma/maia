Alright — here comes the **engine** of Maia:  
the file that makes *everything* work behind the scenes.

This is **`api.php`**, the backend endpoint your JavaScript will call for:

- loading friends  
- loading groups  
- sending messages  
- fetching messages  
- friend requests  
- creating groups  
- presence updates  
- WebRTC signaling (for voice calls)

It’s intentionally simple and readable.

---

# ⚙️ **api.php**  
Create a file named **`api.php`** in your Maia folder and paste this:

```php
<?php
require_once 'config.php';
header('Content-Type: application/json');

if (!isset($_SESSION['user_id'])) {
    echo json_encode(['error' => 'not_logged_in']);
    exit;
}

$user_id = $_SESSION['user_id'];
$action = $_GET['action'] ?? $_POST['action'] ?? '';

function respond($data) {
    echo json_encode($data);
    exit;
}

switch ($action) {

    /* -------------------------
       FRIENDS
    --------------------------*/

    case 'get_friends':
        $stmt = $db->prepare("
            SELECT u.id, u.display_name, u.friend_code,
                   (SELECT last_active FROM presence WHERE user_id = u.id) AS last_active
            FROM friends f
            JOIN users u ON u.id = f.friend_id
            WHERE f.user_id = :id
        ");
        $stmt->execute([':id' => $user_id]);
        respond(['friends' => $stmt->fetchAll(PDO::FETCH_ASSOC)]);
        break;

    case 'send_friend_request':
        $code = trim($_POST['friend_code'] ?? '');

        $stmt = $db->prepare("SELECT id FROM users WHERE friend_code = :fc");
        $stmt->execute([':fc' => $code]);
        $target = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$target) respond(['error' => 'no_user']);
        if ($target['id'] == $user_id) respond(['error' => 'self']);

        // Check existing
        $stmt = $db->prepare("
            SELECT id FROM friend_requests
            WHERE from_user_id = :me AND to_user_id = :them AND status = 'pending'
        ");
        $stmt->execute([':me' => $user_id, ':them' => $target['id']]);
        if ($stmt->fetch()) respond(['error' => 'already_sent']);

        $stmt = $db->prepare("
            INSERT INTO friend_requests (from_user_id, to_user_id)
            VALUES (:me, :them)
        ");
        $stmt->execute([':me' => $user_id, ':them' => $target['id']]);

        respond(['success' => true]);
        break;

    case 'get_friend_requests':
        $stmt = $db->prepare("
            SELECT fr.id, u.display_name, u.friend_code
            FROM friend_requests fr
            JOIN users u ON u.id = fr.from_user_id
            WHERE fr.to_user_id = :id AND fr.status = 'pending'
        ");
        $stmt->execute([':id' => $user_id]);
        respond(['requests' => $stmt->fetchAll(PDO::FETCH_ASSOC)]);
        break;

    case 'accept_friend':
        $req_id = intval($_POST['request_id'] ?? 0);

        // Get request
        $stmt = $db->prepare("
            SELECT * FROM friend_requests
            WHERE id = :id AND to_user_id = :me
        ");
        $stmt->execute([':id' => $req_id, ':me' => $user_id]);
        $req = $stmt->fetch(PDO::FETCH_ASSOC);

        if (!$req) respond(['error' => 'not_found']);

        // Add both ways
        $db->prepare("INSERT INTO friends (user_id, friend_id) VALUES (:a, :b)")
           ->execute([':a' => $user_id, ':b' => $req['from_user_id']]);
        $db->prepare("INSERT INTO friends (user_id, friend_id) VALUES (:a, :b)")
           ->execute([':a' => $req['from_user_id'], ':b' => $user_id]);

        // Mark request accepted
        $db->prepare("UPDATE friend_requests SET status='accepted' WHERE id=:id")
           ->execute([':id' => $req_id]);

        respond(['success' => true]);
        break;

    /* -------------------------
       GROUPS
    --------------------------*/

    case 'get_groups':
        $stmt = $db->prepare("
            SELECT g.id, g.name
            FROM group_members gm
            JOIN groups g ON g.id = gm.group_id
            WHERE gm.user_id = :id
        ");
        $stmt->execute([':id' => $user_id]);
        respond(['groups' => $stmt->fetchAll(PDO::FETCH_ASSOC)]);
        break;

    case 'create_group':
        $name = trim($_POST['name'] ?? '');
        if ($name === '') respond(['error' => 'empty']);

        $stmt = $db->prepare("
            INSERT INTO groups (name, owner_id)
            VALUES (:n, :o)
        ");
        $stmt->execute([':n' => $name, ':o' => $user_id]);
        $group_id = $db->lastInsertId();

        // Add creator as member
        $db->prepare("
            INSERT INTO group_members (group_id, user_id)
            VALUES (:g, :u)
        ")->execute([':g' => $group_id, ':u' => $user_id]);

        respond(['success' => true, 'group_id' => $group_id]);
        break;

    /* -------------------------
       MESSAGES
    --------------------------*/

    case 'get_messages':
        $friend = intval($_GET['friend_id'] ?? 0);
        $group = intval($_GET['group_id'] ?? 0);

        if ($friend) {
            // Direct messages
            $stmt = $db->prepare("
                SELECT * FROM messages
                WHERE (from_user_id = :me AND to_user_id = :them)
                   OR (from_user_id = :them AND to_user_id = :me)
                ORDER BY id ASC
            ");
            $stmt->execute([':me' => $user_id, ':them' => $friend]);
        } else {
            // Group messages
            $stmt = $db->prepare("
                SELECT * FROM messages
                WHERE group_id = :g
                ORDER BY id ASC
            ");
            $stmt->execute([':g' => $group]);
        }

        respond(['messages' => $stmt->fetchAll(PDO::FETCH_ASSOC)]);
        break;

    case 'send_message':
        $content = trim($_POST['content'] ?? '');
        $friend = intval($_POST['friend_id'] ?? 0);
        $group = intval($_POST['group_id'] ?? 0);

        if ($content === '') respond(['error' => 'empty']);

        $stmt = $db->prepare("
            INSERT INTO messages (from_user_id, to_user_id, group_id, content)
            VALUES (:from, :to, :group, :content)
        ");
        $stmt->execute([
            ':from' => $user_id,
            ':to' => $friend ?: null,
            ':group' => $group ?: null,
            ':content' => $content
        ]);

        respond(['success' => true]);
        break;

    /* -------------------------
       PRESENCE
    --------------------------*/

    case 'update_presence':
        $db->prepare("
            INSERT INTO presence (user_id, last_active)
            VALUES (:id, datetime('now'))
            ON CONFLICT(user_id) DO UPDATE SET last_active = excluded.last_active
        ")->execute([':id' => $user_id]);

        respond(['success' => true]);
        break;

    /* -------------------------
       WEBRTC SIGNALING
    --------------------------*/

    case 'send_signal':
        $to = intval($_POST['to_user_id'] ?? 0);
        $payload = $_POST['payload'] ?? '';

        $stmt = $db->prepare("
            INSERT INTO signaling (from_user_id, to_user_id, payload)
            VALUES (:f, :t, :p)
        ");
        $stmt->execute([':f' => $user_id, ':t' => $to, ':p' => $payload]);

        respond(['success' => true]);
        break;

    case 'get_signals':
        $stmt = $db->prepare("
            SELECT * FROM signaling
            WHERE to_user_id = :me
            ORDER BY id ASC
        ");
        $stmt->execute([':me' => $user_id]);
        $rows = $stmt->fetchAll(PDO::FETCH_ASSOC);

        // Delete after reading
        $db->prepare("DELETE FROM signaling WHERE to_user_id = :me")
           ->execute([':me' => $user_id]);

        respond(['signals' => $rows]);
        break;

    default:
        respond(['error' => 'unknown_action']);
}
```

---

# 🌼 What’s next?

Now that the backend exists, the next files are:

### 👉 **`style.css`**  
The warm, clean, Copilot‑like UI you asked for.

### 👉 **`app.js`**  
Handles:
- loading friends  
- loading groups  
- selecting chats  
- sending messages  
- polling for updates  
- opening modals  

### 👉 **`webrtc.js`**  
Handles:
- starting a call  
- answering a call  
- exchanging WebRTC signals via `api.php`  

### 👉 **`logo.svg`**  
A simple Maia logo.

---

Just say **“next”** and I’ll send the **CSS (style.css)** — the part that makes Maia look beautiful.
