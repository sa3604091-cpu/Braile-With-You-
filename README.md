<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>نظام برايل الهجين المطور</title>

    <script src="https://unpkg.com/docx@7.1.0/build/index.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/FileSaver.js/2.0.5/FileSaver.min.js"></script>

    <style>
        :root {
            --bg: #050505;
            --cyan: #00f3ff;
            --purple: #bc13fe;
            --key-bg: rgba(25, 25, 25, 0.9);
            --red: #ff3b3b;
            --glass: rgba(255, 255, 255, 0.05);
        }

        body {
            background-color: var(--bg);
            color: #fff;
            font-family: 'Segoe UI', Tahoma, sans-serif;
            margin: 0;
            overflow: hidden;
            touch-action: none;
            user-select: none;
            background-image: radial-gradient(circle at 50% 50%, #111 0%, #000 100%);
        }

        #display-section {
            height: 22vh;
            background: var(--glass);
            backdrop-filter: blur(15px);
            border-bottom: 2px solid var(--cyan);
            padding: 10px;
            display: flex;
            flex-direction: column;
        }

        #output {
            flex: 1;
            background: transparent;
            border: none;
            color: var(--cyan);
            font-size: 22px;
            resize: none;
            outline: none;
            text-align: right;
            text-shadow: 0 0 8px var(--cyan);
            font-weight: bold;
            user-select: text;
            -webkit-user-select: text;
            touch-action: auto;
        }

        #braille-preview {
            height: 30px;
            color: var(--purple);
            font-size: 18px;
            border-top: 1px solid #333;
            padding-top: 5px;
            overflow-x: auto;
            white-space: nowrap;
        }

        #keyboard-container {
            height: 60vh;
            overflow-y: auto;
            background: transparent;
        }

        #keyboard-sighted {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            gap: 5px;
            padding: 10px;
        }

        #keyboard-blind {
            display: none;
            grid-template-columns: 1fr 1fr;
            gap: 30px;
            padding: 20px;
        }

        .cluster-main {
            background: radial-gradient(circle, #222 0%, #000 100%);
            border: 2px solid var(--purple);
            border-radius: 50%;
            position: relative;
            aspect-ratio: 1/1;
        }

        .sub-key {
            position: absolute;
            width: 60px;
            height: 60px;
            background: #111;
            border: 2px solid var(--cyan);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 18px;
            color: #fff;
            box-shadow: 0 0 10px var(--cyan);
            touch-action: manipulation;
        }

        .sub-key:active {
            background: var(--cyan);
            color: #000;
        }

        .pos-0 { right: -5%; top: 35%; }
        .pos-1 { right: 15%; top: -5%; }
        .pos-2 { left: 15%; top: -5%; }
        .pos-3 { left: -5%; top: 35%; }
        .pos-4 { left: 15%; bottom: -5%; }
        .pos-5 { right: 15%; bottom: -5%; }

        .key-s {
            background: var(--key-bg);
            border: 1px solid rgba(0, 243, 255, 0.3);
            border-radius: 8px;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 55px;
        }

        .key-s b {
            font-size: 18px;
            color: var(--purple);
        }

        #nav-bar {
            height: 18vh;
            display: grid;
            grid-template-columns: repeat(5, 20%);
            background: #000;
            border-top: 2px solid var(--purple);
        }

        .nav-btn {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            font-size: 11px;
            color: #fff;
            cursor: pointer;
            border: 0.1px solid #222;
            text-align: center;
        }

        .nav-btn b {
            color: var(--cyan);
            font-size: 14px;
        }

        .popup-menu {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0);
            width: 90%;
            background: rgba(0,0,0,0.95);
            border: 2px solid var(--cyan);
            border-radius: 15px;
            padding: 15px;
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            z-index: 10001;
            transition: 0.2s;
        }

        .popup-menu.active {
            transform: translate(-50%, -50%) scale(1);
        }

        #mode-toggle {
            position: fixed;
            top: 10px;
            left: 10px;
            z-index: 10002;
            background: var(--purple);
            border: none;
            color: #fff;
            padding: 8px 15px;
            border-radius: 15px;
            font-weight: bold;
            box-shadow: 0 0 10px var(--purple);
        }

        #overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            z-index: 9999;
            display: none;
        }

        /* نافذة اكتشف اللغة */

        #language-popup {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%) scale(0);
            width: 88%;
            max-width: 420px;
            padding: 20px;
            box-sizing: border-box;
            background: #080808;
            border: 2px solid var(--cyan);
            border-radius: 18px;
            z-index: 10003;
            transition: .2s;
            box-shadow: 0 0 20px var(--purple);
        }

        #language-popup.active {
            transform: translate(-50%, -50%) scale(1);
        }

        #language-popup h3 {
            color: var(--cyan);
            text-align: center;
        }

        #language-word {
            width: 100%;
            box-sizing: border-box;
            padding: 13px;
            margin: 12px 0;
            background: #151515;
            color: white;
            border: 1px solid var(--cyan);
            border-radius: 8px;
            font-size: 18px;
            text-align: right;
            outline: none;
            user-select: text;
            -webkit-user-select: text;
            touch-action: auto;
        }

        .language-action {
            width: 100%;
            padding: 12px;
            margin-top: 8px;
            border: 0;
            border-radius: 8px;
            background: var(--purple);
            color: white;
            font-size: 16px;
            font-weight: bold;
        }

        #language-note {
            color: #ccc;
            font-size: 13px;
            line-height: 1.8;
            text-align: center;
        }
    </style>
