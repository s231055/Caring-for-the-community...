<!DOCTYPE html>
<html lang="zh-Hant">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>屯門優惠報料站 - 即時優惠分享</title>
    <!-- 載入 Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 移除單獨的 Lucide Icons 導入區塊，避免腳本執行順序問題 -->
    <style>
        /* 設定字體為 Inter，並確保中文字體能正常顯示 */
        body {
            font-family: 'Inter', 'Noto Sans TC', sans-serif;
            background-color: #f3f4f6;
        }
        /* 自定義滾動條樣式，讓它更美觀 */
        .custom-scrollbar::-webkit-scrollbar {
            width: 8px;
        }
        .custom-scrollbar::-webkit-scrollbar-thumb {
            background-color: #d1d5db;
            border-radius: 20px;
        }
        .custom-scrollbar::-webkit-scrollbar-track {
            background: #f3f4f6;
        }
        /* 隱藏原生檔案輸入的文字 */
        .file-input::-webkit-file-upload-button {
            visibility: hidden;
        }
    </style>
</head>
<body class="min-h-screen">

    <div id="app" class="max-w-xl mx-auto p-4 pt-8 md:pt-4">
        <!-- 標題與導航 -->
        <header class="mb-6 bg-white shadow-lg rounded-xl p-4 sticky top-0 z-10">
            <h1 class="text-3xl font-extrabold text-red-600 mb-2 flex items-center">
                <span class="mr-2 text-4xl">🚨</span>屯門優惠報料站
            </h1>
            <p class="text-sm text-gray-500 mb-4">即時分享屯門區內 (及模擬 1000m 內) 的急清、超市、街市等優惠。</p>

            <!-- 導航/切換按鈕 -->
            <div class="flex space-x-2 p-1 bg-gray-100 rounded-lg">
                <button onclick="window.app.setView('feed')" id="nav-feed"
                        class="flex-1 py-2 text-center rounded-md font-semibold text-white bg-red-600 transition duration-150 ease-in-out shadow-md">
                    優惠牆
                </button>
                <button onclick="window.app.setView('submit')" id="nav-submit"
                        class="flex-1 py-2 text-center rounded-md font-semibold text-gray-700 transition duration-150 ease-in-out">
                    報料平台
                </button>
            </div>
            
            <!-- 分類篩選 (只在優惠牆顯示) -->
            <div id="filters" class="mt-4 space-y-2">
                <h3 class="text-sm font-bold text-gray-600">篩選類別:</h3>
                <div id="category-filter-container" class="flex flex-wrap gap-2 text-sm">
                    <!-- 篩選按鈕將由 JS 動態生成 -->
                </div>
            </div>
        </header>

        <!-- 優惠牆 (主頁面) -->
        <div id="feed-view" class="space-y-4">
            <div id="loading-indicator" class="text-center text-gray-500 py-8 hidden">
                <svg class="animate-spin h-6 w-6 text-red-500 mx-auto" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                    <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                    <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                <p class="mt-2">正在讀取即時優惠...</p>
            </div>
            <div id="deals-list" class="space-y-4">
                <!-- 優惠卡片將由 JS 動態生成 -->
            </div>
            <div id="no-deals-message" class="text-center text-gray-500 py-12 hidden">
                <p>目前沒有符合條件的優惠。請稍後再查看或成為報料大使！</p>
                <p class="text-xs mt-2">（此 App 僅顯示「屯門區」優惠）</p>
            </div>
        </div>

        <!-- 報料平台 (提交頁面) -->
        <div id="submit-view" class="hidden bg-white p-6 rounded-xl shadow-lg">
            <h2 class="text-2xl font-bold text-gray-800 mb-4">📢 報料大使平台</h2>
            <p class="text-sm text-gray-600 mb-6">請填寫優惠詳情，幫助街坊們捕捉限時優惠！</p>
            
            <form id="deal-submission-form" class="space-y-4">
                <!-- 優惠類別 -->
                <div>
                    <label for="category" class="block text-sm font-medium text-gray-700">🚨 優惠類別 <span class="text-red-500">*</span></label>
                    <select id="category" name="category" required
                            class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-3 border focus:border-red-500 focus:ring-red-500">
                        <option value="">請選擇</option>
                        <option value="急清貨">🚨 急清貨 (賞味期限剩數小時)</option>
                        <option value="超市">🛍️ 超市優惠</option>
                        <option value="街市">🥬 街市/菜檔優惠</option>
                        <option value="食肆">🍽️ 食肆 (餐飲)</option>
                        <option value="長者專屬">👴 長者專屬優惠</option>
                        <option value="其他">🛒 其他優惠</option>
                    </select>
                </div>

                <!-- 地點/分店 (模擬 GPS 定位) -->
                <div>
                    <label for="location" class="block text-sm font-medium text-gray-700">📍 地點/分店 <span class="text-red-500">*</span></label>
                    <input type="text" id="location" name="location" required placeholder="例如：屯門市廣場惠康 (近電梯) / 友愛街市陳記菜檔"
                           class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-3 border focus:border-red-500 focus:ring-red-500">
                </div>
                
                <!-- 優惠詳情 -->
                <div>
                    <label for="details" class="block text-sm font-medium text-gray-700">💡 優惠詳情 <span class="text-red-500">*</span></label>
                    <textarea id="details" name="details" rows="3" required placeholder="例如：急凍餃子買一送一，$59.9/兩包。原價$59.9/包。"
                              class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-3 border focus:border-red-500 focus:ring-red-500"></textarea>
                </div>
                
                <!-- 有效期限 -->
                <div>
                    <label for="expiry" class="block text-sm font-medium text-gray-700">⏳ 有效期限/清貨截止時間 <span class="text-red-500">*</span></label>
                    <input type="text" id="expiry" name="expiry" required placeholder="例如：今晚 8:00pm 截止 / 食物賞味期限至 11/7"
                           class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-3 border focus:border-red-500 focus:ring-red-500">
                </div>

                <!-- 實物照片 (使用 Placeholder URL) -->
                <div>
                    <label for="photoUrl" class="block text-sm font-medium text-gray-700">📸 實物照片 (URL 或 Placeholder)</label>
                    <input type="text" id="photoUrl" name="photoUrl" placeholder="請貼上圖片網址 (可選)"
                           class="mt-1 block w-full rounded-md border-gray-300 shadow-sm p-3 border focus:border-red-500 focus:ring-red-500">
                </div>

                <!-- 提交按鈕 -->
                <button type="submit" id="submit-button"
                        class="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 px-4 rounded-xl transition duration-300 shadow-lg mt-4">
                    提交優惠情報
                </button>
            </form>
            <p id="submission-message" class="mt-4 text-center text-green-600 font-semibold hidden"></p>

            <!-- 新增：模擬數據生成按鈕 (FIX: 預設禁用，直到初始化完成) -->
            <button id="seed-mock-button" onclick="window.app.seedData()" disabled
                    class="w-full bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded-xl transition duration-300 shadow-lg mt-6 text-sm opacity-50 cursor-not-allowed">
                一鍵生成模擬優惠 (測試用)
            </button>
        </div>

        <!-- 全局通知 Toast 模擬 Push Notification -->
        <div id="toast-notification" class="fixed bottom-4 left-1/2 transform -translate-x-1/2 p-3 bg-green-500 text-white rounded-lg shadow-xl hidden transition duration-500 ease-in-out z-20">
            有新的屯門急清優惠！請點擊查看。
        </div>

        <!-- 自定義確認/錯誤訊息Modal (模擬 alert/confirm 替代) -->
        <div id="custom-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 flex items-center justify-center p-4 z-50 hidden">
            <div class="bg-white p-6 rounded-xl shadow-2xl max-w-sm w-full">
                <h3 id="modal-title" class="text-xl font-bold mb-3 text-gray-800"></h3>
                <p id="modal-message" class="text-gray-600 mb-6"></p>
                <div class="flex justify-end space-x-3">
                    <button id="modal-cancel" class="px-4 py-2 bg-gray-200 text-gray-700 rounded-lg hover:bg-gray-300 transition">取消</button>
                    <button id="modal-confirm" class="px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 transition">確定</button>
                </div>
            </div>
        </div>

    </div>

    <!-- Firebase 腳本和應用程式邏輯 -->
    <script type="module">
        // 載入 Firebase SDK 模組
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, collection, query, orderBy, onSnapshot, addDoc, updateDoc, increment, getDocs, setDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // 【修復】將 Lucide Icons 導入合併到主應用程式模組中，確保它在所有使用前都被定義
        import * as lucide from 'https://cdn.jsdelivr.net/npm/lucide@latest/dist/esm/lucide.js';
        window.lucide = lucide;

        // Firestore Log Level (方便調試)
        setLogLevel('Debug');

        // 全局 Firebase 變量
        let app, db, auth;
        let userId = 'anonymous'; // 預設為匿名

        // 從環境中獲取強制性的全局變量
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        // Firestore 路徑 (公開資料，用於多人協作/分享)
        const DEALS_COLLECTION_PATH = `artifacts/${appId}/public/data/tuen_mun_deals`;
        
        // 篩選類別列表
        const CATEGORIES = ['全部', '急清貨', '超市', '街市', '食肆', '長者專屬', '其他'];

        // 應用程式狀態管理
        const state = {
            currentView: 'feed', // 'feed' 或 'submit'
            selectedCategory: '全部',
            deals: [],
            isAuthReady: false,
            lastDealCount: 0, // 用於檢測是否有新優惠
        };

        // --- 輔助函數 ---

        /**
         * 自定義彈出式窗口 (取代 alert/confirm)
         * @param {string} title - 彈窗標題
         * @param {string} message - 彈窗內容
         * @param {boolean} showCancel - 是否顯示取消按鈕 (模擬 confirm)
         * @returns {Promise<boolean>} - 如果是確認類型，返回 Promise 確定或取消
         */
        function showCustomModal(title, message, showCancel = false) {
            return new Promise(resolve => {
                const modal = document.getElementById('custom-modal');
                document.getElementById('modal-title').textContent = title;
                document.getElementById('modal-message').textContent = message;
                const confirmBtn = document.getElementById('modal-confirm');
                const cancelBtn = document.getElementById('modal-cancel');

                cancelBtn.classList.toggle('hidden', !showCancel);

                modal.classList.remove('hidden');

                const cleanup = () => {
                    modal.classList.add('hidden');
                    confirmBtn.onclick = null;
                    cancelBtn.onclick = null;
                };

                confirmBtn.onclick = () => {
                    cleanup();
                    resolve(true);
                };

                cancelBtn.onclick = () => {
                    cleanup();
                    resolve(false);
                };
            });
        }

        /**
         * 顯示 Toast 通知 (模擬 Push Notification)
         * @param {string} message - 通知訊息
         * @param {string} color - 顏色類別
         */
        function showToast(message, color = 'bg-green-500') {
            const toast = document.getElementById('toast-notification');
            toast.textContent = message;
            toast.className = `fixed bottom-4 left-1/2 transform -translate-x-1/2 p-3 text-white rounded-lg shadow-xl transition duration-500 ease-in-out z-20 ${color}`;
            toast.classList.remove('hidden');
            setTimeout(() => {
                toast.classList.add('hidden');
            }, 5000);
        }

        /**
         * 格式化時間戳
         * @param {number} timestamp - Unix 時間戳
         * @returns {string} 格式化的日期時間
         */
        function formatTimestamp(timestamp) {
            if (!timestamp) return 'N/A';
            const date = new Date(timestamp);
            return date.toLocaleDateString('zh-TW', { year: 'numeric', month: 'numeric', day: 'numeric', hour: '2-digit', minute: '2-digit' });
        }


        // --- 介面渲染函數 ---
        
        /**
         * 設置當前活動視圖
         * @param {string} view - 'feed' 或 'submit'
         */
        window.app = {
            setView: (view) => {
                state.currentView = view;
                const feedView = document.getElementById('feed-view');
                const submitView = document.getElementById('submit-view');
                const navFeed = document.getElementById('nav-feed');
                const navSubmit = document.getElementById('nav-submit');
                const filters = document.getElementById('filters');

                if (view === 'feed') {
                    feedView.classList.remove('hidden');
                    submitView.classList.add('hidden');
                    filters.classList.remove('hidden');
                    navFeed.classList.add('bg-red-600', 'text-white', 'shadow-md');
                    navFeed.classList.remove('text-gray-700', 'bg-gray-100');
                    navSubmit.classList.remove('bg-red-600', 'text-white', 'shadow-md');
                    navSubmit.classList.add('text-gray-700', 'bg-gray-100');
                    renderDealsList(state.deals);
                } else {
                    feedView.classList.add('hidden');
                    submitView.classList.remove('hidden');
                    filters.classList.add('hidden');
                    navSubmit.classList.add('bg-red-600', 'text-white', 'shadow-md');
                    navSubmit.classList.remove('text-gray-700', 'bg-gray-100');
                    navFeed.classList.remove('bg-red-600', 'text-white', 'shadow-md');
                    navFeed.classList.add('text-gray-700', 'bg-gray-100');
                }
            },
            setSelectedCategory: (category) => {
                state.selectedCategory = category;
                renderCategoryFilters();
                renderDealsList(state.deals);
            },
            // 新增：用於生成模擬數據的公開方法
            seedData: async () => {
                await seedMockData();
            }
        };

        /**
         * 渲染分類篩選按鈕
         */
        function renderCategoryFilters() {
            const container = document.getElementById('category-filter-container');
            container.innerHTML = '';

            CATEGORIES.forEach(category => {
                const isActive = state.selectedCategory === category;
                const button = document.createElement('button');
                button.textContent = category;
                button.className = `px-3 py-1 rounded-full font-medium transition duration-150 ease-in-out ${
                    isActive ? 'bg-red-600 text-white shadow-md' : 'bg-white text-gray-700 border border-gray-300 hover:bg-red-50'
                }`;
                button.onclick = () => window.app.setSelectedCategory(category);
                container.appendChild(button);
            });
        }
        
        /**
         * 渲染優惠列表
         * @param {Array<Object>} deals - 優惠數據陣列
         */
        function renderDealsList(deals) {
            const listContainer = document.getElementById('deals-list');
            const noDealsMessage = document.getElementById('no-deals-message');
            listContainer.innerHTML = '';

            const filteredDeals = deals.filter(deal => {
                // 模擬地理圍欄/GPS 過濾 (只顯示屯門區的優惠) - 實際上所有數據都假設是屯門區的
                // 模擬分類篩選
                return (state.selectedCategory === '全部' || deal.category === state.selectedCategory) && !deal.isExpired;
            });
            
            if (filteredDeals.length === 0) {
                listContainer.classList.add('hidden');
                noDealsMessage.classList.remove('hidden');
                document.getElementById('loading-indicator').classList.add('hidden');
                return;
            }

            listContainer.classList.remove('hidden');
            noDealsMessage.classList.add('hidden');
            document.getElementById('loading-indicator').classList.add('hidden');


            // 根據時間倒序排序 (最新報料在上)
            filteredDeals.sort((a, b) => b.timestamp - a.timestamp);

            filteredDeals.forEach(deal => {
                const isUrgent = deal.category === '急清貨';
                const card = document.createElement('div');
                card.id = `deal-${deal.id}`;
                card.className = `bg-white p-5 rounded-xl shadow-lg border-l-8 ${isUrgent ? 'border-red-600' : 'border-blue-500'} transition duration-300 hover:shadow-xl`;

                const categoryTag = `<span class="text-xs font-semibold px-2 py-0.5 rounded-full ${isUrgent ? 'bg-red-100 text-red-700' : 'bg-blue-100 text-blue-700'}">${deal.category}</span>`;
                
                const photoHtml = deal.photoUrl ? 
                    `<img src="${deal.photoUrl}" alt="實物照片" class="w-full h-32 object-cover rounded-lg my-3 shadow-inner" onerror="this.onerror=null; this.src='https://placehold.co/400x150/fca5a5/ffffff?text=無圖或載入失敗';">` :
                    `<div class="h-20 flex items-center justify-center text-gray-400 bg-gray-50 rounded-lg my-3 border border-dashed">無實物照片</div>`;

                const likeIcon = deal.userLiked ? 'heart' : 'heart'; 
                const likeColor = deal.userLiked ? 'text-red-500' : 'text-gray-400';

                card.innerHTML = `
                    <div class="flex justify-between items-start mb-2">
                        <h3 class="text-xl font-bold text-gray-800">${isUrgent ? '🚨 ' : ''}${deal.location}</h3>
                        ${categoryTag}
                    </div>
                    
                    <p class="text-sm text-gray-500 mb-1 flex items-center">
                        <i data-lucide="map-pin" class="w-4 h-4 mr-1"></i>
                        屯門區 ${deal.location}
                    </p>
                    
                    ${photoHtml}

                    <p class="text-gray-700 font-medium mb-3 whitespace-pre-wrap">💡 ${deal.details}</p>

                    <p class="text-sm font-semibold mb-3 ${isUrgent ? 'text-red-600' : 'text-green-600'} flex items-center">
                        <i data-lucide="clock" class="w-4 h-4 mr-1"></i>
                        ⏳ 有效期限：${deal.expiry}
                    </p>
                    
                    <hr class="my-3 border-gray-100">

                    <div class="flex justify-between items-center text-sm text-gray-400">
                        <div class="flex items-center space-x-4">
                            <!-- 優惠核實/評分 -->
                            <button id="like-btn-${deal.id}" class="flex items-center space-x-1 transition duration-150 hover:text-red-500">
                                <i data-lucide="${deal.userLiked ? 'heart' : 'heart'}" class="w-5 h-5 ${likeColor} fill-current"></i>
                                <span class="font-bold text-gray-600">${deal.likes || 0}</span>
                                <span class="text-gray-500">核實</span>
                            </button>
                            
                            <!-- 優惠已過回報 -->
                            <button id="report-btn-${deal.id}" class="flex items-center space-x-1 text-gray-400 transition duration-150 hover:text-red-700">
                                <i data-lucide="flag" class="w-5 h-5"></i>
                                <span>回報已過期</span>
                            </button>
                        </div>
                        <p>報料時間: ${formatTimestamp(deal.timestamp)}</p>
                    </div>
                `;

                listContainer.appendChild(card);
                
                // 初始化 Lucide Icons
                window.lucide.createIcons({ icons: window.lucide.icons });

                // 綁定事件監聽器
                document.getElementById(`like-btn-${deal.id}`).onclick = () => handleLikeDeal(deal.id, deal.userLiked);
                document.getElementById(`report-btn-${deal.id}`).onclick = () => handleReportExpired(deal.id);
            });
        }

        // --- Firebase 互動函數 ---

        /**
         * 初始化 Firebase 應用程式和認證
         */
        async function initFirebase() {
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    throw new Error("Firebase Config is missing. Cannot initialize Firebase.");
                }
                
                app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // 使用 custom token 登入，如果沒有則匿名登入
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }
                
                userId = auth.currentUser?.uid || crypto.randomUUID();
                state.isAuthReady = true;

                // FIX: 認證完成後，解除禁用模擬數據生成按鈕
                const seedButton = document.getElementById('seed-mock-button');
                if (seedButton) {
                    seedButton.disabled = false;
                    seedButton.classList.remove('opacity-50', 'cursor-not-allowed');
                }
                
                console.log(`Firebase 初始化成功。User ID: ${userId}`);

                // 啟動即時監聽
                setupRealtimeListener();

            } catch (error) {
                console.error("Firebase 初始化失敗:", error);
                document.getElementById('loading-indicator').innerHTML = `<p class="text-red-500">載入錯誤: ${error.message}</p>`;
                document.getElementById('loading-indicator').classList.remove('hidden');
                state.isAuthReady = true; // 即使失敗也設為 true 避免無限 loading
            }
        }

        /**
         * 設置 Firestore 上的即時數據監聽 (onSnapshot)
         */
        function setupRealtimeListener() {
            if (!db) return;

            const q = query(collection(db, DEALS_COLLECTION_PATH));
            const loadingIndicator = document.getElementById('loading-indicator');
            loadingIndicator.classList.remove('hidden');

            onSnapshot(q, (snapshot) => {
                const newDeals = [];
                let hasNewUrgentDeal = false;

                snapshot.forEach(doc => {
                    const data = doc.data();
                    // 檢查用戶是否已點讚 (模擬)
                    // 注意：Firestore 的 updateDoc increment 不會自動創建 likes 欄位，但我們在 seedData 或 addDoc 時會確保它存在。
                    const userLiked = data.likedBy && data.likedBy[userId];
                    newDeals.push({ id: doc.id, ...data, userLiked });
                });
                
                // 檢查是否有新的急清貨優惠
                if (newDeals.length > state.lastDealCount) {
                    // 找出最新的交易，並檢查是否為急清貨
                    const latestDeal = newDeals.sort((a, b) => b.timestamp - a.timestamp)[0];
                    if (latestDeal && latestDeal.category === '急清貨' && newDeals.length > state.lastDealCount) {
                        hasNewUrgentDeal = true;
                    }
                }
                
                state.deals = newDeals;
                renderDealsList(state.deals);
                
                // 模擬 Push Notification
                if (hasNewUrgentDeal) {
                    showToast('🚨 緊急通知！有新的屯門急清貨優惠！', 'bg-red-600');
                }
                
                state.lastDealCount = newDeals.length;
            }, (error) => {
                console.error("即時監聽數據失敗:", error);
                showToast(`數據載入錯誤: ${error.message}`, 'bg-red-500');
            });
        }

        /**
         * 處理優惠點讚/核實
         * @param {string} dealId - 優惠文檔 ID
         * @param {boolean} currentlyLiked - 用戶當前是否已點讚
         */
        async function handleLikeDeal(dealId, currentlyLiked) {
            if (!state.isAuthReady) {
                showToast('請等待認證完成後再操作', 'bg-yellow-500');
                return;
            }

            const dealRef = doc(db, DEALS_COLLECTION_PATH, dealId);
            const batchUpdate = {};

            if (currentlyLiked) {
                // 取消點讚
                batchUpdate.likes = increment(-1);
                batchUpdate[`likedBy.${userId}`] = null; // 移除用戶ID
            } else {
                // 點讚
                batchUpdate.likes = increment(1);
                batchUpdate[`likedBy.${userId}`] = true; // 標記用戶ID
            }

            try {
                await updateDoc(dealRef, batchUpdate);
                // onSnapshot 會自動更新介面
            } catch (error) {
                console.error("更新點讚失敗:", error);
                showToast('點讚操作失敗，請稍後再試', 'bg-red-500');
            }
        }

        /**
         * 處理回報優惠已過期
         * @param {string} dealId - 優惠文檔 ID
         */
        async function handleReportExpired(dealId) {
            const confirmed = await showCustomModal(
                '回報優惠已過期',
                '您確定要回報此優惠已過期嗎？回報後，它將被標記為無效。',
                true
            );

            if (confirmed) {
                try {
                    const dealRef = doc(db, DEALS_COLLECTION_PATH, dealId);
                    await updateDoc(dealRef, {
                        isExpired: true,
                        reportedBy: userId,
                        reportedAt: serverTimestamp()
                    });
                    showToast('成功回報！優惠已從列表中移除。', 'bg-green-500');
                } catch (error) {
                    console.error("回報過期失敗:", error);
                    showToast('回報操作失敗，請稍後再試', 'bg-red-500');
                }
            }
        }

        // --- 表單處理函數 ---

        /**
         * 處理優惠報料表單提交
         * @param {Event} event - 提交事件
         */
        async function handleSubmission(event) {
            event.preventDefault();
            
            if (!state.isAuthReady) {
                showToast('請等待認證完成後再操作', 'bg-yellow-500');
                return;
            }

            const form = event.target;
            const submitButton = document.getElementById('submit-button');
            const messageElement = document.getElementById('submission-message');

            submitButton.disabled = true;
            submitButton.textContent = '正在提交...';
            messageElement.classList.add('hidden');

            const newDeal = {
                category: form.category.value,
                location: form.location.value,
                details: form.details.value,
                expiry: form.expiry.value,
                photoUrl: form.photoUrl.value || '', // 允許為空
                likes: 0,
                isExpired: false,
                reporterId: userId,
                timestamp: Date.now() // 使用前端時間戳
            };
            
            // 由於所有優惠都預設在屯門區，不需要額外的經緯度欄位來篩選。
            // 但我們可以加入一個標籤，模擬地理位置標記。
            newDeal.geoTag = 'TuenMun';
            
            try {
                // 將數據添加到 Firestore
                await addDoc(collection(db, DEALS_COLLECTION_PATH), newDeal);

                submitButton.textContent = '提交成功！';
                messageElement.textContent = '感謝您的報料！優惠已即時發佈到優惠牆！';
                messageElement.classList.remove('hidden');
                form.reset(); // 清空表單
                
                // 成功後延遲切換視圖
                setTimeout(() => {
                    window.app.setView('feed');
                }, 2000);

            } catch (error) {
                console.error("提交優惠失敗:", error);
                messageElement.textContent = `提交失敗: ${error.message}`;
                messageElement.classList.remove('hidden');
                messageElement.classList.remove('text-green-600');
                messageElement.classList.add('text-red-600');
                submitButton.textContent = '提交優惠情報';
            } finally {
                submitButton.disabled = false;
            }
        }

        /**
         * 新增：生成模擬數據並寫入 Firestore
         * 【已優化】新增載入狀態和日誌記錄，提供即時視覺回饋。
         */
        async function seedMockData() {
            const seedButton = document.getElementById('seed-mock-button');
            
            // 檢查初始化狀態
            if (!state.isAuthReady || !db) {
                showToast('應用程式初始化尚未完成。', 'bg-yellow-500');
                console.error("SEED DATA ERROR: App not ready (Auth or DB missing)");
                return;
            }

            // 設置按鈕為載入狀態
            const originalText = seedButton.textContent;
            seedButton.disabled = true;
            seedButton.textContent = '正在生成中... (請稍候)';
            seedButton.classList.remove('bg-gray-500');
            seedButton.classList.add('bg-gray-700');
            
            console.log(`SEED DATA: Starting mock data generation for User ID: ${userId}`);


            // 模擬優惠數據
            const mockDeals = [
                {
                    category: '急清貨',
                    location: '屯門市廣場-百佳',
                    details: '三文魚刺身半價！原價 $50/盒，現售 $25/盒！只限 2 盒。',
                    expiry: '今晚 8:30pm 截止',
                    photoUrl: 'https://placehold.co/400x150/ef4444/ffffff?text=急清三文魚',
                    likes: 15,
                },
                {
                    category: '超市',
                    location: '新墟街市-惠康',
                    details: '所有國產零食買二送一，可混合搭配！',
                    expiry: '本週末 (週六日) 全天',
                    photoUrl: 'https://placehold.co/400x150/3b82f6/ffffff?text=零食大優惠',
                    likes: 8,
                },
                {
                    category: '街市',
                    location: '友愛街市-陳記菜檔',
                    details: '本地生菜 $5/斤，大白菜 $10/個，比平時便宜 30%！',
                    expiry: '賣完即止',
                    photoUrl: '', // 沒照片
                    likes: 22,
                },
                {
                    category: '食肆',
                    location: 'V city - 某茶餐廳',
                    details: '學生憑證，所有飲品免費加大！',
                    expiry: '長期有效 (週一至五)',
                    photoUrl: 'https://placehold.co/400x150/10b981/ffffff?text=學生優惠',
                    likes: 10,
                },
                {
                    category: '長者專屬',
                    location: '屯門大會堂',
                    details: '65歲或以上長者，憑證件免費觀看本地文化展覽。',
                    expiry: '即日至本月月底',
                    photoUrl: 'https://placehold.co/400x150/8b5cf6/ffffff?text=長者優惠', 
                    likes: 5,
                },
                {
                    category: '其他',
                    location: '屯門公園',
                    details: '最新荷蘭鬱金香品種展覽，免費參觀！',
                    expiry: '限時一週',
                    photoUrl: 'https://placehold.co/400x150/06b6d4/ffffff?text=公園活動', 
                    likes: 18,
                }
            ];

            try {
                const results = await Promise.all(mockDeals.map((deal, index) => {
                    // 使用不同的時間戳確保排序正確
                    const timestampOffset = Date.now() - (mockDeals.length - 1 - index) * 60000;
                    const newDeal = {
                        ...deal,
                        isExpired: false,
                        reporterId: userId,
                        timestamp: timestampOffset,
                        geoTag: 'TuenMun',
                        likes: deal.likes || 0,
                        likedBy: {}
                    };
                    return addDoc(collection(db, DEALS_COLLECTION_PATH), newDeal);
                }));

                console.log(`SEED DATA: Successfully added ${results.length} deals to collection path: ${DEALS_COLLECTION_PATH}`);
                showToast(`成功添加 ${results.length} 條模擬優惠！請查看「優惠牆」。`, 'bg-green-500');
            } catch (error) {
                console.error("SEED DATA ERROR: 生成模擬數據失敗，詳細錯誤如下:", error);
                // 讓錯誤狀態持續 3 秒，確保用戶看到
                showToast('🚫 生成模擬優惠失敗，請檢查控制台 (F12) 獲取詳情。', 'bg-red-500');
                await new Promise(r => setTimeout(r, 3000)); 
            } finally {
                // 恢復按鈕狀態
                seedButton.textContent = originalText;
                seedButton.disabled = false;
                seedButton.classList.remove('bg-gray-700');
                seedButton.classList.add('bg-gray-500');
            }
        }


        // --- 初始化和事件綁定 ---
        
        window.onload = () => {
            initFirebase();
            renderCategoryFilters();
            window.app.setView('feed'); // 初始顯示優惠牆

            // 綁定表單提交事件
            document.getElementById('deal-submission-form').addEventListener('submit', handleSubmission);
            
            // 替換 Lucide Icon 的顯示
            window.lucide.createIcons({ icons: window.lucide.icons });
        };

    </script>
</body>
</html>
