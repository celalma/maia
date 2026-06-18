Absolutely — here are the **four files** you asked for:  
✅ `style.css`  
✅ `app.js`  
✅ `webrtc.js`  
✅ `logo.svg`  

All written so you can **copy → paste → upload**.

These files match the UI style you wanted:  
**light theme, warm tones, clean Copilot/Claude‑like layout.**

---

# 🎨 `assets/css/style.css`

```css
/* ------------------------------
   Maia — Light Warm UI
   Clean, soft, Copilot-like
--------------------------------*/

:root {
    --bg: #faf7f3;
    --bg-light: #ffffff;
    --border: #e5ded6;
    --text: #2b2b2b;
    --text-light: #6a6a6a;
    --accent: #e8c48c;
    --accent-dark: #d3a96a;
    --accent-soft: #f3e4c8;
    --radius: 10px;
    --shadow: 0 2px 6px rgba(0,0,0,0.08);
    --sidebar-width: 260px;
    font-family: "Segoe UI", sans-serif;
}

body {
    margin: 0;
    background: var(--bg);
    color: var(--text);
}

/* ------------------------------
   Auth pages
--------------------------------*/

.auth-body {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.auth-card {
    background: var(--bg-light);
    padding: 30px;
    width: 360px;
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    text-align: center;
}

.auth-logo img {
    width: 60px;
    opacity: 0.9;
}

.auth-logo h1 {
    margin: 10px 0 0;
}

.subtitle {
    color: var(--text-light);
    margin-bottom: 20px;
}

.auth-form label {
    display: block;
    text-align: left;
    margin-bottom: 10px;
}

.auth-form input {
    width: 100%;
    padding: 10px;
    border: 1px solid var(--border);
    border-radius: var(--radius);
    margin-top: 4px;
}

.btn-primary {
    width: 100%;
    padding: 10px;
    background: var(--accent);
    border: none;
    border-radius: var(--radius);
    cursor: pointer;
    margin-top: 15px;
    font-weight: bold;
}

.btn-primary:hover {
    background: var(--accent-dark);
}

.auth-error {
    background: #ffe0e0;
    padding: 10px;
    border-radius: var(--radius);
    margin-bottom: 10px;
}

.auth-success {
    background: #e0ffe7;
    padding: 10px;
    border-radius: var(--radius);
    margin-bottom: 10px;
}

.auth-switch {
    margin-top: 15px;
}

/* ------------------------------
   Dashboard layout
--------------------------------*/

.app-body {
    display: flex;
    height: 100vh;
}

/* Sidebar */
.sidebar {
    width: var(--sidebar-width);
    background: var(--bg-light);
    border-right: 1px solid var(--border);
    padding: 20px;
    box-sizing: border-box;
}

.sidebar-header {
    display: flex;
    align-items: center;
    gap: 10px;
}

.logo-small {
    width: 32px;
}

.user-info {
    margin: 20px 0;
    padding: 10px;
    background: var(--accent-soft);
    border-radius: var(--radius);
}

.friend-code {
    display: block;
    font-size: 12px;
    color: var(--text-light);
}

.logout-btn {
    display: inline-block;
    margin-top: 8px;
    font-size: 13px;
    color: #b33;
}

/* Lists */
.list {
    max-height: 200px;
    overflow-y: auto;
    border: 1px solid var(--border);
    border-radius: var(--radius);
    padding: 5px;
    background: #fff;
}

.list-item {
    padding: 8px;
    border-radius: var(--radius);
    cursor: pointer;
}

.list-item:hover {
    background: var(--accent-soft);
}

/* Chat area */
.chat-area {
    flex: 1;
    display: flex;
    flex-direction: column;
}

.chat-header {
    padding: 15px;
    border-bottom: 1px solid var(--border);
    background: var(--bg-light);
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.btn-call {
    padding: 8px 12px;
    border-radius: var(--radius);
    background: var(--accent-soft);
    border: 1px solid var(--accent);
    cursor: pointer;
}

.chat-messages {
    flex: 1;
    padding: 20px;
    overflow-y: auto;
}

.message {
    margin-bottom: 12px;
    padding: 10px;
    background: var(--bg-light);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    max-width: 60%;
}

.message.me {
    background: var(--accent-soft);
    margin-left: auto;
}

.chat-input {
    display: flex;
    padding: 15px;
    border-top: 1px solid var(--border);
    background: var(--bg-light);
}

.chat-input input {
    flex: 1;
    padding: 10px;
    border: 1px solid var(--border);
    border-radius: var(--radius);
}

.chat-input button {
    margin-left: 10px;
    padding: 10px 20px;
}

/* Modals */
.modal {
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.3);
    display: flex;
    justify-content: center;
    align-items: center;
}

.modal.hidden {
    display: none;
}

.modal-content {
    background: var(--bg-light);
    padding: 20px;
    border-radius: var(--radius);
    width: 300px;
    box-shadow: var(--shadow);
}
```

