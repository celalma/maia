# 🌼 **Maia — A Lightweight Chat & Voice App (PHP + SQLite)**

**Maia** is a simple, warm‑themed chat application inspired by Discord and Snapchat, built for small communities, friends, and gaming groups.  
It focuses on clarity, simplicity, and a clean UI — similar to Claude or Microsoft Copilot, but adapted for real‑time chatting.

Maia is named after **Maia**, the mother of Hermes — the Greek god of messages and communication.

---

## ✨ Features

### 👤 **Accounts & Identity**
- User registration & login  
- Display name + unique friend code (`MAIA-1234`)  
- Lightweight presence system (online / recently active)

### 🤝 **Friends**
- Add friends using their friend code  
- Accept or decline friend requests  
- One‑to‑one private chats  

### 👥 **Groups**
- Create groups  
- Invite friends  
- Group chat with multiple members  

### 💬 **Messaging**
- Direct messages  
- Group messages  
- Auto‑refreshing chat (polling every 2 seconds)  
- Clean, warm, modern UI  

### 📞 **Voice Calls (WebRTC)**
- One‑to‑one audio calls  
- Browser‑based (no plugins)  
- Simple signaling handled through PHP  

### 🎨 **UI / UX**
- Light, warm theme  
- Soft colors and rounded corners  
- Sidebar navigation  
- Modal dialogs for adding friends & creating groups  

---

## 🧱 Tech Stack

| Layer | Technology |
|-------|------------|
| Frontend | HTML, CSS, Vanilla JavaScript |
| Backend | PHP (no frameworks) |
| Database | SQLite |
| Voice Calls | WebRTC (peer‑to‑peer) |
| Communication | AJAX polling |

No external dependencies.  
No Node.js.  
No Composer.  
Just simple, portable PHP.

---

## 📁 Project Structure

```
maia/
│
├── index.php
├── login.php
├── register.php
├── logout.php
├── dashboard.php
├── api.php
├── db_init.php
├── config.php
│
├── assets/
│   ├── css/
│   │   └── style.css
│   ├── js/
│   │   ├── app.js
│   │   └── webrtc.js
│   └── img/
│       └── logo.svg
│
└── db.sqlite   (created automatically)
```

---

## 🚀 Installation (Beginner‑Friendly)

### 1. Upload the files  
Use FileZilla (or any FTP client) to upload the project folder to your hosting space.

### 2. Run the database setup  
Open this in your browser:

```
https://yourdomain.com/db_init.php
```

This creates `db.sqlite` and all required tables.

### 3. Start using Maia  
Visit:

```
https://yourdomain.com/
```

You’ll see the login page.  
Create an account and start chatting.

---

## 🧑‍💻 How Users Use Maia

### 1. Create an account  
Users choose:
- Username  
- Display name  
- Password  

They automatically receive a **friend code** like `MAIA-4821`.

### 2. Add friends  
Users share their friend code.  
Friend requests appear in the sidebar.

### 3. Chat  
Click a friend or group to open the chat.  
Messages appear instantly thanks to lightweight polling.

### 4. Create groups  
Users can create a group, name it, and invite friends.

### 5. Voice call  
In a direct chat, click **📞 Call** to start a WebRTC audio call.

---

## 🔒 Security Notes

- Passwords are hashed using `password_hash()`  
- SQLite is file‑based; ensure your hosting prevents direct downloads  
- WebRTC requires **HTTPS**  
- This is a lightweight demo app — not intended for enterprise use  

---

## 🛠️ Customization

You can easily modify:
- Colors in `style.css`  
- UI layout in `dashboard.php`  
- Polling speed in `app.js`  
- WebRTC behavior in `webrtc.js`  

Maia is intentionally simple so you can learn from it and extend it.

---

## ❤️ Contributing

Feel free to open issues or submit pull requests.  
Maia is meant to be a friendly, beginner‑friendly project.

---

## 📜 License

MIT License — free to use, modify, and share.
