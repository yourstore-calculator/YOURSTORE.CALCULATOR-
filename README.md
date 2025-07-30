# YOURSTORE.CALCULATOR
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار المطورة</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: rtl;
      padding: 30px;
    }
    .logo {
      text-align: center;
      margin-bottom: 20px;
    }
    .logo img {
      max-height: 100px;
    }
    select, input {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
      box-sizing: border-box;
      background-color: #333;
      color: white;
    }
    /* تم إزالة نمط :disabled لأنه لم يعد ضرورياً بنفس القدر */
    input:disabled {
      background-color: #555;
      cursor: not-allowed;
      color: #aaa;
    }
    .section {
      background-color: #252526;
      padding: 15px;
      border-radius: 8px;
      margin-bottom: 20px;
      border: 1px solid #333;
    }
    .addon-section {
        background-color: #2a2d2e;
        padding: 10px;
        border-radius: 6px;
        margin-top: 10px;
    }
    label {
        font-weight: bold;
        color: #00e676;
    }
    button {
      background-color: #007acc;
      color: white;
      padding: 12px 22px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 16px;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #005a9e;
    }
    .btn-secondary { background-color: #555; }
    .btn-secondary:hover { background-color: #777; }
    .btn-danger { background-color: #c0392b; }
    .btn-danger:hover { background-color: #a52f22; }

    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      border-bottom: 1px solid #555;
      padding: 10px 0;
      line-height: 1.8;
    }
    .result-item:last-child {
      border-bottom: none;
    }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 15px;
      border-top: 2px solid #00e676;
      margin-top: 15px;
      font-size: 1.1em;
    }
  </style>
</head>
<body>

<div class="logo">
  <img src="https://i.imgur.com/ih5zjVj.png" alt="شعار الشركة">
</div>

<div class="section">
  <label for="mainCategory">الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()"></select>
</div>

<div class="section">
  <label for="subType">النوع الفرعي:</label>
  <select id="subType" onchange="updateUIBasedOnType()"></select>
</div>

<div class="section">
  <label for="height">الطول (متر):</label>
  <input type="number" id="height" step="0.01" value="1" />
  
  <label for="width">العرض (متر):</label>
  <input type="number" id="width" step="0.01" value="1" />

  <label for="quantity">الكمية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<!-- قسم الإضافات الديناميكي -->
<div class="section" id="addonsHost"></div>

<button onclick="calculate()">➕ أضف للنتائج</button>
<button onclick="clearResults()" class="btn-danger">🗑️ مسح النتائج</button>
<button onclick="saveAsWord()" class="btn-secondary">💾 حفظ كـ Word</button>

<div class="results" id="results"></div>

<script>
// ثابت سعر الشحن لكل CBM
const SHIPPING_RATE = 48;

// هيكل البيانات الشامل
const productData = {
    "نوافذ": {
        "دبل جلاس دبل فريم ثابتة": { price: 34, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس دبل فريم حركة": { price: 73, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس دبل فريم حركتين": { price: 92, cbm: 0.13, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم ثابتة": { price: 26, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم حركة": { price: 46, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "دبل جلاس سنجل فريم حركتين": { price: 58, cbm: 0.07, method: 'per_meter', addons: ['curtain', 'net'] },
        "سنجل جلاس سنجل فريم ثابتة": { price: 20, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "سنجل جلاس سنجل فريم حركة": { price: 43, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "سنجل جلاس سنجل فريم حركتين": { price: 47, cbm: 0.07, method: 'per_meter', addons: ['net'] },
        "نوافذ السلايدنج": { price: 10, cbm: 0.13, method: 'per_meter', special: 'add_10', addons: ['curtain'] },
        "النوافذ الكهربائية": { price: 102, cbm: 0.13, method: 'per_meter' },
        "سكاي لايت بدون مكينة": { price: 56, cbm: 0.13, method: 'per_meter' },
        "سكاي لايت مع مكينة": { price: 145, cbm: 0.13, method: 'per_meter' },
        "كارتن وول ثقيل": { price: 56, cbm: 0.15, method: 'per_meter' },
        "كارتن وول خفيف": { price: 45, cbm: 0.15, method: 'per_meter' },
    },
    "أبواب": {
        "باب المدخل - زينك": { price: 66, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "باب المدخل - ستينلس ستيل": { price: 120, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "باب المدخل - كاست المنيوم": { price: 168, cbm: 0.20, method: 'per_meter', special: 'add_10' },
        "WPC - فارغ": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع خشب": { price: 50, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع حشوة ضد الصوت": { price: 60, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "WPC - مع فريم ألمينيوم": { price: 67, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0, addons: ['telbisa'] },
        "ألمنيوم - فارغ": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - مع خشب": { price: 75, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - فل": { price: 85, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - باب مخفي": { price: 110, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "ألمنيوم - باب خارجي": { price: 61, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 1.0 },
        "دورات مياه - النوع الجديد": { price: 55, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "دورات مياه - النوع القديم": { price: 45, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
        "دورات مياه - مخفي زجاجي": { price: 65, cbm: 0.11, method: 'per_unit', std_h: 2.2, std_w: 0.8 },
    },
    "أبواب سحب": {
        "داخلي - زجاج": { price: 38, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "داخلي - متين": { price: 41, cbm: 0.15, method: 'per_meter' },
        "خارجي - جزء مفتوح": { price: 55, cbm: 0.15, method: 'per_meter' },
        "خارجي - جزئين مفتوحات": { price: 58, cbm: 0.15, method: 'per_meter' },
        "WPC - سلايد": { price: 61, cbm: 0.15, method: 'per_meter' },
    },
    "أبواب فولدنج": {
        "داخلي": { price: 39, cbm: 0.15, method: 'per_meter', addons: ['curtain'] },
        "خارجي": { price: 56, cbm: 0.15, method: 'per_meter' },
    },
    "شتر خارجي": {
        "شتر رول": { price: 28, cbm: 0.20, method: 'per_meter' },
    },
    "أبواب حدائق": {
        "كاست المنيوم": { price: 91, cbm: 0.20, method: 'per_meter' },
    },
    "حواجز": {
        "حاجز درج": { price: 43, cbm: 0.05, method: 'per_meter' },
        "حاجز حمام": { price: 32, cbm: 0.05, method: 'per_meter' },
    }
};

const addonPrices = {
    curtain: 26, // للمتر
    net: {
        'باب': 39, 'فولدنج': 18, 'سلايد': 14
    },
    telbisa: 10 // للمتر
};

let resultsList = [];

// --- تهيئة الواجهة ---
function initializeApp() {
    const mainCat = document.getElementById("mainCategory");
    mainCat.innerHTML = `<option value="">-- اختر الفئة --</option>`;
    Object.keys(productData).forEach(cat => {
        mainCat.innerHTML += `<option value="${cat}">${cat}</option>`;
    });
    loadSubTypes();
}

function loadSubTypes() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subType = document.getElementById("subType");
    subType.innerHTML = "";
    if (mainCatVal && productData[mainCatVal]) {
        Object.keys(productData[mainCatVal]).forEach(sub => {
            subType.innerHTML += `<option value="${sub}">${sub}</option>`;
        });
    }
    updateUIBasedOnType();
}

function updateUIBasedOnType() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    
    if (!mainCatVal || !subVal) {
        document.getElementById("addonsHost").innerHTML = "";
        return;
    }

    const data = productData[mainCatVal][subVal];
    const heightInput = document.getElementById("height");
    const widthInput = document.getElementById("width");
    const addonsHost = document.getElementById("addonsHost");
    addonsHost.innerHTML = ""; // Clear previous addons

    // *** التعديل هنا ***
    // التحكم بالأبعاد: يتم ملء الحقول دائماً ولكن لا يتم تعطيلها
    heightInput.disabled = false;
    widthInput.disabled = false;

    if (data.method === 'per_unit') {
        heightInput.value = data.std_h;
        widthInput.value = data.std_w;
    }
    
    // التحكم بالإضافات
    if (data.addons) {
        if (data.addons.includes('curtain')) {
            addonsHost.innerHTML += `
                <div class="addon-section">
                    <label><input type="checkbox" id="addCurtain" /> إضافة ستارة داخلية (26 ريال للمتر)</label>
                </div>`;
        }
        if (data.addons.includes('net')) {
            addonsHost.innerHTML += `
                <div class="addon-section">
                    <label for="netType">إضافة شبك (بدون شحن إضافي):</label>
                    <select id="netType">
                        <option value="">-- بدون --</option>
                        <option value="باب">باب (${addonPrices.net['باب']} ريال للمتر)</option>
                        <option value="فولدنج">فولدنج (${addonPrices.net['فولدنج']} ريال للمتر)</option>
                        <option value="سلايد">سلايد (${addonPrices.net['سلايد']} ريال للمتر)</option>
                    </select>
                </div>`;
        }
        if (data.addons.includes('telbisa')) {
             addonsHost.innerHTML += `
                <div class="addon-section">
                    <label><input type="checkbox" id="addTelbisa" /> إضافة تلبيسة للسقف</label>
                    <input type="number" id="telbisaLength" placeholder="أدخل طول التلبيسة بالمتر" style="display:none;" value="1" step="0.1" />
                </div>`;
             document.getElementById('addTelbisa').onchange = (e) => {
                 document.getElementById('telbisaLength').style.display = e.target.checked ? 'block' : 'none';
             };
        }
    }
}


function calculate() {
    const mainCatVal = document.getElementById("mainCategory").value;
    const subVal = document.getElementById("subType").value;
    if (!mainCatVal || !subVal) { alert("يرجى اختيار الفئة والنوع."); return; }
    
    const data = productData[mainCatVal][subVal];
    const qty = parseInt(document.getElementById("quantity").value) || 1;
    const h = parseFloat(document.getElementById("height").value) || 0;
    const w = parseFloat(document.getElementById("width").value) || 0;
    if (h === 0 || w === 0) { alert("الأبعاد يجب أن تكون أكبر من صفر."); return; }
    
    const area = h * w;
    let basePrice = 0;
    let sizePenalty = 0;

    if (data.method === 'per_meter') {
        basePrice = data.price * area;
    } else if (data.method === 'per_unit') {
        basePrice = data.price;
        const stdArea = data.std_h * data.std_w;
        if (area > stdArea) {
            const extraArea = area - stdArea;
            // تقريب الناتج لأعلى لأقرب 0.1 ثم حساب التكلفة
            sizePenalty = Math.ceil(extraArea / 0.1) * 2;
        }
    }

    let itemPrice = basePrice + sizePenalty;
    let addonsDetails = "";
    
    // Addons calculation
    const addCurtain = document.getElementById("addCurtain")?.checked;
    if (addCurtain) {
        const curtainCost = addonPrices.curtain * area;
        itemPrice += curtainCost;
        addonsDetails += `<li>ستارة: ${curtainCost.toFixed(2)}</li>`;
    }

    const netType = document.getElementById("netType")?.value;
    if (netType) {
        const netArea = area * 0.5;
        const netCost = netArea * addonPrices.net[netType];
        itemPrice += netCost;
        addonsDetails += `<li>شبك ${netType}: ${netCost.toFixed(2)}</li>`;
    }

    const addTelbisa = document.getElementById("addTelbisa")?.checked;
    let telbisaCost = 0;
    let telbisaLength = 0;
    if (addTelbisa) {
        telbisaLength = parseFloat(document.getElementById("telbisaLength").value) || 0;
        if (telbisaLength > 0) {
            telbisaCost = Math.ceil(telbisaLength) * addonPrices.telbisa;
            itemPrice += telbisaCost;
            addonsDetails += `<li>تلبيسة سقف (${telbisaLength}م): ${telbisaCost.toFixed(2)}</li>`;
        }
    }

    // Special additions
    if (data.special === 'add_10') {
        itemPrice += 10;
        addonsDetails += `<li>إضافة خاصة: 10.00</li>`;
    }

    // Shipping calculation
    let shippingCBM = 0;
    if (addTelbisa && telbisaLength > 0) {
        const totalHeightForShipping = h + telbisaLength;
        shippingCBM = (totalHeightForShipping * w) * data.cbm;
    } else {
        shippingCBM = area * data.cbm;
    }
    const shippingCost = shippingCBM * SHIPPING_RATE;

    const totalItemCost = itemPrice + shippingCost;
    const finalPriceForQty = totalItemCost * qty;

    resultsList.push({
        sub: subVal, h, w, qty,
        basePrice: basePrice * qty,
        sizePenalty: sizePenalty * qty,
        addonsDetails,
        shipping: shippingCost * qty,
        final: finalPriceForQty
    });

    renderResults();
}


function renderResults() {
    const container = document.getElementById("results");
    container.innerHTML = "";
    let sum = 0;

    resultsList.forEach(r => {
        sum += r.final;
        container.innerHTML += `
            <div class="result-item">
                <b>النوع:</b> ${r.sub} (الكمية: ${r.qty})<br>
                <b>المقاس:</b> ${r.h} × ${r.w} متر<br>
                <ul>
                    <li>السعر الأساسي: ${r.basePrice.toFixed(2)}</li>
                    ${r.sizePenalty > 0 ? `<li>تكلفة زيادة الحجم: ${r.sizePenalty.toFixed(2)}</li>` : ''}
                    ${r.addonsDetails}
                    <li>🚚 الشحن: ${r.shipping.toFixed(2)}</li>
                </ul>
                <b>💵 السعر الإجمالي للبند: ${r.final.toFixed(2)} ريال</b>
            </div>`;
    });

    if (resultsList.length > 0) {
        const comm = sum * 0.04;
        const total = sum + comm;
        container.innerHTML += `
            <div class="summary">
                المجموع الفرعي: ${sum.toFixed(2)}<br>
                عمولة المكتب (4%): ${comm.toFixed(2)}<br>
                <b>💰 الناتج النهائي: ${total.toFixed(2)} ريال</b><br><br>
                <span style="color:#ccc; font-size: 0.9em;">* الأسعار شاملة الشحن<br>** الأسعار لا تشمل التركيب</span>
            </div>`;
    }
}

function clearResults() {
    resultsList = [];
    document.getElementById("results").innerHTML = "";
    document.getElementById("mainCategory").value = "";
    document.getElementById("height").value = "1";
    document.getElementById("width").value = "1";
    document.getElementById("quantity").value = "1";
    loadSubTypes(); 
}

function saveAsWord(){
    if (resultsList.length === 0) { alert("لا توجد نتائج لحفظها."); return; }
    const header = `<html xmlns:o='urn:schemas-microsoft-com:office:office' xmlns:w='urn:schemas-microsoft-com:office:word' xmlns='http://www.w3.org/TR/REC-html40'><head><meta charset='utf-8'><title>عرض سعر</title></head><body dir="rtl">`;
    const footer = "</body></html>";
    let content = document.getElementById("results").innerHTML;
    
    const sourceHTML = header + content + footer;
    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = 'عرض_سعر.doc';
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

// بدء تشغيل التطبيق
initializeApp();

</script>

</body>
</html>
