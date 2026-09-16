<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>סורק כשרות - אוקראינה</title>
    <style>
        body { font-family: system-ui, sans-serif; text-align: center; background: #f4f4f9; padding: 20px; margin: 0; }
        .container { max-width: 500px; margin: 0 auto; background: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        video { width: 100%; border-radius: 8px; background: #222; min-height: 200px; }
        button { background: #0070f3; color: white; border: none; padding: 14px 20px; font-size: 16px; border-radius: 6px; cursor: pointer; width: 100%; margin-top: 10px; font-weight: bold; }
        #result { margin-top: 15px; font-weight: bold; padding: 12px; border-radius: 6px; background: #eee; }
        .kosher { background: #d4edda !important; color: #155724; }
        .unknown { background: #fff3cd !important; color: #856404; }
    </style>
</head>
<body>
    <div class="container">
        <h2>סורק מוצרים כשרים 🇺🇦</h2>
        <video id="video" autoplay playsinline></video>
        <button id="camBtn" onclick="startCamera()">הפעל מצלמה</button>
        <button id="scanBtn" onclick="scanProduct()" style="display:none; background:#28a745;">סרוק מוצר</button>
        <div id="result">לחץ על "הפעל מצלמה" כדי להתחיל</div>
    </div>

    <script src="https://cdn.jsdelivr.net/npm/tesseract.js@5/dist/tesseract.min.js"></script>
    <script>
        const video = document.getElementById('video');
        const resultDiv = document.getElementById('result');
        const camBtn = document.getElementById('camBtn');
        const scanBtn = document.getElementById('scanBtn');

        const kosherDatabase = [
            { name: "Chumak", ukr: "Чумак", status: "כשר", details: "קטשופ ורכזי עגבניות (ללא תוספות חלביות)" },
            { name: "Sadochek", ukr: "Садочок", status: "כשר", details: "מיצים טבעיים (תפוח, תפוז)" },
            { name: "Morshynska", ukr: "Моршинська", status: "כשר", details: "מים מינרליים" },
            { name: "Roshen", ukr: "Рошен", status: "טעון בדיקה", details: "יש לבדוק רכיבים ספציפיים / חלב נכרי" }
        ];

        async function startCamera() {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: { facingMode: 'environment' } });
                video.srcObject = stream;
                camBtn.style.display = 'none';
                scanBtn.style.display = 'block';
                resultDiv.innerText = "המצלמה פעילה. כוון למוצר ולחץ סרוק.";
            } catch (err) {
                resultDiv.innerText = "שגיאה בגישה למצלמה: " + err.message;
            }
        }

        async function scanProduct() {
            resultDiv.className = "";
            resultDiv.innerText = "מעבד תמונה (OCR)...";
            
            const canvas = document.createElement('canvas');
            canvas.width = video.videoWidth || 640;
            canvas.height = video.videoHeight || 480;
            canvas.getContext('2d').drawImage(video, 0, 0);

            try {
                const worker = await Tesseract.createWorker(['ukr', 'eng']);
                const ret = await worker.recognize(canvas);
                await worker.terminate();
                
                const text = ret.data.text.toLowerCase();
                let found = kosherDatabase.find(item => 
                    text.includes(item.name.toLowerCase()) || text.includes(item.ukr.toLowerCase())
                );

                if (found) {
                    resultDiv.className = "kosher";
                    resultDiv.innerHTML = `<strong>${found.name} (${found.ukr})</strong><br>סטטוס: ${found.status}<br>${found.details}`;
                } else {
                    resultDiv.className = "unknown";
                    resultDiv.innerText = "המוצר לא זוהה במאגר.";
                }
            } catch (e) {
                resultDiv.innerText = "שגיאה בסריקה, נסה שוב";
            }
        }
    </script>
</body>
</html>