</head>

<body onclick="closeAllPopups()">

<div id="overlay"></div>

<button id="mode-toggle"
        onclick="event.stopPropagation(); toggleAppMode()">
    🔄 وضع التشغيل
</button>

<div id="display-section" onclick="event.stopPropagation()">
    <textarea id="output"
              placeholder="سوف يظهر النص هنا..."
              aria-label="مساحة النص المكتوب"></textarea>

    <div id="braille-preview"
         dir="ltr"
         aria-label="معاينة برايل"></div>
</div>

<div id="keyboard-container">
    <div id="keyboard-sighted" onclick="event.stopPropagation()"></div>
    <div id="keyboard-blind" onclick="event.stopPropagation()"></div>
</div>

<div id="nav-bar" onclick="event.stopPropagation()">

    <div class="nav-btn"
         onclick="unifiedControl(this, () => openPopup('pop-nums', 'الأرقام'), 'أرقام')">
        🔢<br>أرقام<br><b>⠼</b>
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, () => openPopup('pop-tashkeel', 'التشكيل'), 'تشكيل')">
        ✨<br>تشكيل<br><b>⠠</b>
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, () => openPopup('pop-punct', 'علامات الترقيم'), 'ترقيم')">
        ❓<br>ترقيم<br><b>⠦</b>
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, () => openPopup('pop-hamzas', 'الهمزات'), 'همزات')">
        ء<br>همزات<br><b>⠄</b>
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, handleBackspace, 'حذف')"
         style="color:var(--red)">
        🔙<br>حذف<br><b>⠡</b>
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, speakAll, 'نطق النص')">
        🔊<br>نطق النص
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, handleCopy, 'نسخ')">
        📋<br>نسخ
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, handleClearAll, 'مسح الكل')">
        🗑️<br>مسح الكل
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, handleExport, 'حفظ كملف وورد')">
        💾<br>Word
    </div>

    <div class="nav-btn"
         onclick="unifiedControl(this, handleSpace, 'مسافة')"
         style="background:var(--cyan); color:#000;">
        ␣<br>مسافة
    </div>

    <!-- زر عد نقاط برايل الأصلي -->

    <div class="nav-btn"
         onclick="unifiedControl(this, speakBrailleCount, 'عدّ نقاط برايل')">
        ⠿<br>عدّ نقاط برايل
    </div>

    <!-- زر اكتشف اللغة الجديد -->

    <div class="nav-btn"
         onclick="unifiedControl(this, openLanguagePopup, 'اكتشف اللغة')">
        📖<br>اكتشف اللغة
        <b>المعجم الوسيط</b>
    </div>

</div>

<div id="pop-nums" class="popup-menu" onclick="event.stopPropagation()"></div>
<div id="pop-tashkeel" class="popup-menu" onclick="event.stopPropagation()"></div>
<div id="pop-punct" class="popup-menu" onclick="event.stopPropagation()"></div>
<div id="pop-hamzas" class="popup-menu" onclick="event.stopPropagation()"></div>

