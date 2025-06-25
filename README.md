<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>ماشین حساب پیشرفته + نمودار + صوتی</title>
<style>
  body {
    font-family: Tahoma, sans-serif;
    direction: rtl;
    background: #f5f5f5;
    color: #222;
    margin: 0; padding: 0;
  }
  .container {
    max-width: 800px;
    margin: 20px auto;
    background: white;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 0 12px #aaa;
  }
  input, select, button, textarea {
    font-size: 1rem;
    padding: 8px;
    margin: 5px 0;
    width: 100%;
    box-sizing: border-box;
    border-radius: 5px;
    border: 1px solid #ccc;
  }
  button {
    background: #28a745;
    color: white;
    border: none;
    cursor: pointer;
  }
  button:hover {
    background: #218838;
  }
  #calcHistory, #formulaList, #searchResults {
    max-height: 150px;
    overflow-y: auto;
    background: #eee;
    padding: 10px;
    border-radius: 5px;
    font-size: 0.9rem;
  }
  #graphDiv {
    margin-top: 20px;
  }
  label {
    margin-top: 10px;
    display: block;
    font-weight: bold;
  }
  .flex-row {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
  }
  .flex-row > * {
    flex: 1 1 100px;
  }
</style>
<!-- بارگذاری Plotly.js برای نمودار -->
<script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
</head>
<body>

<div class="container">
  <h2>ماشین حساب پیشرفته</h2>
  <input id="calcInput" placeholder="فرمول ریاضی (مثلاً: sin(30) + log(10))" />
  
  <label>تبدیل واحد</label>
  <div class="flex-row">
    <select id="unitType">
      <option value="">انتخاب نوع واحد</option>
      <option value="length">طول</option>
      <option value="weight">وزن</option>
      <option value="temperature">دما</option>
      <option value="volume">حجم</option>
    </select>
    <input id="unitFrom" placeholder="واحد مبدا (مثلاً: m, kg, C)" />
    <input id="unitTo" placeholder="واحد مقصد (مثلاً: cm, g, F)" />
    <input id="unitValue" type="number" placeholder="مقدار" />
    <button onclick="convertUnits()">تبدیل</button>
  </div>
  <div id="unitResult" style="margin:10px 0; font-weight:bold;"></div>

  <label>نتیجه محاسبه</label>
  <button onclick="calculate()">محاسبه فرمول</button>
  <div id="calcResult" style="margin:10px 0; font-weight:bold;"></div>

  <label>تاریخچه محاسبات (جستجو و برچسب‌گذاری)</label>
  <input id="searchCalcHistory" placeholder="جستجو در تاریخچه..." oninput="searchHistory()" />
  <div id="searchResults"></div>
  <button onclick="clearCalcHistory()">پاک کردن تاریخچه</button>
  <div id="calcHistory"></div>

  <label>ذخیره فرمول‌ها برای استفاده مجدد</label>
  <input id="saveFormulaName" placeholder="نام فرمول" />
  <button onclick="saveFormula()">ذخیره فرمول</button>
  <div id="formulaList"></div>

  <label>اسکریپت‌های کوچک (ماکرو)</label>
  <textarea id="macroScript" placeholder="کد جاوااسکریپت اجرا شود..."></textarea>
  <button onclick="runMacro()">اجرای اسکریپت</button>

  <hr />
  <h2>رسم نمودار و تحلیل داده</h2>
  <textarea id="plotData" placeholder="داده‌ها (مثلاً: 1,2,3,4,5)"></textarea>
  <button onclick="plotGraph()">رسم نمودار دو بعدی</button>
  <button onclick="plot3DGraph()">رسم نمودار سه‌بعدی</button>
  <button onclick="exportGraph()">ذخیره نمودار به تصویر</button>
  <div id="graphDiv" style="width:100%;height:400px;"></div>

  <h3>آنالیز آماری ساده</h3>
  <button onclick="showStats()">نمایش آمار</button>
  <div id="statsResult"></div>

  <hr />
  <h2>امکانات چندرسانه‌ای و صوتی</h2>
  <textarea id="chatInput" placeholder="سوالی بپرسید یا متن را بگویید..."></textarea>
  <div class="flex-row">
    <button onclick="chatRespond()">ارسال</button>
    <button onclick="startVoiceRecognition()">🎤 گفتار به متن</button>
    <button onclick="speakText()">🔊 خواندن جواب</button>
  </div>
  <div id="chatHistory"></div>
</div>

