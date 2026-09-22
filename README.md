<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CleanSensor Pro - ระบบแจ้งเตือนห้องน้ำโรงเรียน</title>
    <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;600&display=swap" rel="stylesheet">
    
    <style>
        * { box-sizing: border-box; font-family: 'Prompt', sans-serif; }
        body {
            background-color: #eef2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
        }
        .card {
            background: white;
            padding: 30px;
            border-radius: 16px;
            box-shadow: 0 8px 20px rgba(0,0,0,0.06);
            width: 100%;
            max-width: 480px;
        }
        .header { text-align: center; margin-bottom: 20px; }
        .header h2 { color: #0088cc; margin: 0; font-size: 26px; }
        .header p { color: #666; font-size: 13px; margin-top: 4px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 6px; font-weight: 600; color: #333; font-size: 14px; }
        select, input {
            width: 100%;
            padding: 10px 12px;
            border: 1px solid #ccc;
            border-radius: 8px;
            font-size: 14px;
            outline: none;
        }
        select:focus, input:focus { border-color: #0088cc; }
        
        /* แท็ก Quick Tags สำหรับเลือกปัญหาด่วน */
        .tag-container { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 6px; }
        .tag {
            background: #f0f0f0;
            padding: 6px 12px;
            border-radius: 20px;
            font-size: 12px;
            cursor: pointer;
            user-select: none;
            transition: 0.2s;
        }
        .tag.active { background: #0088cc; color: white; }

        .btn-submit {
            width: 100%;
            background-color: #0088cc;
            color: white;
            border: none;
            padding: 14px;
            border-radius: 8px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            margin-top: 15px;
            transition: 0.3s;
        }
        .btn-submit:hover { background-color: #006699; }
        #statusMessage { margin-top: 15px; text-align: center; font-size: 14px; font-weight: 600; }
        .success { color: #2e7d32; }
        .error { color: #d32f2f; }
    </style>
</head>
<body>

<div class="card">
    <div class="header">
        <h2>CleanSensor Pro 🧼</h2>
        <p>ระบบแจ้งเตือนการทำความสะอาดและดูแลห้องน้ำ</p>
    </div>

    <form id="cleanForm">
        <!-- 1. ระบุผู้รายงาน -->
        <div class="form-group">
            <label for="reporter">ชื่อผู้บันทึก / ตำแหน่ง:</label>
            <input type="text" id="reporter" placeholder="เช่น แม่บ้านสมศรี / ครูสมชาย" required>
        </div>

        <!-- 2. เลือกสถานที่ / ห้องน้ำ -->
        <div class="form-group">
            <label for="location">เลือกตำแหน่งห้องน้ำ:</label>
            <select id="location" required>
                <option value="อาคารเรียน 1 - ชั้น 1 (ชาย)">อาคารเรียน 1 - ชั้น 1 (ชาย)</option>
                <option value="อาคารเรียน 1 - ชั้น 1 (หญิง)">อาคารเรียน 1 - ชั้น 1 (หญิง)</option>
                <option value="อาคารเรียน 1 - ชั้น 2 (ชาย)">อาคารเรียน 1 - ชั้น 2 (ชาย)</option>
                <option value="อาคารเรียน 1 - ชั้น 2 (หญิง)">อาคารเรียน 1 - ชั้น 2 (หญิง)</option>
                <option value="อาคารเอนกประสงค์ (รวม)">อาคารเอนกประสงค์ (รวม)</option>
                <option value="โรงอาหาร (ชาย/หญิง)">โรงอาหาร (ชาย/หญิง)</option>
            </select>
        </div>

        <!-- 3. เลือกสถานะ -->
        <div class="form-group">
            <label for="status">สถานะห้องน้ำ:</label>
            <select id="status" required>
                <option value="🟢|ทำความสะอาดแล้ว พร้อมใช้งาน">🟢 ทำความสะอาดเรียบร้อย (พร้อมใช้)</option>
                <option value="🟡|กำลังเข้าทำความสะอาด">🟡 กำลังเข้าทำความสะอาด (โปรดรอ)</option>
                <option value="🔵|รอการทำความสะอาดตามรอบ">🔵 ถึงรอบทำความสะอาด</option>
                <option value="🔴|พบปัญหา / แจ้งซ่อมด่วน">🔴 พบปัญหา / แจ้งชำรุดด่วน</option>
            </select>
        </div>

        <!-- 4. Quick Issues/Remarks (กดเลือกเพื่อเพิ่มหมายเหตุด่วน) -->
        <div class="form-group">
            <label>ระบุปัญหา/หมายเหตุเพิ่มเติม (ถ้ามี):</label>
            <div class="tag-container">
                <span class="tag" onclick="toggleTag(this)">🧻 กระดาษชำระหมด</span>
                <span class="tag" onclick="toggleTag(this)">💧 น้ำไม่ไหล/ไหลค่อย</span>
                <span class="tag" onclick="toggleTag(this)">🧼 สบู่หมด</span>
                <span class="tag" onclick="toggleTag(this)">🧹 พื้นเปียก/สกปรก</span>
                <span class="tag" onclick="toggleTag(this)">🚽 ชักโครกอุดตัน</span>
                <span class="tag" onclick="toggleTag(this)">💡 ไฟเสีย/มืด</span>
            </div>
            <input type="text" id="customRemark" placeholder="หรือพิมพ์รายละเอียดเพิ่มเติมที่นี่..." style="margin-top: 8px;">
        </div>

        <!-- ปุ่มส่งข้อมูล -->
        <button type="button" class="btn-submit" onclick="sendToTelegram()">ส่งแจ้งเตือนเข้า Telegram</button>
    </form>

    <div id="statusMessage"></div>
</div>

<script>
    // ฟังก์ชันเปิด-ปิด Tag หมายเหตุด่วน
    function toggleTag(element) {
        element.classList.toggle('active');
    }

    async function sendToTelegram() {
        const BOT_TOKEN = "8715068421:AAH9dcq6QEaMrIQ2QyAtkpzaiLsuJdV4zTg"; 
        const CHAT_ID = "7043331377";               

        const reporter = document.getElementById("reporter").value.trim() || "ไม่ระบุตัวตน";
        const location = document.getElementById("location").value;
        const statusValue = document.getElementById("status").value;
        const [icon, statusText] = statusValue.split("|");

        // รวบรวมข้อมูล Tags ที่ถูกเลือก
        const selectedTags = Array.from(document.querySelectorAll('.tag.active')).map(t => t.innerText);
        const customRemark = document.getElementById("customRemark").value.trim();
        
        let remarksList = [...selectedTags];
        if (customRemark) remarksList.push(customRemark);
        
        const remarkString = remarksList.length > 0 ? remarksList.join(", ") : "ไม่มีข้อเสนอแนะเพิ่มเติม";

        // เวลาปัจจุบัน
        const now = new Date();
        const timeString = now.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' }) + " น.";

        // จัดรูปแบบโครงสร้างข้อความ
        const messageText = `${icon} *CleanSensor Report*\n` +
                            `📍 *สถานที่:* ${location}\n` +
                            `📌 *สถานะ:* ${statusText}\n` +
                            `📝 *รายละเอียด:* ${remarkString}\n` +
                            `👤 *ผู้บันทึก:* ${reporter}\n` +
                            `⏰ *เวลา:* ${timeString}`;

        const statusDiv = document.getElementById("statusMessage");
        statusDiv.className = "";
        statusDiv.innerText = "กำลังส่งข้อมูล...";

        // สร้าง Inline Keyboard ปุ่มกดรับทราบข้อมูลผ่าน Telegram
        const inlineKeyboard = {
            inline_keyboard: [
                [
                    { text: "✅ รับทราบ / เข้าตรวจสอบ", callback_data: "acknowledged" }
                ]
            ]
        };

        const url = `https://api.telegram.org/bot${BOT_TOKEN}/sendMessage`;
        
        try {
            const response = await fetch(url, {
                method: "POST",
                headers: { "Content-Type": "application/json" },
                body: JSON.stringify({
                    chat_id: CHAT_ID,
                    text: messageText,
                    parse_mode: "Markdown",
                    reply_markup: inlineKeyboard
                })
            });

            const data = await response.json();

            if (data.ok) {
                statusDiv.className = "success";
                statusDiv.innerText = "✅ ส่งรายงานเข้า Telegram สำเร็จ!";
                
                // ล้างค่าฟอร์มบางส่วน
                document.querySelectorAll('.tag.active').forEach(t => t.classList.remove('active'));
                document.getElementById("customRemark").value = "";
            } else {
                statusDiv.className = "error";
                statusDiv.innerText = "❌ เกิดข้อผิดพลาด: " + data.description;
            }
        } catch (error) {
            statusDiv.className = "error";
            statusDiv.innerText = "❌ ไม่สามารถเชื่อมต่อกับ Telegram API ได้";
            console.error(error);
        }
    }
</script>

</body>
</html>
