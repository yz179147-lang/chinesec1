<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>第一課【你會怎麼回答？】數位預習單 (iPad版)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
    
    <style>
        body {
            font-family: 'BiauKai', 'DFKai-SB', '楷體-繁', 'TW-Kai', serif;
            background-color: #e5e7eb;
            color: #111827;
            -webkit-overflow-scrolling: touch;
            overscroll-behavior: none;
        }

        #app-wrapper {
            position: relative;
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        #worksheet-content {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 2rem;
            padding: 2rem 0 8rem 0; 
            width: 100%;
            z-index: 1;
        }

        .page-container {
            width: 210mm;
            height: 297mm;
            padding: 15mm 20mm;
            background-color: white;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            border-radius: 8px;
            box-sizing: border-box;
            overflow: hidden;
            position: relative;
        }

        /* 畫布層 */
        #draw-layer {
            position: absolute;
            top: 0;
            left: 0;
            z-index: 10;
            touch-action: none; 
        }

        /* 互動/滑動模式：畫布不再攔截點擊事件 */
        #draw-layer.scroll-mode {
            pointer-events: none;
            touch-action: auto;
        }

        /* 工具列 */
        #toolbar {
            position: fixed;
            bottom: 1rem;
            left: 50%;
            transform: translateX(-50%);
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            padding: 0.75rem 1.5rem;
            border-radius: 9999px;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.2);
            display: flex;
            gap: 1rem;
            z-index: 50;
            border: 1px solid #e5e7eb;
        }

        .tool-btn {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 3.5rem;
            height: 3.5rem;
            border-radius: 50%;
            border: 2px solid transparent;
            background: transparent;
            font-size: 1.5rem;
            transition: all 0.2s;
            color: #4b5563;
            cursor: pointer;
        }
        .tool-btn span { font-size: 0.7rem; font-weight: bold; margin-top: 2px; }
        .tool-btn.active { background-color: #eff6ff; border-color: #3b82f6; color: #1d4ed8; transform: scale(1.1); }
        .tool-btn:active { transform: scale(0.95); }

        /* 學習單互動元件樣式 */
        .blank {
            display: inline-flex;
            justify-content: center;
            align-items: flex-end;
            min-width: 80px;
            border-bottom: 2px solid #4b5563;
            text-align: center;
            padding: 0 8px 2px 8px;
            margin: 0 4px;
            position: relative;
            height: 1.7rem;
            vertical-align: bottom;
            transition: background-color 0.2s;
        }
        .blank-lg { min-width: 140px; }
        .blank.drag-over { background-color: #eff6ff; border-color: #3b82f6; }
        
        .user-answer { font-weight: bold; color: #2563eb; } /* 學生填入/拖曳的答案顏色 */
        .answer-text { 
            display: none; 
            position: absolute; 
            bottom: 100%; 
            color: #ef4444; 
            font-weight: bold; 
            font-size: 1rem;
            white-space: nowrap;
        }
        .show-answers .answer-text { display: inline-block; }

        /* 選項點擊功能 */
        .option { 
            padding: 2px 8px; 
            margin: 0 2px; 
            border: 2px solid transparent; 
            border-radius: 9999px; 
            transition: all 0.2s; 
            cursor: pointer;
            line-height: 1;
            display: inline-block;
        }
        /* 學生點擊後的圈選效果 */
        .option.user-selected { border-color: #3b82f6; background-color: #eff6ff; color: #1d4ed8; font-weight: bold; }
        /* 教師顯示正確解答效果 */
        .show-answers .option.correct { border-color: #ef4444; background-color: #fef2f2; color: #ef4444; font-weight: bold; }

        /* 拖曳標籤 */
        .tag { 
            display: inline-block; 
            padding: 6px 14px; 
            background-color: white; 
            border: 1px solid #d1d5db; 
            border-radius: 9999px; 
            font-weight: bold; 
            margin-right: 8px; 
            margin-bottom: 6px; 
            box-shadow: 0 2px 4px rgba(0,0,0,0.05);
            cursor: grab;
            touch-action: none; /* 讓移動設備能順利抓取 */
            user-select: none;
            transition: transform 0.1s;
        }
        .tag:active { cursor: grabbing; transform: scale(0.95); }

        .zy-box { display: inline-block; width: 60px; border-bottom: 2px solid #4b5563; text-align: center; font-weight: bold; margin: 0 4px; font-family: sans-serif; letter-spacing: 1px; height: 1.25rem; vertical-align: bottom; }
        .show-answers .zy-box { color: #ef4444; }
        body:not(.show-answers) .zy-box { color: transparent; }
        .writing-box { display: inline-flex; justify-content: center; align-items: center; width: 36px; height: 36px; border: 2px solid #4b5563; margin: 0 4px; vertical-align: middle; position: relative; background-image: linear-gradient(to right, transparent 48%, rgba(156, 163, 175, 0.4) 49%, rgba(156, 163, 175, 0.4) 51%, transparent 52%), linear-gradient(to bottom, transparent 48%, rgba(156, 163, 175, 0.4) 49%, rgba(156, 163, 175, 0.4) 51%, transparent 52%); }
        .zhuyin-group { display: inline-flex; align-items: center; margin: 0 0.25rem; }
        .zhuyin-text { position: relative; display: inline-block; margin-left: 4px; margin-right: 10px; }
        .zy-chars { display: inline-block; writing-mode: vertical-rl; text-orientation: upright; font-size: 0.85rem; line-height: 1; color: #4b5563; letter-spacing: -1px; }
        .zy-tone { position: absolute; right: -10px; bottom: 2px; font-size: 0.85rem; color: #4b5563; font-weight: bold; }

        .pdf-mode #worksheet-content { padding: 0 !important; gap: 0 !important; }
        .pdf-mode .page-container { box-shadow: none !important; border-radius: 0 !important; margin: 0 !important; }
        
        @media print {
            body { background-color: white; }
            #worksheet-content { gap: 0; padding: 0; }
            .page-container { margin: 0; padding: 10mm; box-shadow: none; page-break-after: always; break-after: page; }
            .page-container:last-child { page-break-after: auto; break-after: auto; }
            .no-print { display: none !important; }
            * { -webkit-print-color-adjust: exact !important; print-color-adjust: exact !important; }
        }
    </style>
</head>
<body>

    <!-- iPad 專屬底部工具列 -->
    <div id="toolbar" class="no-print">
        <button class="tool-btn" id="btn-scroll" onclick="setMode('scroll')" title="點擊或拖曳作答">
            🖐️<span>互動</span>
        </button>
        <button class="tool-btn active" id="btn-pen-black" onclick="setMode('pen-black')" title="黑筆">
            🖋️<span>黑筆</span>
        </button>
        <button class="tool-btn" id="btn-pen-blue" onclick="setMode('pen-blue')" title="藍筆">
            🖊️<span>藍筆</span>
        </button>
        <button class="tool-btn" id="btn-eraser" onclick="setMode('eraser')" title="橡皮擦">
            🧽<span>橡皮擦</span>
        </button>
        
        <div class="w-px h-10 bg-gray-300 mx-2 self-center"></div>
        
        <button class="tool-btn" onclick="clearCanvas()" title="清除全部筆跡">
            🗑️<span>清空</span>
        </button>
        <button class="tool-btn" id="btn-answer" onclick="toggleAnswers()" title="顯示解答">
            👁️<span>解答</span>
        </button>
        <button class="tool-btn" onclick="downloadPDF()" title="交卷 / 儲存成 PDF">
            📥<span>交卷</span>
        </button>
    </div>

    <!-- 學習單與畫布的核心容器 -->
    <div id="app-wrapper">
        <!-- 畫布層：負責接收 Apple Pencil 筆跡 -->
        <canvas id="draw-layer"></canvas>

        <!-- 學習單內容層 -->
        <div id="worksheet-content">
            
            <!-- 第 1 頁 -->
            <div class="page-container flex flex-col justify-start">
                <header class="border-b-2 border-gray-800 pb-2 mb-3">
                    <div class="text-center mb-1">
                        <span class="bg-gray-800 text-white px-3 py-1 rounded text-sm tracking-widest">可切換【互動模式】作答，或使用【畫筆】書寫</span>
                    </div>
                    <h1 class="text-2xl font-bold text-center mb-3">第一課【你會怎麼回答？】考前衝刺單</h1>
                    <div class="flex justify-between text-base font-bold px-4">
                        <div>班級：<span class="inline-block w-20 border-b border-gray-600"></span></div>
                        <div>姓名：<span class="inline-block w-24 border-b border-gray-600"></span></div>
                        <div>得分：<span class="inline-block w-16 border-b border-gray-600"></span></div>
                    </div>
                </header>

                <section class="mb-3">
                    <h2 class="text-xl font-bold mb-2 bg-gray-100 p-2 rounded border-l-4 border-gray-800">一、 課文重點與寫作架構</h2>
                    <p class="text-sm text-gray-600 mb-2 px-2">💡 <span class="font-bold text-blue-600">作答提示：</span>請將下方關鍵字拖曳或直接用畫筆寫在底線上。</p>
                    
                    <div class="bg-gray-50 border border-gray-300 p-2 rounded-lg mb-2 text-center shadow-inner">
                        <span class="tag draggable-tag !mb-0 text-sm">我的想法</span>
                        <span class="tag draggable-tag !mb-0 text-sm">神態自若</span>
                        <span class="tag draggable-tag !mb-0 text-sm">如何解決</span>
                    </div>

                    <div class="text-base space-y-2 px-2">
                        <p class="leading-relaxed">1. 邱吉爾收到寫著「傻瓜」的紙條時，他並沒有勃然大怒，而是展現極高的處事能力，<span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">神態自若</span></span> 地幽默化解。</p>
                        <p class="leading-relaxed">2. 觀察這課的「寫作架構」，文章通常會依照以下三個步驟來安排：<br>
                           ① 描述「遭遇問題」 ➜ ② 接著寫主角 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">如何解決</span></span> 難題 ➜ ③ 最後提出 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">我的想法</span></span> 作結。</p>
                    </div>
                </section>

                <section>
                    <h2 class="text-xl font-bold mb-2 bg-gray-100 p-2 rounded border-l-4 border-gray-800">二、 音形掃雷大作戰</h2>
                    <p class="text-sm text-gray-600 mb-2 px-2">這裡是考試最容易失分的地方，請仔細看清楚題目要求填寫！</p>
                    <div class="space-y-3 px-2 text-base">
                        <div class="grid grid-cols-2 gap-3">
                            <div class="border border-blue-300 p-2 rounded-lg bg-blue-50 shadow-sm">
                                <div class="font-bold mb-1 text-blue-800">🔹 【 埋 】的多音字</div>
                                <ul class="space-y-1 pl-2">
                                    <li>1. 掩「埋」：<span class="zy-box">ㄇㄢˊ</span></li>
                                    <li>2. 「埋」怨：<span class="zy-box">ㄇㄞˊ</span></li>
                                    <li>3. 「埋」伏：<span class="zy-box">ㄇㄞˊ</span></li>
                                </ul>
                            </div>
                            <div class="border border-blue-300 p-2 rounded-lg bg-blue-50 shadow-sm">
                                <div class="font-bold mb-1 text-blue-800">🔹 【 擔 】的多音字</div>
                                <ul class="space-y-1 pl-2">
                                    <li>1. 「擔」任：<span class="zy-box">ㄉㄢ</span></li>
                                    <li>2. 重「擔」：<span class="zy-box">ㄉㄢˋ</span></li>
                                    <li>3. 承「擔」：<span class="zy-box">ㄉㄢ</span></li>
                                </ul>
                            </div>
                        </div>

                        <div class="border border-gray-300 p-2 rounded-lg shadow-sm">
                            <div class="font-bold mb-2 text-gray-800">🔹 容易讀錯的字音（請寫出正確注音）</div>
                            <div class="grid grid-cols-3 gap-y-2 gap-x-1 pl-2">
                                <div>1. 「聆」聽：<span class="zy-box">ㄌㄧㄥˊ</span></div>
                                <div>2. 廣「博」：<span class="zy-box">ㄅㄛˊ</span></div>
                                <div>3. 「論」點：<span class="zy-box">ㄌㄨㄣˋ</span></div>
                                <div>4. 束「縛」：<span class="zy-box">ㄈㄨˊ</span></div>
                                <div>5. 「艱」辛：<span class="zy-box">ㄐㄧㄢ</span></div>
                                <div>6. 音「符」：<span class="zy-box">ㄈㄨˊ</span></div>
                            </div>
                        </div>

                        <div class="border border-gray-300 p-2 rounded-lg shadow-sm">
                            <div class="font-bold mb-2 text-gray-800">🔹 看注音寫國字（小心部首陷阱！）</div>
                            <div class="grid grid-cols-3 gap-y-3 gap-x-1 pl-2">
                                <div class="flex items-center">1. 傳<div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">遞</span></div><span class="zhuyin-text"><span class="zy-chars">ㄉㄧ</span><span class="zy-tone">ˋ</span></span></div></div>
                                <div class="flex items-center">2. 麻<div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">痺</span></div><span class="zhuyin-text"><span class="zy-chars">ㄅㄧ</span><span class="zy-tone">ˋ</span></span></div></div>
                                <div class="flex items-center">3. 災<div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">患</span></div><span class="zhuyin-text"><span class="zy-chars">ㄏㄨㄢ</span><span class="zy-tone">ˋ</span></span></div></div>
                                <div class="flex items-center">4. 懸<div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">河</span></div><span class="zhuyin-text"><span class="zy-chars">ㄏㄜ</span><span class="zy-tone">ˊ</span></span></div></div>
                                <div class="flex items-center">5. 肉<div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">搏</span></div><span class="zhuyin-text"><span class="zy-chars">ㄅㄛ</span><span class="zy-tone">ˊ</span></span></div></div>
                                <div class="flex items-center">6. <div class="zhuyin-group"><div class="writing-box"><span class="answer-text !bottom-1 !text-xl">傻</span></div><span class="zhuyin-text"><span class="zy-chars">ㄕㄚ</span><span class="zy-tone">ˇ</span></span></div>瓜</div>
                            </div>
                        </div>
                    </div>
                </section>
            </div>

            <!-- 第 2 頁 -->
            <div class="page-container flex flex-col justify-start">
                <section class="mb-4 pt-1">
                    <h2 class="text-xl font-bold mb-2 bg-gray-100 p-2 rounded border-l-4 border-gray-800">三、 字形辨正連連看</h2>
                    <p class="text-sm text-gray-600 mb-2 px-2">💡 <span class="font-bold text-blue-600">請切換至「🖐️ 互動」模式。</span> 直接點擊括號內正確的國字，系統會自動幫你圈起來！</p>
                    <div class="space-y-2 text-base px-2 bg-white border border-gray-200 p-3 rounded-lg shadow-sm">
                        <p>1. 他花了一整晚的時間，終於把演講的草（ <span class="option selectable-option">搞</span> ／ <span class="option selectable-option correct">稿</span> ／ <span class="option selectable-option">槁</span> ）寫完了。</p>
                        <p>2. 這位學者不僅學識廣（ <span class="option selectable-option">搏</span> ／ <span class="option selectable-option">縛</span> ／ <span class="option selectable-option correct">博</span> ），待人也非常謙虛。</p>
                        <p>3. 面對突發狀況，他依然神態自（ <span class="option selectable-option">苦</span> ／ <span class="option selectable-option correct">若</span> ／ <span class="option selectable-option">惹</span> ），立刻把事情（ <span class="option selectable-option correct">搞</span> ／ <span class="option selectable-option">稿</span> ／ <span class="option selectable-option">槁</span> ）定了。</p>
                        <p>4. 為了準備明天的比賽，他正仔細地（ <span class="option selectable-option">伶</span> ／ <span class="option selectable-option">玲</span> ／ <span class="option selectable-option">鈴</span> ／ <span class="option selectable-option correct">聆</span> ）聽對手的分析錄音。</p>
                    </div>
                </section>

                <section class="mb-2">
                    <h2 class="text-xl font-bold mb-2 bg-gray-100 p-2 rounded border-l-4 border-gray-800">四、 必考成語大補帖</h2>
                    <p class="text-sm text-gray-600 mb-2 px-2">💡 <span class="font-bold text-blue-600">請切換至「🖐️ 互動」模式。</span> 長按下方黃色區塊的成語標籤，拖曳並放入對應的底線中。</p>
                    
                    <div class="bg-yellow-50 border border-yellow-300 p-2 rounded-lg mb-3 text-center shadow-inner">
                        <span class="tag draggable-tag text-sm !mb-1">瓜田李下</span>
                        <span class="tag draggable-tag text-sm !mb-1">虛懷若谷</span>
                        <span class="tag draggable-tag text-sm !mb-1">防患未然</span>
                        <span class="tag draggable-tag text-sm !mb-1">口若懸河</span>
                        <span class="tag draggable-tag text-sm !mb-1">高談闊論</span>
                        <span class="tag draggable-tag text-sm !mb-1">神態自若</span>
                    </div>

                    <div class="space-y-3 text-base px-2">
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">1.</span>
                            <div class="flex-1 leading-normal">
                                比喻容易引起懷疑的場合。擔任裁判的他，為了避免 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">瓜田李下</span></span> ，避開了親戚場次。
                            </div>
                        </div>
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">2.</span>
                            <div class="flex-1 leading-normal">
                                趁禍患還未發生之前就加以防備。在颱風季來臨前清理水溝，就是 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">防患未然</span></span> 。
                            </div>
                        </div>
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">3.</span>
                            <div class="flex-1 leading-normal">
                                形容人非常謙虛，能接納他人的意見。老教授學問淵博，但依然 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">虛懷若谷</span></span> 。
                            </div>
                        </div>
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">4.</span>
                            <div class="flex-1 leading-normal">
                                神情態度從容不迫。即使面對台下觀眾尖銳的提問，他依然 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">神態自若</span></span> 地回答。
                            </div>
                        </div>
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">5.</span>
                            <div class="flex-1 leading-normal">
                                比喻能言善道，說話流利。只要一談到棒球，他就會 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">口若懸河</span></span> 、滔滔不絕。
                            </div>
                        </div>
                        <div class="flex items-start">
                            <span class="font-bold mr-2 mt-1">6.</span>
                            <div class="flex-1 leading-normal">
                                暢快而無拘束的談論。下課時間，幾位同學聚在走廊上 <span class="blank blank-lg droppable-blank"><span class="user-answer"></span><span class="answer-text">高談闊論</span></span> 分享趣事。
                            </div>
                        </div>
                    </div>
                </section>
            </div>

        </div>
    </div>

    <script>
        // ==========================================
        // 1. 互動畫布與 Apple Pencil 繪圖邏輯
        // ==========================================
        const canvas = document.getElementById('draw-layer');
        const ctx = canvas.getContext('2d');
        const content = document.getElementById('worksheet-content');

        let isDrawing = false;
        let currentMode = 'pen-black'; // scroll, pen-black, pen-blue, eraser

        function resizeCanvas() {
            const tempCanvas = document.createElement('canvas');
            tempCanvas.width = canvas.width;
            tempCanvas.height = canvas.height;
            const tempCtx = tempCanvas.getContext('2d');
            if (canvas.width > 0 && canvas.height > 0) {
                tempCtx.drawImage(canvas, 0, 0);
            }

            const rect = content.getBoundingClientRect();
            const dpr = window.devicePixelRatio || 1;
            
            canvas.style.width = `${rect.width}px`;
            canvas.style.height = `${rect.height}px`;
            canvas.width = rect.width * dpr;
            canvas.height = rect.height * dpr;
            
            ctx.scale(dpr, dpr);
            ctx.lineCap = 'round';
            ctx.lineJoin = 'round';
            
            if (tempCanvas.width > 0 && tempCanvas.height > 0) {
                ctx.save();
                ctx.setTransform(1, 0, 0, 1, 0, 0);
                ctx.drawImage(tempCanvas, 0, 0);
                ctx.restore();
            }
        }

        window.addEventListener('load', () => {
            initInteractions(); // 啟動互動與拖曳事件
            setTimeout(resizeCanvas, 100);
        });
        window.addEventListener('resize', resizeCanvas);

        function setMode(mode) {
            currentMode = mode;
            document.querySelectorAll('.tool-btn').forEach(btn => btn.classList.remove('active'));
            document.getElementById(`btn-${mode.split('-')[0] === 'pen' ? mode : mode}`).classList.add('active');

            if (mode === 'scroll') {
                canvas.classList.add('scroll-mode');
            } else {
                canvas.classList.remove('scroll-mode');
            }
        }

        function getPos(e) {
            const rect = canvas.getBoundingClientRect();
            return {
                x: e.clientX - rect.left,
                y: e.clientY - rect.top
            };
        }

        canvas.addEventListener('pointerdown', (e) => {
            if (currentMode === 'scroll') return;
            e.preventDefault();
            isDrawing = true;
            const pos = getPos(e);
            ctx.beginPath();
            ctx.moveTo(pos.x, pos.y);

            if (currentMode === 'eraser') {
                ctx.globalCompositeOperation = 'destination-out';
                ctx.lineWidth = 30;
            } else {
                ctx.globalCompositeOperation = 'source-over';
                ctx.lineWidth = 2; 
                ctx.strokeStyle = currentMode === 'pen-black' ? '#111827' : '#2563eb';
            }
        });

        canvas.addEventListener('pointermove', (e) => {
            if (!isDrawing || currentMode === 'scroll') return;
            e.preventDefault();
            const pos = getPos(e);
            ctx.lineTo(pos.x, pos.y);
            ctx.stroke();

            // Apple Pencil 壓力感應
            if (e.pressure && e.pointerType === 'pen' && currentMode !== 'eraser') {
                ctx.lineWidth = 2 * (e.pressure * 1.5 + 0.5);
            }
        });

        canvas.addEventListener('pointerup', () => { if (isDrawing) { isDrawing = false; ctx.closePath(); }});
        canvas.addEventListener('pointercancel', () => isDrawing = false);
        canvas.addEventListener('pointerout', () => isDrawing = false);

        function clearCanvas() {
            if (confirm('確定要清除所有筆跡嗎？（文字拖曳與點選的答案不會被清除）')) {
                ctx.clearRect(0, 0, canvas.width, canvas.height);
            }
        }

        // ==========================================
        // 2. HTML 元素點選與拖曳互動邏輯 (供「互動模式」使用)
        // ==========================================
        let activeDragEl = null;
        let ghostEl = null;

        function initInteractions() {
            // -- 功能一：點選選項自動圈起來 --
            document.querySelectorAll('.selectable-option').forEach(opt => {
                opt.addEventListener('click', function(e) {
                    if (currentMode !== 'scroll') return; // 必須是互動模式才能點擊
                    const parentLine = this.closest('p');
                    parentLine.querySelectorAll('.selectable-option').forEach(sibling => {
                        sibling.classList.remove('user-selected');
                    });
                    this.classList.add('user-selected');
                });
            });

            // -- 功能二：iPad 友善的手指/滑鼠拖曳填空 --
            const tags = document.querySelectorAll('.draggable-tag');
            tags.forEach(tag => {
                tag.addEventListener('touchstart', handleDragStart, {passive: false});
                tag.addEventListener('mousedown', handleDragStart);
            });

            document.addEventListener('touchmove', handleDragMove, {passive: false});
            document.addEventListener('mousemove', handleDragMove);
            
            document.addEventListener('touchend', handleDragEnd);
            document.addEventListener('mouseup', handleDragEnd);
        }

        function handleDragStart(e) {
            if (currentMode !== 'scroll') return; // 必須在互動模式才能拖曳
            
            activeDragEl = e.target.closest('.draggable-tag');
            if(!activeDragEl) return;

            // 觸控螢幕為了防止捲動，攔截預設事件
            if(e.type === 'touchstart') e.preventDefault(); 

            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;

            // 創造一個跟隨滑鼠/手指的「幽靈分身」
            ghostEl = activeDragEl.cloneNode(true);
            ghostEl.style.position = 'fixed';
            ghostEl.style.zIndex = '1000';
            ghostEl.style.opacity = '0.9';
            ghostEl.style.pointerEvents = 'none'; // 讓滑鼠能穿透分身偵測底下的底線
            ghostEl.style.boxShadow = '0 10px 15px -3px rgba(0,0,0,0.3)';
            document.body.appendChild(ghostEl);

            moveGhost(clientX, clientY);
            activeDragEl.style.opacity = '0.3';
        }

        function moveGhost(x, y) {
            if (!ghostEl) return;
            ghostEl.style.left = (x - ghostEl.offsetWidth / 2) + 'px';
            ghostEl.style.top = (y - ghostEl.offsetHeight / 2) + 'px';
        }

        function handleDragMove(e) {
            if (!activeDragEl || !ghostEl) return;
            e.preventDefault(); // 拖曳時禁止畫面捲動

            const clientX = e.touches ? e.touches[0].clientX : e.clientX;
            const clientY = e.touches ? e.touches[0].clientY : e.clientY;
            moveGhost(clientX, clientY);
            
            // 偵測底線並加入高亮效果
            const elementBelow = document.elementFromPoint(clientX, clientY);
            document.querySelectorAll('.drag-over').forEach(el => el.classList.remove('drag-over'));
            
            if (elementBelow) {
                const dropZone = elementBelow.closest('.droppable-blank');
                if (dropZone) dropZone.classList.add('drag-over');
            }
        }

        function handleDragEnd(e) {
            if (!activeDragEl || !ghostEl) return;
            
            const clientX = e.changedTouches ? e.changedTouches[0].clientX : e.clientX;
            const clientY = e.changedTouches ? e.changedTouches[0].clientY : e.clientY;
            
            // 找出手指離開時，正下方是不是填空底線
            const elementBelow = document.elementFromPoint(clientX, clientY);
            if (elementBelow) {
                const dropZone = elementBelow.closest('.droppable-blank');
                if (dropZone) {
                    // 把標籤文字寫入底線中
                    dropZone.querySelector('.user-answer').innerText = activeDragEl.innerText;
                    dropZone.classList.remove('drag-over');
                }
            }

            // 清理狀態
            activeDragEl.style.opacity = '1';
            ghostEl.remove();
            ghostEl = null;
            activeDragEl = null;
        }

        // ==========================================
        // 3. 解答切換與 PDF 輸出
        // ==========================================
        let answersVisible = false;
        function toggleAnswers() {
            answersVisible = !answersVisible;
            const body = document.body;
            const btn = document.getElementById('btn-answer');
            
            if (answersVisible) {
                body.classList.add('show-answers');
                btn.classList.add('active');
            } else {
                body.classList.remove('show-answers');
                btn.classList.remove('active');
            }
        }

        function downloadPDF() {
            const element = document.getElementById('app-wrapper');
            const toolbar = document.getElementById('toolbar');
            const originalText = document.getElementById('btn-pdf-text') ? document.getElementById('btn-pdf-text').innerText : '一鍵下載 PDF';
            
            // 隱藏工具列並設定為 PDF 輸出模式
            toolbar.style.display = 'none';
            element.classList.add('pdf-mode');
            resizeCanvas();

            const opt = {
                margin:       0,
                filename:     '我的_考前衝刺預習單.pdf',
                image:        { type: 'jpeg', quality: 1 },
                html2canvas:  { 
                    scale: 2, 
                    useCORS: true,
                    letterRendering: true,
                    scrollY: 0,
                    windowWidth: document.getElementById('worksheet-content').scrollWidth 
                },
                jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
            };

            html2pdf().set(opt).from(element).save().then(() => {
                toolbar.style.display = 'flex';
                element.classList.remove('pdf-mode');
            }).catch(err => {
                console.error("PDF 產生失敗：", err);
                toolbar.style.display = 'flex';
                element.classList.remove('pdf-mode');
            });
        }
    </script>
</body>
</html>