<script>
  // --- تبدیل واحد ---
  const unitConversionTable = {
    length: {
      m: 1,
      cm: 0.01,
      mm: 0.001,
      km: 1000,
      inch: 0.0254,
      ft: 0.3048,
      yd: 0.9144,
      mile: 1609.34
    },
    weight: {
      kg: 1,
      g: 0.001,
      mg: 0.000001,
      lb: 0.453592,
      oz: 0.0283495
    },
    temperature: {
      C: 'C',
      F: 'F',
      K: 'K'
    },
    volume: {
      l: 1,
      ml: 0.001,
      m3: 1000,
      gallon: 3.78541,
      pint: 0.473176
    }
  };

  function convertUnits() {
    const type = document.getElementById('unitType').value;
    const from = document.getElementById('unitFrom').value.trim();
    const to = document.getElementById('unitTo').value.trim();
    const value = parseFloat(document.getElementById('unitValue').value);
    const resultDiv = document.getElementById('unitResult');
    if (!type || !from || !to || isNaN(value)) {
      resultDiv.textContent = 'لطفاً همه فیلدها را به درستی پر کنید.';
      return;
    }
    if (!(type in unitConversionTable)) {
      resultDiv.textContent = 'نوع واحد پشتیبانی نمی‌شود.';
      return;
    }
    if (type === 'temperature') {
      // دما با فرمول تبدیل
      let valC;
      switch(from.toUpperCase()){
        case 'C': valC = value; break;
        case 'F': valC = (value - 32) * 5/9; break;
        case 'K': valC = value - 273.15; break;
        default:
          resultDiv.textContent = 'واحد مبدا دما نامعتبر است.';
          return;
      }
      let finalVal;
      switch(to.toUpperCase()){
        case 'C': finalVal = valC; break;
        case 'F': finalVal = (valC * 9/5) + 32; break;
        case 'K': finalVal = valC + 273.15; break;
        default:
          resultDiv.textContent = 'واحد مقصد دما نامعتبر است.';
          return;
      }
      resultDiv.textContent = `${value} ${from} برابر است با ${finalVal.toFixed(3)} ${to}`;
      return;
    }

    const table = unitConversionTable[type];
    if (!(from in table) || !(to in table)) {
      resultDiv.textContent = 'واحد مبدا یا مقصد نامعتبر است.';
      return;
    }
    const baseVal = value * table[from];
    const converted = baseVal / table[to];
    resultDiv.textContent = `${value} ${from} برابر است با ${converted.toFixed(5)} ${to}`;
  }

  // --- ماشین حساب پیشرفته ---

  let calcHistory = JSON.parse(localStorage.getItem('calcHistory') || '[]');
  let savedFormulas = JSON.parse(localStorage.getItem('savedFormulas') || '[]');

  function updateCalcHistory() {
    const histDiv = document.getElementById('calcHistory');
    histDiv.innerHTML = calcHistory.map(item =>
      `<div><b>${item.expr}</b> = ${item.result} ${item.tags ? `<small> [${item.tags.join(', ')}]</small>` : ''}</div>`
    ).join('');
    updateFormulaList();
  }

  function searchHistory() {
    const term = document.getElementById('searchCalcHistory').value.trim();
    const searchDiv = document.getElementById('searchResults');
    if (!term) {
      searchDiv.innerHTML = '';
      return;
    }
    const filtered = calcHistory.filter(item =>
      item.expr.includes(term) || (item.tags && item.tags.some(tag => tag.includes(term)))
    );
    searchDiv.innerHTML = filtered.map(item =>
      `<div><b>${item.expr}</b> = ${item.result} ${item.tags ? `<small> [${item.tags.join(', ')}]</small>` : ''}</div>`
    ).join('');
  }

  function calculate() {
    let input = document.getElementById('calcInput').value.trim();
    if (!input) return;
    try {
      // توابع پایه ریاضی
      let expr = input
        .replace(/sin/g, 'Math.sin')
        .replace(/cos/g, 'Math.cos')
        .replace(/tan/g, 'Math.tan')
        .replace(/sqrt/g, 'Math.sqrt')
        .replace(/log/g, 'Math.log')
        .replace(/ln/g, 'Math.log')
        .replace(/pi/g, Math.PI)
        .replace(/e/g, Math.E);

      let result = eval(expr);
      document.getElementById('calcResult').textContent = 'نتیجه: ' + result;
      // افزودن به تاریخچه
      calcHistory.push({expr: input, result: result, tags: []});
      localStorage.setItem('calcHistory', JSON.stringify(calcHistory));
      updateCalcHistory();
    } catch(e) {
      document.getElementById('calcResult').textContent = 'فرمول اشتباه است!';
    }
  }

  function clearCalcHistory() {
    calcHistory = [];
    localStorage.removeItem('calcHistory');
    updateCalcHistory();
    document.getElementById('calcResult').textContent = '';
  }

  // ذخیره فرمول‌ها
  function saveFormula() {
    let name = document.getElementById('saveFormulaName').value.trim();
    let expr = document.getElementById('calcInput').value.trim