---

# ⚡ `assets/js/app.js`

```javascript
// -----------------------------
// Maia — Main App Logic
// -----------------------------

let currentFriend = null;
let currentGroup = null;

function $(id) {
    return document.getElementById(id);
}

// -----------------------------
// Load friends & groups
// -----------------------------

async function loadFriends() {
    const res = await fetch("api.php?action=get_friends");
    const data = await res.json();

    const list = $("friends-list");
    list.innerHTML = "";

    data.friends.forEach(f => {
        const div = document.createElement("div");
        div.className = "list-item";
        div.textContent = f.display_name;
        div.onclick = () => openChatWithFriend(f.id, f.display_name);
        list.appendChild(div);
    });
}

async function loadGroups() {
    const res = await fetch("api.php?action=get_groups");
    const data = await res.json();

    const list = $("groups-list");
    list.innerHTML = "";

    data.groups.forEach(g => {
        const div = document.createElement("div");
        div.className = "list-item";
        div.textContent = g.name;
        div.onclick = () => openChatWithGroup(g.id, g.name);
        list.appendChild(div);
    });
}

// -----------------------------
// Chat handling
// -----------------------------

async function openChatWithFriend(id, name) {
    currentFriend = id;
    currentGroup = null;

    $("chat-title").textContent = name;
    $("message-input").disabled = false;
    document.querySelector(".chat-input button").disabled = false;
    $("call-btn").disabled = false;

    loadMessages();
}

async function openChatWithGroup(id, name) {
    currentGroup = id;
    currentFriend = null;

    $("chat-title").textContent = name;
    $("message-input").disabled = false;
    document.querySelector(".chat-input button").disabled = false;
    $("call-btn").disabled = true; // group calls disabled for now

    loadMessages();
}

async function loadMessages() {
    if (!currentFriend && !currentGroup) return;

    const url = currentFriend
        ? `api.php?action=get_messages&friend_id=${currentFriend}`
        : `api.php?action=get_messages&group_id=${currentGroup}`;

    const res = await fetch(url);
    const data = await res.json();

    const box = $("chat-messages");
    box.innerHTML = "";

    data.messages.forEach(m => {
        const div = document.createElement("div");
        div.className = "message";
        if (m.from_user_id == window.myUserId) div.classList.add("me");
        div.textContent = m.content;
        box.appendChild(div);
    });

    box.scrollTop = box.scrollHeight;
}

async function sendMessage() {
    const input = $("message-input");
    const text = input.value.trim();
    if (!text) return;

    const form = new FormData();
    form.append("action", "send_message");
    form.append("content", text);

    if (currentFriend) form.append("friend_id", currentFriend);
    if (currentGroup) form.append("group_id", currentGroup);

    await fetch("api.php", { method: "POST", body: form });

    input.value = "";
    loadMessages();
}

// -----------------------------
// Friend modal
// -----------------------------

function openFriendModal() {
    $("friend-modal").classList.remove("hidden");
}

function closeFriendModal() {
    $("friend-modal").classList.add("hidden");
}

async function sendFriendRequest() {
    const code = $("friend-code-input").value.trim();
    if (!code) return;

    const form = new FormData();
    form.append("action", "send_friend_request");
    form.append("friend_code", code);

    await fetch("api.php", { method: "POST", body: form });

    closeFriendModal();
}

// -----------------------------
// Group modal
// -----------------------------

function openGroupModal() {
    $("group-modal").classList.remove("hidden");
}

function closeGroupModal() {
    $("group-modal").classList.add("hidden");
}

async function createGroup() {
    const name = $("group-name-input").value.trim();
    if (!name) return;

    const form = new FormData();
    form.append("action", "create_group");
    form.append("name", name);

    await fetch("api.php", { method: "POST", body: form });

    closeGroupModal();
    loadGroups();
}

// -----------------------------
// Polling
// -----------------------------

setInterval(() => {
    loadFriends();
    loadGroups();
    loadMessages();
}, 2000);

// Initial load
loadFriends();
loadGroups();
```

