<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zia Trading Assistant</title>
    <style>
        body, html { margin: 0; padding: 0; height: 100%; overflow: hidden; background: #001219; font-family: 'Segoe UI', Arial, sans-serif; }
        .ocean { position: fixed; bottom: 0; width: 100%; height: 100%; z-index: -1; }
        .wave { position: absolute; bottom: -50px; width: 200%; height: 300px; background: rgba(0, 150, 199, 0.2); border-radius: 40%; animation: move 12s infinite linear; }
        @keyframes move { 0% { transform: translateX(0) rotate(0deg); } 100% { transform: translateX(-50%) rotate(360deg); } }
        .orb-container { position: absolute; top: 40%; left: 50%; transform: translate(-50%, -50%); display: none; text-align: center; }
        .orb { width: 160px; height: 160px; background: radial-gradient(circle, #e9d8a6, #94d2bd, #0077b6); border-radius: 50%; box-shadow: 0 0 50px #e9d8a6; animation: breathe 4s infinite ease-in-out; }
        @keyframes breathe { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.1); box-shadow: 0 0 80px #94d2bd; } }
        .active-orb { animation: vibrate 0.2s infinite linear !important; }
        @keyframes vibrate { 0% { transform: scale(1.1) translate(2px, 2px); } 50% { transform: scale(1.1) translate(-2px, -2px); } 100% { transform: scale(1.1) translate(2px, -2px); } }
        #login-screen { position: fixed; width: 100%; height: 100%; background: #000; z-index: 2000; display: flex; justify-content: center; align-items: center; }
        .login-box { text-align: center; background: rgba(255,255,255,0.05); padding: 40px; border-radius: 20px; border: 1px solid rgba(255,255,255,0.2); backdrop-filter: blur(10px); }
        .login-box input { display: block; margin: 15px auto; padding: 12px; border-radius: 10px; border: none; width: 240px; }
        .login-box button { background: #e9d8a6; border: none; padding: 12px 40px; border-radius: 25px; font-weight: bold; cursor: pointer; }
        .input-area { position: fixed; bottom: 30px; width: 100%; display: none; flex-direction: column; align-items: center; gap: 10px; }
        .chat-input-container { display: flex; width: 90%; gap: 10px; }
        .chat-input { flex: 1; background: rgba(255, 255, 255, 0.1); border: 1px solid rgba(255,255,255,0.2); padding: 15px; border-radius: 30px; color: white; outline: none; text-align: center; }
        .send-btn { background: #e9d8a6; border: none; padding: 12px 20px; border-radius: 50%; cursor: pointer; font-size: 18px; }
        .options-container { display: flex; gap: 8px; flex-wrap: wrap; justify-content: center; width: 90%; }
        .opt-btn { background: rgba(148, 210, 189, 0.15); border: 1px solid #94d2bd; color: #94d2bd; padding: 8px 15px; border-radius: 20px; font-size: 13px; cursor: pointer; }
    </style>
</head>
<body>

    <div id="login-screen">
        <div class="login-box">
            <h2 style="color: #e9d8a6;">Zia AI Login</h2>
            <input type="text" id="userid" placeholder="User ID">
            <input type="password" id="pass" placeholder="Password">
            <button onclick="handleLogin()">SIGN IN</button>
        </div>
    </div>

    <div class="orb-container" id="app-content">
        <div class="orb" id="ai-orb"></div>
        <p style="color: #94d2bd; margin-top: 15px; font-size: 14px;">Zia is Online</p>
    </div>

    <div class="input-area" id="input-bar">
        <div class="options-container" id="quick-options"></div>
        <div class="chat-input-container">
            <input type="text" id="message" class="chat-input" placeholder="Ask Zia anything...">
            <button class="send-btn" onclick="handleSend()">➤</button>
        </div>
    </div>

    <div class="ocean"><div class="wave"></div></div>

    <script>
        const projectID = '69d296c4e0100add16a65aef';
        let femaleVoice = null;

        // লোড হওয়ার সময় ফিমেল ভয়েস খুঁজে রাখা
        function loadVoices() {
            let voices = window.speechSynthesis.getVoices();
            // বাংলা ফিমেল ভয়েস খোঁজার চেষ্টা (Google বা Microsoft এর ফিমেল ভয়েসগুলো)
            femaleVoice = voices.find(voice => 
                (voice.lang === 'bn-BD' || voice.lang === 'bn-IN') && 
                (voice.name.includes('Female') || voice.name.includes('Google') || voice.name.includes('Microsoft'))
            );
        }
        window.speechSynthesis.onvoiceschanged = loadVoices;

        window.onload = function() {
            loadVoices();
            if(localStorage.getItem('zia_auth') === 'true') showApp();
        }

        function handleLogin() {
            if(document.getElementById('userid').value !== "" && document.getElementById('pass').value !== "") {
                localStorage.setItem('zia_auth', 'true');
                showApp();
            } else alert("Enter ID & Password");
        }

        function showApp() {
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('app-content').style.display = 'block';
            document.getElementById('input-bar').style.display = 'flex';
            speak("Welcome. I am Zia. How can I help you with trading today?");
            renderButtons(['Profit Tips', 'Withdraw', 'Market Status']);
        }

        function renderButtons(buttons) {
            const container = document.getElementById('quick-options');
            container.innerHTML = "";
            buttons.forEach(text => {
                const b = document.createElement('button');
                b.className = 'opt-btn';
                b.innerText = text;
                b.onclick = () => { document.getElementById('message').value = text; handleSend(); };
                container.appendChild(b);
            });
        }

        function speak(text) {
            window.speechSynthesis.cancel();
            const msg = new SpeechSynthesisUtterance(text);
            msg.lang = 'bn-BD';
            
            // যদি ফিমেল ভয়েস পাওয়া যায়, সেটি সেট করা
            if (femaleVoice) {
                msg.voice = femaleVoice;
            }
            
            // কণ্ঠস্বর একটু নরম এবং নারীসুলভ করার জন্য পিচ বাড়ানো
            msg.pitch = 1.2; 
            msg.rate = 1.0;

            const orb = document.getElementById('ai-orb');
            msg.onstart = () => orb.classList.add('active-orb');
            msg.onend = () => orb.classList.remove('active-orb');
            window.speechSynthesis.speak(msg);
        }

        async function handleSend() {
            const input = document.getElementById('message');
            const msg = input.value;
            if(!msg) return;
            input.value = "";
            const orb = document.getElementById('ai-orb');
            orb.classList.add('active-orb');

            try {
                const res = await fetch(`https://general-runtime.voiceflow.com/state/user/user_123/interact`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json', 'versionID': 'production' },
                    body: JSON.stringify({ action: { type: 'text', payload: msg }, projectID: projectID })
                });
                const data = await res.json();
                orb.classList.remove('active-orb');
                data.forEach(trace => {
                    if (trace.type === 'text' || trace.type === 'speak') speak(trace.payload.message || trace.payload.text);
                    if (trace.type === 'choice') renderButtons(trace.payload.buttons.map(b => b.name));
                });
            } catch (err) {
                orb.classList.remove('active-orb');
                speak("Connection error.");
            }
        }
        document.getElementById('message').addEventListener('keypress', (e) => { if(e.key === 'Enter') handleSend(); });
    </script>
</body>
</html>
