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

.result {
    margin-top: 20px;
    font-size: 15px;
    text-align: right;
}

.add { color: green; }
.deduct { color: red; }

.points-final {
    font-size: 18px;
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

    if(id === ""){
        resultBox.innerHTML = "الرجاء إدخال الرقم التعريفي";
        return;
    }

    resultBox.innerHTML = "جاري البحث...";

    try {

        let response = await fetch(sheetURL);
        let data = await response.text();

        let rows = data.split("\n").slice(1);

        let totalPoints = 12;
        let html = "";
        let found = false;
        let name = "";

        rows.forEach(row => {

            if(row.trim() === "") return;

            let cols = row.split(",");

            let rowId = (cols[0] || "").trim();

            if(rowId === id){

                found = true;

                name = cols[1] ? cols[1].trim() : "";
                let date = cols[2] ? cols[2].trim() : "";
                let type = cols[3] ? cols[3].trim() : "";
                let reason = cols[4] ? cols[4].trim() : "";
                let points = parseFloat(cols[5]) || 0;

                // توحيد كلمة اضافة
                type = type.replace("إ","ا");

                if(type === "خصم"){
                    totalPoints -= points;
                    html += `<div class="deduct">${date} | خصم | ${reason} | ${points}</div>`;
                }
                else if(type === "اضافة"){
                    totalPoints += points;
                    html += `<div class="add">${date} | إضافة | ${reason} | ${points}</div>`;
                }
            }

        });

        if(!found){
            resultBox.innerHTML = "لا توجد بيانات لهذا الرقم";
            return;
        }

        // ضبط الحدود
        if(totalPoints > 12) totalPoints = 12;
        if(totalPoints < 0) totalPoints = 0;

        resultBox.innerHTML = `
            <b>الاسم:</b> ${name}<br><br>
            ${html}
            <div class="points-final">النقاط الحالية: ${totalPoints} / 12</div>
        `;

    }
    catch(error){
        resultBox.innerHTML = "فشل الاتصال بملف البيانات";
        console.error(error);
    }
}
</script>

</body>
</html>