---

# 📞 `assets/js/webrtc.js`

```javascript
// -----------------------------------
// Maia — Simple WebRTC Voice Calls
// -----------------------------------

let peer = null;
let localStream = null;

async function startCall() {
    if (!currentFriend) return;

    peer = new RTCPeerConnection();

    localStream = await navigator.mediaDevices.getUserMedia({ audio: true });
    localStream.getTracks().forEach(t => peer.addTrack(t, localStream));

    peer.onicecandidate = e => {
        if (e.candidate) sendSignal(JSON.stringify({ ice: e.candidate }));
    };

    peer.ontrack = e => {
        const audio = new Audio();
        audio.srcObject = e.streams[0];
        audio.play();
    };

    const offer = await peer.createOffer();
    await peer.setLocalDescription(offer);

    sendSignal(JSON.stringify({ offer }));
}

async function sendSignal(payload) {
    const form = new FormData();
    form.append("action", "send_signal");
    form.append("to_user_id", currentFriend);
    form.append("payload", payload);

    await fetch("api.php", { method: "POST", body: form });
}

async function checkSignals() {
    const res = await fetch("api.php?action=get_signals");
    const data = await res.json();

    for (const s of data.signals) {
        const msg = JSON.parse(s.payload);

        if (msg.offer) {
            // Incoming call
            peer = new RTCPeerConnection();

            localStream = await navigator.mediaDevices.getUserMedia({ audio: true });
            localStream.getTracks().forEach(t => peer.addTrack(t, localStream));

            peer.onicecandidate = e => {
                if (e.candidate) sendSignal(JSON.stringify({ ice: e.candidate }));
            };

            peer.ontrack = e => {
                const audio = new Audio();
                audio.srcObject = e.streams[0];
                audio.play();
            };

            await peer.setRemoteDescription(msg.offer);
            const answer = await peer.createAnswer();
            await peer.setLocalDescription(answer);

            sendSignal(JSON.stringify({ answer }));
        }

        if (msg.answer) {
            await peer.setRemoteDescription(msg.answer);
        }

        if (msg.ice) {
            try {
                await peer.addIceCandidate(msg.ice);
            } catch {}
        }
    }
}

setInterval(checkSignals, 1500);
```

---

# 🖼️ `assets/img/logo.svg`

A simple warm‑tone Maia logo.

```svg
<svg width="80" height="80" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
  <circle cx="50" cy="50" r="45" fill="#f3e4c8" stroke="#d3a96a" stroke-width="4"/>
  <text x="50" y="58" font-size="32" text-anchor="middle" fill="#8a6a3a" font-family="Segoe UI">M</text>
</svg>
```

---

# 🎉 All files delivered  
You now have:

- Full backend  
- Full frontend  
- Full UI  
- Voice calls  
- Warm, clean design  

If you want, I can now help you:

- Test it  
- Upload it  
- Fix errors  
- Add features (typing indicator, avatars, dark mode, etc.)

Just tell me what you want next.
