<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 體檢報告智慧分析助手</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.tailwindcss.com?plugins=typography"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <script src="https://cdn.jsdelivr.net/npm/marked/marked.min.js"></script>
    <style>
        body { background-color: #F0F4F8; }
        .medical-card { background: white; border-radius: 1rem; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.05), 0 2px 4px -1px rgba(0, 0, 0, 0.03); }
        /* 美化表格結構 */
        .prose table { width: 100%; border-collapse: collapse; margin-top: 1rem; margin-bottom: 1rem; }
        .prose th, .prose td { border: 1px solid #e2e8f0; padding: 0.75rem; text-align: left; }
        .prose th { background-color: #f8fafc; font-weight: 600; color: #1e293b; }
    </style>
</head>
<body class="min-h-screen font-sans text-slate-800">

    <header class="bg-white border-b border-slate-200 sticky top-0 z-50">
        <div class="max-w-4xl mx-auto px-4 py-3 flex flex-col sm:flex-row justify-between items-center gap-3">
            <div class="flex items-center gap-2">
                <span class="text-2xl">📑</span>
                <h1 class="text-xl font-bold text-slate-900">AI 體檢報告智慧分析助手</h1>
            </div>
            <div class="w-full sm:w-auto flex items-center gap-2 bg-slate-100 px-3 py-1.5 rounded-lg border border-slate-300">
                <i class="fa-solid fa-key text-slate-400 text-sm"></i>
                <input type="password" id="apiKey" placeholder="請輸入 Gemini API Key" 
                       class="bg-transparent text-sm focus:outline-none w-full sm:w-48 text-slate-700" />
                <span class="text-xs text-slate-400 border-l border-slate-300 pl-2 hidden md:inline" title="您的 Key 僅在本地瀏覽器使用，絕不上傳伺服器">本地安全</span>
            </div>
        </div>
    </header>

    <main class="max-w-4xl mx-auto px-4 py-8">
        
        <section id="welcomeSection" class="text-center my-8 md:my-12">
            <h2 class="text-3xl md:text-4xl font-extrabold text-slate-900 mb-3 tracking-tight">
                拍照上傳，一鍵獲取您的專屬健康調理計畫
            </h2>
            <p class="text-slate-500 max-w-xl mx-auto mb-8">
                免去繁瑣的醫學名詞查詢。只需拍下您的體檢報告，AI 就會自動幫您抓出紅字異常，並給予最貼心的白話解說與日常改善方針。
            </p>

            <div class="medical-card p-8 max-w-md mx-auto border-2 border-dashed border-blue-200 hover:border-blue-400 transition-all shadow-md">
                <label class="cursor-pointer flex flex-col items-center justify-center gap-4 group">
                    <div class="w-20 h-20 bg-blue-50 rounded-full flex items-center justify-center text-blue-500 group-hover:scale-105 transition-transform">
                        <i class="fa-solid fa-camera text-3xl"></i>
                    </div>
                    <div class="text-center">
                        <span class="block text-lg font-semibold text-slate-700">📸 拍攝 / 上傳體檢報告</span>
                        <span class="text-sm text-slate-400 mt-1 block">支援手機相機直拍、PNG 或 JPG 格式</span>
                    </div>
                    <input type="file" id="imageInput" accept="image/*" capture="environment" class="hidden" onchange="handleFileSelect(event)">
                </label>
            </div>
        </section>

        <section id="loadingSection" class="hidden text-center my-12">
            <div class="medical-card p-6 max-w-sm mx-auto flex flex-col items-center gap-4">
                <div class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"></div>
                <p class="text-lg font-medium text-slate-700">👨‍⚕️ AI 醫師正在仔細閱讀您的報告...</p>
                <p class="text-sm text-slate-400">這大約需要 15-30 秒，請勿關閉視窗</p>
                <img id="previewImage" class="w-full max-h-48 object-contain rounded-lg border mt-2 shadow-sm" alt="報告預覽">
            </div>
        </section>

        <section id="resultSection" class="hidden space-y-6">
            
            <div class="flex justify-between items-center bg-white p-4 rounded-xl shadow-sm">
                <button onclick="resetApp()" class="text-sm font-medium text-blue-600 hover:text-blue-800 flex items-center gap-1">
                    <i class="fa-solid fa-arrow-left"></i> 重新上傳
                </button>
                <div class="flex gap-2">
                    <button onclick="copyResult()" class="px-3 py-1.5 bg-slate-100 hover:bg-slate-200 rounded-lg text-sm font-medium flex items-center gap-1 transition-colors">
                        <i class="fa-solid fa-copy"></i> 複製文字
                    </button>
                    <button onclick="window.print()" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-700 text-white rounded-lg text-sm font-medium flex items-center gap-1 transition-colors">
                        <i class="fa-solid fa-download"></i> 列印 / 儲存 PDF
                    </button>
                </div>
            </div>

            <div class="bg-amber-50 border-l-4 border-amber-500 p-4 rounded-r-xl shadow-sm">
                <div class="flex gap-2">
                    <span class="text-amber-600 font-bold">⚠️</span>
                    <p class="text-amber-800 text-sm font-medium">
                        本分析僅供健康管理參考，無法取代專業醫師診斷。若身體嚴重不適，請立即就醫。
                    </p>
                </div>
            </div>

            <div class="medical-card p-6 md:p-8">
                <div class="flex items-center gap-2 mb-6 border-b pb-4">
                    <span class="text-emerald-500 text-xl"><i class="fa-solid fa-circle-check"></i></span>
                    <h3 class="text-xl font-bold text-slate-900">AI 智慧健檢評估報告</h3>
                </div>
                <div id="aiOutput" class="prose prose-slate max-w-none text-slate-700 leading-relaxed"></div>
            </div>

        </section>
    </main>

    <script>
        let rawAiText = "";

        function handleFileSelect(event) {
            const file = event.target.files[0];
            if (!file) return;

            const apiKey = document.getElementById('apiKey').value.trim();
            if (!apiKey) {
                alert('請先在網頁右上方輸入您的 Gemini API Key 喔！');
                return;
            }

            const reader = new FileReader();
            reader.onload = function(e) {
                document.getElementById('previewImage').src = e.target.result;
                document.getElementById('welcomeSection').classList.add('hidden');
                document.getElementById('loadingSection').classList.remove('hidden');
                
                const base64Data = e.target.result.split(',')[1];
                const mimeType = file.type;
                
                callGeminiVisionAPI(apiKey, base64Data, mimeType);
            };
            reader.readAsDataURL(file);
        }

        async function callGeminiVisionAPI(apiKey, base64Data, mimeType) {
            // 使用 Gemini 2.5 Flash 穩定版端點
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${apiKey}`;
            
            const promptText = `
你是一位專業且溫柔的家庭醫學科醫生與健康顧問。請分析使用者上傳的體檢報告圖片，並嚴格遵守以下格式提供繁體中文的分析報告。
【重要】在報告最開頭，必須加上免責聲明：「⚠️ 本分析僅供健康管理參考，無法取代專業醫師診斷。若身體嚴重不適，請立即就醫。」

請依序輸出：
1. ⚖️ 異常指標分析：用最白話、外行人都看得懂的語言，解釋哪些數據異常（紅字），以及這些異常可能引起什麼身體症狀。
2. 🌱 改善與調理建議：提供具體的飲食（該吃、不該吃）與生活作息建議。
3. 📅 30天健康改善計畫表：請用精美的表格（Table）呈現四週的具體行動方針。
4. 🚨 危機就醫警訊：用紅色顯眼區塊說明，若上述異常指標「同時伴隨哪些特定症狀」出現時，必須立刻前往醫院就醫。
            `;

            const requestBody = {
                contents: [{
                    parts: [
                        { text: promptText },
                        {
                            inlineData: {
                                mimeType: mimeType,
                                data: base64Data
                            }
                        }
                    ]
                }]
            };

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(requestBody)
                });

                if (!response.ok) {
                    throw new Error(`API 回傳錯誤，狀態碼: ${response.status}`);
                }

                const data = await response.json();
                rawAiText = data.candidates[0].content.parts[0].text;
                
                renderResult(rawAiText);

            } catch (error) {
                console.error(error);
                alert(`分析失敗：${error.message}。請確認您的 API Key 是否輸入正確。`);
                resetApp();
            }
        }

        function renderResult(markdownText) {
            document.getElementById('loadingSection').classList.add('hidden');
            document.getElementById('resultSection').classList.remove('hidden');
            document.getElementById('aiOutput').innerHTML = marked.parse(markdownText);
        }

        function copyResult() {
            navigator.clipboard.writeText(rawAiText).then(() => {
                alert('報告文字已成功複製！');
            }).catch(err => {
                alert('複製失敗，請手動全選複製。');
            });
        }

        function resetApp() {
            document.getElementById('imageInput').value = '';
            document.getElementById('resultSection').classList.add('hidden');
            document.getElementById('loadingSection').classList.add('hidden');
            document.getElementById('welcomeSection').classList.remove('hidden');
        }
    </script>
</body>
</html>