<!-- نافذة اكتشف اللغة -->

<div id="language-popup" onclick="event.stopPropagation()">

    <h3>📖 اكتشف اللغة</h3>

    <p style="text-align:center;color:white">
        اكتشف معنى الكلمة وجذرها من المعجم الوسيط.
    </p>

    <input
        id="language-word"
        type="text"
        placeholder="اكتبي كلمة عربية..."
        aria-label="الكلمة المراد البحث عنها"
    >

    <button class="language-action"
            onclick="searchLanguage()">
        🔍 ابحث في المعجم الوسيط
    </button>

    <button class="language-action"
            style="background:#333"
            onclick="closeLanguagePopup()">
        إغلاق
    </button>

    <p id="language-note">
        سيتم فتح نتائج البحث الموجهة إلى المعجم الوسيط الرسمي.
    </p>

</div>

<script>
    const CHARS = {
        'ا': '⠁', 'ب': '⠃', 'ت': '⠞', 'ث': '⠹',
        'ج': '⠚', 'ح': '⠱', 'خ': '⠭', 'د': '⠙',
        'ذ': '⠮', 'ر': '⠗', 'ز': '⠵', 'س': '⠎',
        'ش': '⠩', 'ص': '⠯', 'ض': '⠻', 'ط': '⠾',
        'ظ': '⠿', 'ع': '⠷', 'غ': '⠣', 'ف': '⠋',
        'ق': '⠟', 'ك': '⠅', 'ل': '⠇', 'م': '⠍',
        'ن': '⠝', 'هـ': '⠓', 'و': '⠺', 'ي': '⠊',
        'ة': '⠡', 'ى': '⠕', 'پ': '⠏', 'چ': '⠡',
        'ڤ': '⠧', 'گ': '⠛'
    };

    const HAMZAS = {
        'أ': '⠁⠃', 'إ': '⠌', 'ؤ': '⠳',
        'ئ': '⠽', 'ء': '⠄', 'آ': '⠜'
    };

    const TASHKEEL = {
        'َ': '⠂', 'ُ': '⠕', 'ِ': '⠐', 'ْ': '⠒',
        'ً': '⠘', 'ٌ': '⠰', 'ٍ': '⠡', 'ّ': '⠠⠠'
    };

    const PUNCT = {
        '.': '⠲', '،': '⠂', '؟': '⠦', '!': '⠖',
        '(': '⠶', ')': '⠶', '+': '⠖', '-': '⠒',
        '×': '⠦', '÷': '⠲'
    };

    const NUMS = {
        '1': '⠼⠁', '2': '⠼⠃', '3': '⠼⠉', '4': '⠼⠙',
        '5': '⠼⠑', '6': '⠼⠋', '7': '⠼⠛', '8': '⠼⠓',
        '9': '⠼⠊', '0': '⠼⠚'
    };

    const NAMES_MAP = {
        'َ': 'فتحة', 'ُ': 'ضمة', 'ِ': 'كسرة', 'ْ': 'سكون',
        'ً': 'تنوين فتح', 'ٌ': 'تنوين ضم', 'ٍ': 'تنوين كسر',
        'ّ': 'شدة'
    };

    let isBlindMode = false;
    let brailleBuffer = [];
    let lastTapTime = 0;
    let lastTarget = null;

    function speak(t) {
        window.speechSynthesis.cancel();

        const msg = new SpeechSynthesisUtterance(t);
        msg.lang = 'ar-SA';

        window.speechSynthesis.speak(msg);
    }

    function unifiedControl(element, action, description) {
        speak(description || element.innerText);

        if (!isBlindMode) {
            action();
        } else {
            const currentTime = new Date().getTime();
            const tapLength = currentTime - lastTapTime;

            if (
                lastTarget === element &&
                tapLength < 500 &&
                tapLength > 0
            ) {
                action();
                lastTarget = null;
            } else {
                lastTarget = element;
            }

            lastTapTime = currentTime;
        }
    }

    function handleInput(char, type = 'letter') {
        const out = document.getElementById('output');

        out.value += char;

        const bCode =
            CHARS[char] ||
            HAMZAS[char] ||
            TASHKEEL[char] ||
            PUNCT[char] ||
            NUMS[char] ||
            "";

        brailleBuffer.push(bCode);

        let speechText =
            type === 'tashkeel' ? NAMES_MAP[char] : char;

        speak(speechText);

        updateUI();
    }

    function handleSpace() {
        document.getElementById('output').value += " ";

        brailleBuffer.push(" ");

        updateUI();
    }

    function handleBackspace() {
        const out = document.getElementById('output');

        if (out.value.length > 0) {
            const lastChar = out.value.slice(-1);

            if (!isBlindMode) {
                speak("حذف " + lastChar);
            }

            out.value = out.value.slice(0, -1);

            brailleBuffer.pop();

            updateUI();
        }
    }

    function handleClearAll() {
        document.getElementById('output').value = "";

        brailleBuffer = [];

        updateUI();
    }

    function updateUI() {
        document.getElementById('braille-preview').innerText =
            brailleBuffer.join('');

        const out = document.getElementById('output');

        out.scrollTop = out.scrollHeight;
    }

    // مزامنة النص المكتوب أو الملصوق مع معاينة برايل

    document.getElementById('output').addEventListener('input', function() {
        const text = this.value;

        brailleBuffer = Array.from(text).map(char => {
            return CHARS[char] ||
                   (char === 'ه' ? CHARS['هـ'] : '') ||
                   HAMZAS[char] ||
                   TASHKEEL[char] ||
                   PUNCT[char] ||
                   NUMS[char] ||
                   (/[\u2800-\u28FF]/.test(char) ? char : char);
        });

        updateUI();
    });

    function openPopup(id, name) {
        document.getElementById('overlay').style.display = 'block';

        document.getElementById(id).classList.add('active');
    }

    function closeAllPopups() {
        document.querySelectorAll('.popup-menu').forEach(p => {
            p.classList.remove('active');
        });

        document.getElementById('language-popup').classList.remove('active');

        document.getElementById('overlay').style.display = 'none';
    }

    function toggleAppMode() {
        isBlindMode = !isBlindMode;

        document.getElementById('keyboard-sighted').style.display =
            isBlindMode ? 'none' : 'grid';

        document.getElementById('keyboard-blind').style.display =
            isBlindMode ? 'grid' : 'none';

        let msg = isBlindMode
            ? "تم تفعيل وضع الكفيف. استكشف باللمس واضغط مرتين للتنفيذ"
            : "تم تفعيل وضع المبصر. التنفيذ بلمسة واحدة مع النطق";

        speak(msg);

        renderAll();
    }

    // نسخ النص المكتوب كما هو، سواء كان عربيًا أو برايل

    async function handleCopy() {
        const text = document.getElementById('output').value;

        try {
            await navigator.clipboard.writeText(text);

        } catch (err) {
            const textArea = document.createElement("textarea");

            textArea.value = text;

            textArea.style.position = "fixed";
            textArea.style.left = "-9999px";

            document.body.appendChild(textArea);

            textArea.focus();
            textArea.select();

            document.execCommand('copy');

            document.body.removeChild(textArea);
        }
    }

    function renderAll() {
        const sighted = document.getElementById('keyboard-sighted');

        sighted.innerHTML = "";

        Object.keys(CHARS).forEach(c => {
            const btn = document.createElement('div');

            btn.className = "key-s";

            btn.innerHTML = `<span>${c}</span><b>${CHARS[c]}</b>`;

            btn.onclick = (e) => {
                e.stopPropagation();

                handleInput(c);
            };

            sighted.appendChild(btn);
        });

        const blind = document.getElementById('keyboard-blind');

        blind.innerHTML = "";

        const charKeys = Object.keys(CHARS);

        for (let i = 0; i < charKeys.length; i += 6) {
            const cluster = document.createElement('div');

            cluster.className = "cluster-main";

            const chunk = charKeys.slice(i, i + 6);

            const clusterRange =
                `أنت الآن في عنقود دائري من حرف ${chunk[0]} إلى حرف ${chunk[chunk.length-1]}`;

            cluster.onclick = (e) => {
                e.stopPropagation();

                speak(clusterRange);
            };

            chunk.forEach((c, index) => {
                const sub = document.createElement('div');

                sub.className = `sub-key pos-${index}`;

                sub.innerText = c;

                sub.onclick = (e) => {
                    e.stopPropagation();

                    const fullDesc =
                        `${clusterRange}، وأنت الآن ضغطت على حرف ${c}`;

                    unifiedControl(
                        sub,
                        () => handleInput(c),
                        fullDesc
                    );
                };

                cluster.appendChild(sub);
            });

            blind.appendChild(cluster);
        }

        const fill = (id, map, type, title) => {
            const el = document.getElementById(id);

            el.innerHTML = "";

            Object.keys(map).forEach(k => {
                const btn = document.createElement('div');

                btn.className = "key-s";

                btn.innerHTML = `<span>${k}</span><b>${map[k]}</b>`;

                btn.onclick = (e) => {
                    e.stopPropagation();

                    let desc =
                        type === 'tashkeel' ? NAMES_MAP[k] : k;

                    unifiedControl(
                        btn,
                        () => handleInput(k, type),
                        desc
                    );
                };

                el.appendChild(btn);
            });
        };

        fill('pop-nums', NUMS, 'number', 'رقم');

        fill('pop-tashkeel', TASHKEEL, 'tashkeel', 'تشكيل');

        fill('pop-punct', PUNCT, 'punct', 'ترقيم');

        fill('pop-hamzas', HAMZAS, 'hamza', 'همزة');
    }

    // ==========================================
    // عدّ نقاط برايل بالصوت فقط
    // ==========================================

    function getBrailleForChar(char) {
        return CHARS[char] ||
               (char === 'ه' ? CHARS['هـ'] : '') ||
               HAMZAS[char] ||
               TASHKEEL[char] ||
               PUNCT[char] ||
               NUMS[char] ||
               (/^[\u2800-\u28FF]$/.test(char) ? char : '');
    }

    function getBrailleDots(cell) {
        const value = cell.codePointAt(0) - 0x2800;

        const positions = [
            'أقصى اليسار في الأعلى',
            'أقصى اليسار في المنتصف',
            'أقصى اليسار في الأسفل',
            'أقصى اليمين في الأعلى',
            'أقصى اليمين في المنتصف',
            'أقصى اليمين في الأسفل'
        ];

        const dots = [];

        for (let i = 0; i < 6; i++) {
            if (value & (1 << i)) {
                dots.push({
                    number: i + 1,
                    position: positions[i]
                });
            }
        }

        return dots;
    }

    function describeBrailleCell(cell, cellNumber) {
        const dots = getBrailleDots(cell);

        let speech =
            `الخلية رقم ${cellNumber}. عدد النقاط البارزة فيها ${dots.length} نقطة. `;

        if (dots.length === 0) {
            speech += 'لا توجد نقاط بارزة في هذه الخلية. ';
        } else {
            dots.forEach(dot => {
                speech +=
                    `النقطة رقم ${dot.number} في ${dot.position}. `;
            });
        }

        return {
            speech: speech,
            dotsCount: dots.length
        };
    }

    function speakBrailleSequence(parts) {
        window.speechSynthesis.cancel();

        let index = 0;

        function speakNext() {
            if (index >= parts.length) return;

            const utterance =
                new SpeechSynthesisUtterance(parts[index]);

            utterance.lang = 'ar-SA';
            utterance.rate = 0.9;

            utterance.onend = function() {
                index++;
                speakNext();
            };

            utterance.onerror = function() {
                index++;
                speakNext();
            };

            window.speechSynthesis.speak(utterance);
        }

        speakNext();
    }

    function speakBrailleCount() {
        const text = document.getElementById('output').value;

        if (!text.trim()) {
            speak('لا يوجد نص لحساب نقاط برايل.');
            return;
        }

        let totalCells = 0;
        let totalDots = 0;
        let speechParts = [];

        const characters = Array.from(text);

        characters.forEach(char => {
            if (char === ' ') return;

            const brailleCode = getBrailleForChar(char);

            if (!brailleCode) return;

            const cells = Array.from(brailleCode).filter(cell =>
                /^[\u2800-\u28FF]$/.test(cell)
            );

            if (cells.length === 0) return;

            const characterName = NAMES_MAP[char] || char;

            speechParts.push(
                `الحرف ${characterName}. رمز برايل المقابل له هو ${brailleCode}.`
            );

            let characterDots = 0;

            cells.forEach((cell, index) => {
                totalCells++;

                const result =
                    describeBrailleCell(cell, index + 1);

                characterDots += result.dotsCount;

                totalDots += result.dotsCount;

                speechParts.push(result.speech);
            });

            speechParts.push(
                `إجمالي النقاط البارزة في الحرف ${characterName} هو ${characterDots} نقطة.`
            );
        });

        if (totalCells === 0) {
            speak('لم أجد خلايا برايل معروفة في النص.');
            return;
        }

        speechParts.push(
            `انتهى عد الحروف. إجمالي خلايا برايل في النص هو ${totalCells} خلية.`
        );

        speechParts.push(
            `إجمالي النقاط البارزة في النص كله هو ${totalDots} نقطة.`
        );

        speakBrailleSequence(speechParts);
    }

    // ==========================================
    // اكتشف اللغة - البحث في المعجم الوسيط
    // ==========================================

    function openLanguagePopup() {
        const popup =
            document.getElementById('language-popup');

        const input =
            document.getElementById('language-word');

        document.getElementById('overlay').style.display = 'block';

        popup.classList.add('active');

        const currentText =
            document.getElementById('output').value.trim();

        // تعبئة الكلمة تلقائيًا إذا كان النص كلمة واحدة

        if (currentText && !/\s/.test(currentText)) {
            input.value = currentText;
        } else {
            input.value = '';
        }

        setTimeout(() => input.focus(), 250);
    }

    function closeLanguagePopup() {
        document.getElementById('language-popup')
            .classList.remove('active');

        const otherPopups =
            document.querySelectorAll('.popup-menu.active');

        if (otherPopups.length === 0) {
            document.getElementById('overlay').style.display = 'none';
        }
    }

    function searchLanguage() {
        const input =
            document.getElementById('language-word');

        const word = input.value.trim();

        if (!word) {
            speak('من فضلك اكتبي الكلمة التي تريدين البحث عنها');

            input.focus();

            return;
        }

        if (/\s/.test(word)) {
            speak('من فضلك أدخلي كلمة واحدة فقط');

            return;
        }

        speak('جاري البحث عن ' + word + ' في المعجم الوسيط');

        const query = encodeURIComponent(
            'site:arabicacademy.gov.eg/ar/محرك-البحث/معجم/dic-19/ ' + word
        );

        const searchURL =
            'https://www.google.com/search?q=' + query;

        window.open(searchURL, '_blank', 'noopener');

        closeLanguagePopup();
    }

    // ==========================================
    // نهاية إضافة اكتشف اللغة
    // ==========================================

    // دالة تصدير Word الأصلية

    function handleExport() {
        const text = document.getElementById('output').value;

        if (!text) return;

        const { Document, Packer, Paragraph, TextRun } = docx;

        const doc = new Document({
            sections: [{
                properties: {
                    page: {
                        margin: {
                            top: 1000,
                            right: 1000,
                            bottom: 1000,
                            left: 1000
                        }
                    }
                },

                children: [
                    new Paragraph({
                        children: [
                            new TextRun({
                                text: "نظام برايل الهجين المطور",
                                bold: true,
                                size: 32,
                                rightToLeft: true
                            })
                        ],

                        alignment: "center",
                        bidirectional: true
                    }),

                    new Paragraph({ text: "" }),

                    new Paragraph({
                        children: [
                            new TextRun({
                                text: "النص العربي: ",
                                bold: true,
                                rightToLeft: true
                            }),

                            new TextRun({
                                text: text,
                                rightToLeft: true
                            })
                        ],

                        alignment: "right",
                        bidirectional: true
                    }),

                    new Paragraph({
                        children: [
                            new TextRun({
                                text: "نص برايل: ",
                                bold: true,
                                rightToLeft: true
                            }),

                            new TextRun({
                                text: brailleBuffer.join(''),
                                rightToLeft: false
                            })
                        ],

                        alignment: "right",
                        bidirectional: true
                    })
                ]
            }]
        });

        Packer.toBlob(doc).then(blob => {
            saveAs(blob, "Braille_Export.docx");
        });
    }

    function speakAll() {
        const t = document.getElementById('output').value;

        speak(t || "لا يوجد نص مكتوب حالياً");
    }

    // تشغيل لوحة المفاتيح الأصلية

    renderAll();

</script>

</body>
</html>
