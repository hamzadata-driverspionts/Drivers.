<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
<meta charset="UTF-8">
<title>الاستعلام عن النقاط</title>

<style>
body {
    font-family: Arial;
    text-align: center;
    background: #f2f2f2;
}

.box {
    background: white;
    padding: 20px;
    margin: 50px auto;
    width: 340px;
    border-radius: 10px;
    box-shadow: 0 0 10px #ccc;
}

.logo {
    width: 120px;
    margin-bottom: 10px;
}

input {
    width: 90%;
    padding: 10px;
    margin: 10px;
    font-size: 16px;
}

button {
    padding: 10px 20px;
    background: #2e7d32;
    color: white;
    border: none;
    cursor: pointer;
    font-size: 16px;
}

button:hover {
    background: #1b5e20;
}

.result {
    margin-top: 20px;
    font-size: 15px;
    line-height: 1.7;
    text-align: right;
}

ul {
    list-style: none;
    padding: 0;
}

li {
    padding: 6px;
    border-bottom: 1px solid #eee;
}

.deduct {
    color: red;
    font-weight: bold;
}

.add {
    color: green;
    font-weight: bold;
}

.points-final {
    margin-top: 15px;
    font-size: 18px;
    font-weight: bold;
}

.warning {
    color: red;
    font-weight: bold;
    margin-top: 10px;
}
</style>
</head>

<body>

<div class="box">
    <img src="logo.png" class="logo">
    <h3>ادخل رقمك التعريفي</h3>
    <input type="text" id="searchId" placeholder="الرقم التعريفي">
    <button onclick="search()">بحث</button>
    <div class="result" id="result"></div>
</div>

<script>
const sheetURL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vTq27e_VfyW4FIqgMTj3zv3kcHqBrUfQWJU1wf4rskMFD1nlT50PGDLzu7gbvxseHJFq69o64d3ENJa/pub?output=csv";

async function search() {

    let id = document.getElementById("searchId").value.trim();
    let resultBox = document.getElementById("result");

    if(!id){
        resultBox.innerHTML = "الرجاء إدخال الرقم التعريفي";
        return;
    }

    resultBox.innerHTML = "جاري البحث...";

    try {

        let res = await fetch(sheetURL);
        let text = await res.text();

        let rows = text.split("\n").slice(1);
        let records = [];

        for(let row of rows){
            if(!row.trim()) continue;

            let cols = row.split(",");

            if(cols[0].trim() === id){
                records.push({
                    name: cols[1].trim(),
                    date: cols[2].trim(),
                    type: cols[3].trim(),
                    reason: cols[4].trim(),
                    points: parseFloat(cols[5])
                });
            }
        }

        if(records.length === 0){
            resultBox.innerHTML = "لا توجد بيانات لهذا الرقم";
            return;
        }

        // ===== الرصيد الأساسي =====
        let totalPoints = 12;

        // ===== عرض البيانات =====
        let html = `<b>الاسم:</b> ${records[0].name}<br><br>`;
        html += `<ul>`;

        records.forEach(rec => {

            // توحيد كلمة اضافة (مع أو بدون همزة)
            let type = rec.type.replace("إ","ا").trim();

            if(type === "خصم"){
                totalPoints -= rec.points;
                html += `<li class="deduct">${rec.date} | خصم | ${rec.reason} | ${rec.points}</li>`;
            }
            else if(type === "اضافة"){
                totalPoints += rec.points;
                html += `<li class="add">${rec.date} | إضافة | ${rec.reason} | ${rec.points}</li>`;
            }
        });

        html += `</ul>`;

        // ===== حدود النقاط =====
        if(totalPoints > 12) totalPoints = 12;
        if(totalPoints < 0) totalPoints = 0;

        html += `<div class="points-final">النقاط الحالية: ${totalPoints} / 12</div>`;

        // ===== تنبيه إذا النقاط منخفضة =====
        if(totalPoints <= 6){
            html += `<div class="warning">تنبيه: النقاط منخفضة</div>`;
        }

        resultBox.innerHTML = html;

    }
    catch(error){
        resultBox.innerHTML = "حدث خطأ في جلب البيانات";
        console.error(error);
    }
}
</script>

</body>
</html>
