<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 內容生成器</title>
    <!-- 載入 Tailwind CSS 確保響應式設計 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 設置字體為 Inter -->
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f4f7f9;
        }
    </style>
</head>
<body class="p-4 sm:p-8 min-h-screen flex items-center justify-center">

    <div class="w-full max-w-3xl bg-white shadow-xl rounded-xl p-6 md:p-10 border border-gray-100">
        <h1 class="text-3xl font-extrabold text-gray-900 mb-6 text-center">
            一鍵內容生成器
        </h1>
        <p class="text-gray-500 mb-8 text-center">點擊按鈕，使用 Gemini API 獲取一篇關於「未來科技趨勢」的簡短文章。</p>

        <!-- 輸入區域 (簡單範例，無輸入框) -->
        <div class="mb-8">
            <button id="generateButton" 
                    class="w-full bg-blue-600 text-white font-bold py-3 px-6 rounded-lg 
                           hover:bg-blue-700 transition duration-300 ease-in-out 
                           shadow-md hover:shadow-lg focus:outline-none focus:ring-4 focus:ring-blue-500/50 
                           disabled:bg-blue-400 disabled:cursor-not-allowed flex items-center justify-center"
                    aria-label="點擊生成內容">
                <svg id="loadingSpinner" class="animate-spin -ml-1 mr-3 h-5 w-5 text-white hidden" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                <span id="buttonText">一鍵生成內容</span>
            </button>
        </div>

        <!-- 結果顯示區域 -->
        <div class="bg-gray-50 border border-gray-200 rounded-lg p-6 min-h-[150px]">
            <h2 class="text-xl font-semibold text-gray-800 mb-4 border-b pb-2">生成結果:</h2>
            <div id="resultContent" class="text-gray-700 whitespace-pre-wrap leading-relaxed">
                點擊上方的按鈕開始生成...
            </div>
            <div id="citationSources" class="mt-4 text-sm text-gray-500 hidden">
                <strong class="text-gray-600">引用來源:</strong>
                <ul id="sourceList" class="list-disc list-inside ml-2 mt-1"></ul>
            </div>
        </div>

        <!-- 錯誤訊息提示框 (自定義警示) -->
        <div id="errorMessage" class="mt-4 p-3 bg-red-100 border border-red-400 text-red-700 rounded-lg hidden" role="alert">
            <strong class="font-bold">錯誤:</strong>
            <span id="errorText" class="block sm:inline ml-2">發生了一個意料之外的錯誤。</span>
        </div>
    </div>

    <!-- JavaScript 邏輯 (包含 Firebase 與 Gemini API 呼叫) -->
    <script type="module">
        // 導入 Firebase 模組
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // 設定 Firebase 偵錯等級 (建議開啟)
        setLogLevel('debug');

        // ==== Firebase 環境變數 (強制要求) ====
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // ==== 應用程式狀態變數 ====
        let db;
        let auth;
        let userId = null;
        let isAuthReady = false;

        // ==== UI 元素 ====
        const generateButton = document.getElementById('generateButton');
        const buttonText = document.getElementById('buttonText');
        const loadingSpinner = document.getElementById('loadingSpinner');
        const resultContent = document.getElementById('resultContent');
        const errorMessageDiv = document.getElementById('errorMessage');
        const errorText = document.getElementById('errorText');
        const citationSourcesDiv = document.getElementById('citationSources');
        const sourceList = document.getElementById('sourceList');

        // 顯示錯誤訊息
        function displayError(message) {
            errorText.textContent = message;
            errorMessageDiv.classList.remove('hidden');
            resultContent.textContent = '生成失敗。請查看上方的錯誤訊息。';
        }

        // 隱藏錯誤訊息
        function hideError() {
            errorMessageDiv.classList.add('hidden');
        }

        // ==== 1. Firebase 初始化與認證 ====
        async function initializeFirebase() {
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    throw new Error("Firebase 設定遺失。");
                }
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                console.log("Firebase 初始化成功。");

                // 認證監聽器
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        isAuthReady = true;
                        console.log("認證成功，使用者 ID:", userId);
                    } else {
                        // 如果沒有認證 token，則匿名登入
                        signInAnonymously(auth).then(userCredential => {
                             userId = userCredential.user.uid;
                             isAuthReady = true;
                             console.log("匿名登入成功，使用者 ID:", userId);
                        }).catch(err => {
                             console.error("匿名登入失敗:", err);
                             isAuthReady = false;
                             displayError("Firebase 認證失敗，請檢查網路連接。");
                        });
                    }
                });

                // 使用 custom token 登入 (Canvas 環境專用)
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    // 如果沒有 token，onAuthStateChanged 會處理匿名登入
                }

            } catch (error) {
                console.error("Firebase 初始化或認證發生錯誤:", error);
                displayError("Firebase 啟動失敗：" + error.message);
                isAuthReady = false;
            }
        }

        // ==== 2. Gemini API 內容生成函式 (包含指數退避重試) ====
        async function fetchWithRetry(url, options, maxRetries = 5) {
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(url, options);
                    if (response.status === 429 && i < maxRetries - 1) {
                        // 處理 429 (請求過多) 錯誤，等待並重試
                        const delay = Math.pow(2, i) * 1000 + Math.random() * 1000; // 指數退避
                        console.warn(`API 請求過多 (429)。第 ${i + 1} 次重試，等待 ${Math.round(delay)}ms...`);
                        await new Promise(resolve => setTimeout(resolve, delay));
                        continue;
                    }
                    if (!response.ok) {
                        const errorBody = await response.text();
                        throw new Error(`API 錯誤: ${response.status} ${response.statusText} - ${errorBody}`);
                    }
                    return response;
                } catch (error) {
                    if (i === maxRetries - 1) throw error; // 最後一次嘗試失敗，拋出錯誤
                    console.error(`網路請求失敗 (非 429 錯誤)。第 ${i + 1} 次重試...`, error);
                }
            }
        }

        async function generateContent(userPrompt) {
            const systemPrompt = "你是一位科技評論家。請用中文(繁體)提供一篇關於未來科技趨勢的簡潔、單一的文章。請確保內容專業且吸引人。";
            const apiKey = ""; // Canvas runtime 會自動提供金鑰
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`;

            const payload = {
                contents: [{ parts: [{ text: userPrompt }] }],
                // 啟用 Google Search grounding 獲取最新資訊
                tools: [{ "google_search": {} }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                },
            };

            const response = await fetchWithRetry(apiUrl, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            const result = await response.json();
            const candidate = result.candidates?.[0];

            if (candidate && candidate.content?.parts?.[0]?.text) {
                const text = candidate.content.parts[0].text;
                let sources = [];
                const groundingMetadata = candidate.groundingMetadata;

                if (groundingMetadata && groundingMetadata.groundingAttributions) {
                    sources = groundingMetadata.groundingAttributions
                        .map(attribution => ({
                            uri: attribution.web?.uri,
                            title: attribution.web?.title,
                        }))
                        .filter(source => source.uri && source.title);
                }
                
                return { text, sources };

            } else {
                const errorMessage = result.error?.message || "無法從 API 獲取有效的內容。可能是 API 請求或響應格式錯誤。";
                throw new Error(errorMessage);
            }
        }

        // ==== 3. 按鈕點擊處理函式 ====
        async function handleButtonClick() {
            // 檢查認證狀態是否就緒
            if (!isAuthReady) {
                displayError("應用程式仍在等待 Firebase 認證完成。請稍候再試。");
                return;
            }

            // UI 狀態：開始載入
            generateButton.disabled = true;
            buttonText.textContent = '生成中...';
            loadingSpinner.classList.remove('hidden');
            resultContent.textContent = '正在呼叫 AI 服務，請稍候...';
            hideError();
            citationSourcesDiv.classList.add('hidden');
            sourceList.innerHTML = '';

            const prompt = "未來五年內最具影響力的三大科技趨勢是什麼？";

            try {
                const { text, sources } = await generateContent(prompt);
                
                // UI 狀態：顯示結果
                resultContent.textContent = text;
                
                // 顯示引用來源
                if (sources.length > 0) {
                    sources.forEach(source => {
                        const li = document.createElement('li');
                        li.innerHTML = `<a href="${source.uri}" target="_blank" class="text-blue-600 hover:underline" rel="noopener noreferrer">${source.title}</a>`;
                        sourceList.appendChild(li);
                    });
                    citationSourcesDiv.classList.remove('hidden');
                }

            } catch (error) {
                // UI 狀態：顯示錯誤
                console.error("生成內容時發生錯誤:", error);
                displayError(`內容生成失敗: ${error.message}`);

            } finally {
                // UI 狀態：停止載入
                generateButton.disabled = false;
                buttonText.textContent = '一鍵生成內容';
                loadingSpinner.classList.add('hidden');
            }
        }

        // ==== 4. 事件監聽器 (確保按鈕被點擊時觸發函式) ====
        document.addEventListener('DOMContentLoaded', () => {
            initializeFirebase(); // 初始化 Firebase
            
            // 此行是確保按鈕響應的關鍵
            generateButton.addEventListener('click', handleButtonClick); 
            console.log("事件監聽器已附加到生成按鈕。");
        });

    </script>
</body>
</html>
