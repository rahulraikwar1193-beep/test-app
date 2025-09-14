आईपी# test-app
मैं यह सब टेस्ट करने के लिए लिख रहाहूं
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Simple Messenger (Firebase)</title>
  <style>
    body{font-family: Arial; max-width:700px;margin:20px auto;}
    #messages{border:1px solid #ddd; height:400px; overflow:auto;padding:10px;}
    .msg{margin:8px 0;padding:6px;border-radius:6px;background:#f1f1f1;}
    .me{background:#d1ffd1;text-align:right;}
    #inputRow{display:flex; gap:8px; margin-top:10px;}
    input[type="text"]{flex:1;padding:8px;}
  </style>
</head>
<body>
  <h2>Simple Messenger (Firebase)</h2>
  <div id="status">Connecting...</div>
  <div id="messages"></div>

  <div id="inputRow">
    <input id="text" type="text" placeholder="Type a message" />
    <button id="sendBtn">Send</button>
  </div>

  <!-- Firebase SDKs (modular) -->
  <script type="module">
    // ====== Paste your Firebase config here (from Firebase Console) ======
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_AUTH_DOMAIN",
      projectId: "YOUR_PROJECT_ID",
      // ... rest of config
    };
    // ===================================================================

    import { initializeApp } from "https://www.gstatic.com/firebasejs/9.24.0/firebase-app.js";
    import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/9.24.0/firebase-auth.js";
    import { getFirestore, collection, addDoc, query, orderBy, onSnapshot, serverTimestamp } from "https://www.gstatic.com/firebasejs/9.24.0/firebase-firestore.js";

    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    const statusEl = document.getElementById('status');
    const messagesEl = document.getElementById('messages');
    const textInput = document.getElementById('text');
    const sendBtn = document.getElementById('sendBtn');

    // Sign in anonymously
    signInAnonymously(auth)
      .catch(err => statusEl.textContent = 'Auth error: ' + err.message);

    let myUid = null;
    onAuthStateChanged(auth, user => {
      if (user) {
        myUid = user.uid;
        statusEl.textContent = 'Connected as: ' + myUid.slice(0,6);
        startListening();
      } else {
        statusEl.textContent = 'Signed out';
      }
    });

    // Firestore collection 'messages'
    const messagesCol = collection(db, 'messages');

    function startListening(){
      const q = query(messagesCol, orderBy('createdAt'));
      onSnapshot(q, snapshot => {
        messagesEl.innerHTML = '';
        snapshot.forEach(doc => {
          const d = doc.data();
          const div = document.createElement('div');
          div.className = 'msg' + (d.uid === myUid ? ' me' : '');
          const time = d.createdAt && d.createdAt.toDate ? d.createdAt.toDate().toLocaleTimeString() : '';
          div.innerHTML = `<small>${escapeHtml(d.name || d.uid.slice(0,6))} ${time}</small><div>${escapeHtml(d.text)}</div>`;
          messagesEl.appendChild(div);
        });
        messagesEl.scrollTop = messagesEl.scrollHeight;
      }, err => {
        console.error('listen error', err);
      });
    }

    sendBtn.addEventListener('click', async () => {
      const text = textInput.value.trim();
      if (!text) return;
      try {
        await addDoc(messagesCol, {
          text,
          uid: myUid,
          name: 'User-' + myUid.slice(0,6),
          createdAt: serverTimestamp()
        });
        textInput.value = '';
      } catch (e) {
        console.error(e);
        alert('Send failed');
      }
    });

    textInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') sendBtn.click();
    });

    // tiny helper
    function escapeHtml(s){ return String(s).replaceAll('&','&amp;').replaceAll('<','&lt;').replaceAll('>','&gt;'); }
  </script>
</body>
</html>
