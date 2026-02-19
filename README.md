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
    width: 320px;
    border-radius: 10px;
    box-shadow: 0 0 10px #ccc;
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
    font-size: 18px;
    line-height: 1.8;
}
</style>
</head>

<body>

<div class="box">
    <img src="logo.png" class="logo">
<h2>ادخل رقمك التعريفي</h2>
<input type="text" id="searchId" placeholder="الرقم التعريفي">
<button onclick="search()">بحث</button>
<div class="result" id="result"></div>
</div>

<script>
const sheetURL = "https://docs.google.com/spreadsheets/d/e/2PACX-1vTq27e_VfyW4FIqgMTj3zv3kcHqBrUfQWJU1wf4rskMFD1nlT50PGDLzu7gbvxseHJFq69o64d3ENJa/pub?output=csv";

async function search(){
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
        let found = false;

        for(let row of rows){
            if(!row.trim()) continue;

            let cols = row.split(",");

            if(cols[0].trim() === id){
                found = true;
                resultBox.innerHTML = `
                    <b>الاسم:</b> ${cols[1]}<br>
                    <b>النقاط المتبقية:</b> ${cols[2]}<br>
                    <b>النقاط المخصومة:</b> ${cols[3]}
                `;
                break;
            }
        }

        if(!found){
            resultBox.innerHTML = "لا توجد بيانات لهذا الرقم";
        }

    } catch (error) {
        resultBox.innerHTML = "حدث خطأ في جلب البيانات";
        console.error(error);
    }
}
</script>

</body>
</html>
