<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>拼一下 Carpool++ | 大學生專屬極簡拼車媒合</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=Noto+Sans+TC:wght@300;400;500;700&display=swap');
        
        body {
            font-family: 'Inter', 'Noto Sans TC', sans-serif;
            background-color: #FAFAFA;
            -webkit-tap-highlight-color: transparent;
        }

        /* 優雅點擊縮放效果 */
        .btn-click {
            transition: transform 0.1s ease, background-color 0.2s ease, box-shadow 0.2s ease;
        }
        .btn-click:active {
            transform: scale(0.96);
        }

        /* 隱藏滾動條但保持滾動 */
        .no-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .no-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }

        /* 載入動畫 */
        .loader {
            border-top-color: #000000;
            animation: spinner 1s linear infinite;
        }
        @keyframes spinner {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="text-neutral-900 min-h-screen flex flex-col antialiased">

    <!-- 全域通知訊息 -->
    <div id="toast" class="fixed top-5 left-1/2 -translate-x-1/2 z-50 bg-neutral-900 text-white px-5 py-3 rounded-full text-sm font-medium shadow-lg transition-all duration-300 transform translate-y-[-100px] opacity-0 flex items-center gap-2">
        <i data-lucide="info" class="w-4 h-4"></i>
        <span id="toast-text">提示訊息</span>
    </div>

    <!-- 頂部導航列 -->
    <header class="sticky top-0 bg-white/90 backdrop-blur-md border-b border-neutral-100 z-40 px-4 py-3.5 flex justify-between items-center">
        <div class="flex items-center gap-2 cursor-pointer" onclick="switchTab('home')">
            <div class="bg-black text-white p-1.5 rounded-xl">
                <i data-lucide="car" class="w-5 h-5"></i>
            </div>
            <span class="text-xl font-bold tracking-tight">拼一下 <span class="text-xs font-semibold text-neutral-400">Carpool++</span></span>
        </div>
        <div class="flex items-center gap-2">
            <!-- 登入狀態/會員中心 -->
            <div id="user-info" class="hidden flex items-center gap-2">
                <span id="username-display" class="text-xs font-medium text-neutral-500 bg-neutral-100 px-2.5 py-1.5 rounded-full max-w-[100px] truncate">...</span>
                <button onclick="logout()" class="p-2 text-neutral-400 hover:text-black btn-click">
                    <i data-lucide="log-out" class="w-4 h-4"></i>
                </button>
            </div>
            <button id="login-trigger" onclick="openLoginModal()" class="text-xs bg-black text-white px-3.5 py-1.5 rounded-full font-medium btn-click">
                快速登入
            </button>
            <button onclick="switchTab('admin')" class="p-2 text-neutral-400 hover:text-black btn-click" title="後台管理">
                <i data-lucide="settings" class="w-4 h-4"></i>
            </button>
        </div>
    </header>

    <!-- 主要內容區 -->
    <main class="flex-grow max-w-md w-full mx-auto px-4 py-6 flex flex-col justify-start">

        <!-- ================= 首頁 ================= -->
        <section id="tab-home" class="tab-content flex flex-col justify-center min-h-[70vh]">
            <div class="text-center mb-10">
                <h1 class="text-3xl font-extrabold tracking-tight mb-2">大學生的省錢拼車神隊友</h1>
                <p class="text-neutral-500 text-sm">極簡、快速、安全的校園拼車媒合平台</p>
            </div>

            <!-- 三大功能主按鈕 -->
            <div class="space-y-4">
                <button onclick="switchTab('find-ride')" class="w-full bg-black text-white py-5 px-6 rounded-2xl flex items-center justify-between shadow-sm hover:shadow-md btn-click">
                    <div class="flex items-center gap-4 text-left">
                        <div class="bg-neutral-800 p-3 rounded-xl">
                            <i data-lucide="search" class="w-6 h-6 text-white"></i>
                        </div>
                        <div>
                            <p class="font-bold text-lg">我要找拼車</p>
                            <p class="text-xs text-neutral-400">目前沒有座位？發布你的需求</p>
                        </div>
                    </div>
                    <i data-lucide="chevron-right" class="w-5 h-5 text-neutral-400"></i>
                </button>

                <button onclick="switchTab('offer-ride')" class="w-full bg-white text-black border border-neutral-200 py-5 px-6 rounded-2xl flex items-center justify-between shadow-sm hover:shadow-md btn-click">
                    <div class="flex items-center gap-4 text-left">
                        <div class="bg-neutral-100 p-3 rounded-xl">
                            <i data-lucide="user-plus" class="w-6 h-6 text-black"></i>
                        </div>
                        <div>
                            <p class="font-bold text-lg">我要提供座位</p>
                            <p class="text-xs text-neutral-500">你是司機？發布剩餘座位分攤油資</p>
                        </div>
                    </div>
                    <i data-lucide="chevron-right" class="w-5 h-5 text-neutral-400"></i>
                </button>

                <button onclick="switchTab('view-rides')" class="w-full bg-white text-black border border-neutral-200 py-5 px-6 rounded-2xl flex items-center justify-between shadow-sm hover:shadow-md btn-click">
                    <div class="flex items-center gap-4 text-left">
                        <div class="bg-neutral-100 p-3 rounded-xl">
                            <i data-lucide="list" class="w-6 h-6 text-black"></i>
                        </div>
                        <div>
                            <p class="font-bold text-lg">查看目前需求</p>
                            <p class="text-xs text-neutral-500">探索目前已有的拼車需求與座位</p>
                        </div>
                    </div>
                    <span id="active-count-badge" class="bg-red-500 text-white text-xs px-2.5 py-1 rounded-full font-bold">...</span>
                </button>
            </div>
            
            <div class="mt-12 p-4 bg-neutral-100 rounded-2xl text-center">
                <p class="text-xs text-neutral-500 font-medium">✨ 新功能上線：加入行程後可直接進行即時聊天協調！</p>
            </div>
        </section>

        <!-- ================= 我要找拼車表單 ================= -->
        <section id="tab-find-ride" class="tab-content hidden space-y-6">
            <div>
                <button onclick="switchTab('home')" class="mb-4 text-neutral-500 flex items-center gap-1 text-sm font-medium btn-click">
                    <i data-lucide="arrow-left" class="w-4 h-4"></i> 返回首頁
                </button>
                <h2 class="text-2xl font-bold tracking-tight">我要找拼車</h2>
                <p class="text-neutral-500 text-sm">輸入目的地，讓有開車的人找上你！</p>
            </div>

            <form id="find-ride-form" class="space-y-4" onsubmit="submitRideRequest(event)">
                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">出發地</label>
                    <input type="text" id="find-from" required placeholder="例如：台北科技大學 / 捷運公館站" 
                        class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                </div>

                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">目的地</label>
                    <input type="text" id="find-to" required placeholder="例如：新竹科學園區 / 台中火車站" 
                        class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                </div>

                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">出發時間</label>
                        <input type="datetime-local" id="find-time" required 
                            class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                    </div>
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">乘客總人數</label>
                        <select id="find-passengers" class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                            <option value="1">1 人</option>
                            <option value="2">2 人</option>
                            <option value="3">3 人</option>
                            <option value="4">4 人以上</option>
                        </select>
                    </div>
                </div>

                <button type="submit" class="w-full bg-black text-white py-4 rounded-xl font-bold btn-click shadow-sm mt-4">
                    送出需求
                </button>
            </form>
        </section>

        <!-- ================= 我要提供座位表單 ================= -->
        <section id="tab-offer-ride" class="tab-content hidden space-y-6">
            <div>
                <button onclick="switchTab('home')" class="mb-4 text-neutral-500 flex items-center gap-1 text-sm font-medium btn-click">
                    <i data-lucide="arrow-left" class="w-4 h-4"></i> 返回首頁
                </button>
                <h2 class="text-2xl font-bold tracking-tight">我要提供座位</h2>
                <p class="text-neutral-500 text-sm">提供空位讓同學拼車，一起分攤油資！</p>
            </div>

            <form id="offer-ride-form" class="space-y-4" onsubmit="submitRideOffer(event)">
                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">出發地</label>
                    <input type="text" id="offer-from" required placeholder="例如：新竹清華大學門口" 
                        class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                </div>

                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">目的地</label>
                    <input type="text" id="offer-to" required placeholder="例如：台北轉運站" 
                        class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                </div>

                <div class="grid grid-cols-2 gap-4">
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">出發時間</label>
                        <input type="datetime-local" id="offer-time" required 
                            class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                    </div>
                    <div>
                        <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">可提供空位數</label>
                        <select id="offer-seats" class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                            <option value="1">1 個位置</option>
                            <option value="2">2 個位置</option>
                            <option value="3">3 個位置</option>
                            <option value="4">4 個位置</option>
                        </select>
                    </div>
                </div>

                <button type="submit" class="w-full bg-black text-white py-4 rounded-xl font-bold btn-click shadow-sm mt-4">
                    發布座位
                </button>
            </form>
        </section>

        <!-- ================= 查看目前需求列表 ================= -->
        <section id="tab-view-rides" class="tab-content hidden space-y-6">
            <div class="flex justify-between items-end">
                <div>
                    <button onclick="switchTab('home')" class="mb-2 text-neutral-500 flex items-center gap-1 text-sm font-medium btn-click">
                        <i data-lucide="arrow-left" class="w-4 h-4"></i> 返回首頁
                    </button>
                    <h2 class="text-2xl font-bold tracking-tight">所有拼車項目</h2>
                    <p class="text-neutral-500 text-sm">看看有誰的目的地跟你一樣！</p>
                </div>
                <!-- 篩選器 -->
                <div class="flex gap-1.5">
                    <button onclick="filterRides('all')" id="filter-all" class="text-xs px-3 py-1.5 rounded-full font-semibold bg-black text-white transition-all btn-click">全部</button>
                    <button onclick="filterRides('passenger')" id="filter-passenger" class="text-xs px-3 py-1.5 rounded-full font-semibold bg-neutral-100 text-neutral-500 transition-all btn-click">找車中</option>
                    <button onclick="filterRides('driver')" id="filter-driver" class="text-xs px-3 py-1.5 rounded-full font-semibold bg-neutral-100 text-neutral-500 transition-all btn-click">有座位</option>
                </div>
            </div>

            <!-- 清單容器 -->
            <div id="rides-list" class="space-y-4">
                <!-- 動態注入卡片 -->
                <div class="text-center py-12 text-neutral-400">
                    <div class="loader w-8 h-8 rounded-full border-2 border-neutral-200 mx-auto mb-4"></div>
                    載入拼車資料中...
                </div>
            </div>
        </section>

        <!-- ================= 聊天室頁面 ================= -->
        <section id="tab-chat" class="tab-content hidden flex flex-col h-[75vh]">
            <div class="border-b border-neutral-100 pb-3 flex justify-between items-center">
                <div class="flex items-center gap-2">
                    <button onclick="switchTab('view-rides')" class="text-neutral-500 btn-click">
                        <i data-lucide="arrow-left" class="w-6 h-6"></i>
                    </button>
                    <div>
                        <h3 class="font-bold text-sm" id="chat-title-route">載入中...</h3>
                        <p class="text-[10px] text-neutral-400" id="chat-subtitle">即時討論對談</p>
                    </div>
                </div>
                <div class="bg-black/5 px-2.5 py-1 rounded-full text-xs font-semibold text-black" id="chat-badge-type">
                    --
                </div>
            </div>

            <!-- 聊天訊息串 -->
            <div id="chat-messages" class="flex-grow overflow-y-auto py-4 space-y-3.5 no-scrollbar flex flex-col">
                <!-- 訊息動態加載 -->
            </div>

            <!-- 發送欄 -->
            <form id="chat-form" onsubmit="sendChatMessage(event)" class="mt-2 flex gap-2">
                <input type="text" id="chat-input" required placeholder="輸入訊息..." 
                    class="flex-grow bg-neutral-100 border-0 rounded-xl px-4 py-3 focus:ring-2 focus:ring-black text-sm transition-all">
                <button type="submit" class="bg-black text-white p-3 rounded-xl btn-click">
                    <i data-lucide="send" class="w-5 h-5"></i>
                </button>
            </form>
        </section>

        <!-- ================= 後台管理頁面 ================= -->
        <section id="tab-admin" class="tab-content hidden space-y-6">
            <div>
                <button onclick="switchTab('home')" class="mb-4 text-neutral-500 flex items-center gap-1 text-sm font-medium btn-click">
                    <i data-lucide="arrow-left" class="w-4 h-4"></i> 返回首頁
                </button>
                <h2 class="text-2xl font-bold tracking-tight">後台管理中心</h2>
                <p class="text-neutral-500 text-sm">管理與清理平台的拼車需求與座位</p>
            </div>

            <div class="bg-neutral-100 p-4 rounded-xl space-y-2">
                <h3 class="font-bold text-sm text-neutral-700">系統數據</h3>
                <div class="grid grid-cols-2 gap-2 text-center">
                    <div class="bg-white p-3 rounded-lg">
                        <p class="text-xs text-neutral-400">總行程數</p>
                        <p id="stat-total" class="text-xl font-bold">0</p>
                    </div>
                    <div class="bg-white p-3 rounded-lg">
                        <p class="text-xs text-neutral-400">登入用戶</p>
                        <p id="stat-users" class="text-xl font-bold">1</p>
                    </div>
                </div>
            </div>

            <div class="space-y-3">
                <h3 class="font-bold text-sm text-neutral-700">行程項目管理</h3>
                <div id="admin-rides-list" class="space-y-2 max-h-[40vh] overflow-y-auto no-scrollbar">
                    <!-- 動態載入 -->
                </div>
            </div>
        </section>

    </main>

    <!-- 底部導航 & 狀態列 -->
    <footer class="bg-white border-t border-neutral-100 px-6 py-4 mt-auto">
        <div class="max-w-md mx-auto flex justify-around text-neutral-400">
            <button onclick="switchTab('home')" id="nav-home" class="flex flex-col items-center gap-1 text-black font-semibold btn-click">
                <i data-lucide="home" class="w-5 h-5"></i>
                <span class="text-[10px]">首頁</span>
            </button>
            <button onclick="switchTab('view-rides')" id="nav-view-rides" class="flex flex-col items-center gap-1 btn-click">
                <i data-lucide="compass" class="w-5 h-5"></i>
                <span class="text-[10px]">探索</span>
            </button>
            <button onclick="switchTab('admin')" id="nav-admin" class="flex flex-col items-center gap-1 btn-click">
                <i data-lucide="shield-alert" class="w-5 h-5"></i>
                <span class="text-[10px]">管理</span>
            </button>
        </div>
    </footer>

    <!-- 登入 Modal -->
    <div id="login-modal" class="fixed inset-0 bg-black/60 backdrop-blur-sm z-50 flex items-end sm:items-center justify-center opacity-0 pointer-events-none transition-opacity duration-300">
        <div class="bg-white w-full sm:max-w-md p-6 rounded-t-3xl sm:rounded-3xl shadow-xl space-y-6 transform translate-y-10 transition-transform duration-300">
            <div class="flex justify-between items-center">
                <div>
                    <h3 class="text-xl font-extrabold tracking-tight">歡迎來到拼一下</h3>
                    <p class="text-xs text-neutral-500">大學生極速登入，即刻體驗拼車</p>
                </div>
                <button onclick="closeLoginModal()" class="p-2 bg-neutral-100 rounded-full btn-click">
                    <i data-lucide="x" class="w-5 h-5"></i>
                </button>
            </div>

            <div class="space-y-4">
                <div>
                    <label class="block text-xs font-bold uppercase tracking-wider text-neutral-400 mb-1.5">暱稱 / 名稱</label>
                    <input type="text" id="login-nickname" placeholder="例如：企管系小陳 / 帥氣學長" 
                        class="w-full bg-neutral-100 border-0 rounded-xl px-4 py-3.5 focus:ring-2 focus:ring-black text-sm transition-all">
                </div>
                
                <button onclick="handleNicknameLogin()" class="w-full bg-black text-white py-4 rounded-xl font-bold btn-click shadow-sm">
                    一鍵創建/登入
                </button>
            </div>
        </div>
    </div>

    <!-- ==================== FIREBASE 核心核心與業務邏輯 ==================== -->
    <script type="module">
        // 導入 Firebase JS 11
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, signOut } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, getDoc, collection, query, onSnapshot, addDoc, updateDoc, deleteDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // 全域配置 (由環境動態提供，若無則降級為本地模擬資料或演示安全沙盒)
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            apiKey: "",
            authDomain: "carpool-demo-project.firebaseapp.com",
            projectId: "carpool-demo-project",
            storageBucket: "carpool-demo-project.appspot.com",
            messagingSenderId: "1234567890",
            appId: "1:12345:web:12345"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'carpool-demo-12345';

        // 全域狀態
        let currentUser = null;
        let currentRides = [];
        let currentFilter = 'all';
        let activeChatRideId = null;
        let chatUnsubscribe = null;

        // Firebase 認證與初始化
        const initAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (err) {
                console.error("Auth Error:", err);
            }
        };

        initAuth();

        onAuthStateChanged(auth, async (user) => {
            currentUser = user;
            if (user) {
                // 檢查是否已有名稱設定
                let profileDoc = await getDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'data'));
                let nickname = '匿名同學';
                if (profileDoc.exists()) {
                    nickname = profileDoc.data().nickname;
                } else {
                    // 初始化使用者設定
                    await setDoc(doc(db, 'artifacts', appId, 'users', user.uid, 'profile', 'data'), {
                        nickname: nickname,
                        createdAt: new Date().toISOString()
                    });
                }
                
                document.getElementById('username-display').innerText = nickname;
                document.getElementById('user-info').classList.remove('hidden');
                document.getElementById('login-trigger').classList.add('hidden');
            } else {
                document.getElementById('user-info').classList.add('hidden');
                document.getElementById('login-trigger').classList.remove('hidden');
            }
        });

        // 監聽所有行程需求
        const setupRidesListener = () => {
            if (!currentUser) return;
            const ridesRef = collection(db, 'artifacts', appId, 'public', 'data', 'rides');
            
            onSnapshot(ridesRef, (snapshot) => {
                const rides = [];
                snapshot.forEach((doc) => {
                    rides.push({ id: doc.id, ...doc.data() });
                });
                
                // 按建立時間由新到舊排序
                rides.sort((a, b) => new Date(b.createdAt) - new Date(a.createdAt));
                currentRides = rides;
                
                renderRides();
                renderAdminRides();
                
                // 更新計數角標
                document.getElementById('active-count-badge').innerText = rides.length;
                document.getElementById('stat-total').innerText = rides.length;
            }, (error) => {
                showToast("資料讀取失敗，請重新整理！");
            });
        };

        // 在使用者登入完成後載入資料
        onAuthStateChanged(auth, (user) => {
            if (user) {
                setupRidesListener();
            }
        });

        // ==================== 頁面互動 / 導航 ====================

        window.switchTab = function(tabName) {
            // 隱藏所有 tab-content
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            // 顯示當前 tab
            const target = document.getElementById(`tab-${tabName}`);
            if (target) {
                target.classList.remove('hidden');
            }

            // 更新底部導航按鈕樣式
            document.querySelectorAll('footer button').forEach(el => {
                el.classList.remove('text-black', 'font-semibold');
                el.classList.add('text-neutral-400');
            });
            const activeNav = document.getElementById(`nav-${tabName}`);
            if (activeNav) {
                activeNav.classList.remove('text-neutral-400');
                activeNav.classList.add('text-black', 'font-semibold');
            }

            // 如果離開聊天室，清除即時聊天監聽器
            if (tabName !== 'chat' && chatUnsubscribe) {
                chatUnsubscribe();
                activeChatRideId = null;
            }

            window.scrollTo({ top: 0, behavior: 'smooth' });
        };

        // ==================== 會員功能 ====================
        window.openLoginModal = function() {
            const modal = document.getElementById('login-modal');
            modal.classList.remove('opacity-0', 'pointer-events-none');
            modal.querySelector('div').classList.remove('translate-y-10');
        };

        window.closeLoginModal = function() {
            const modal = document.getElementById('login-modal');
            modal.classList.add('opacity-0', 'pointer-events-none');
            modal.querySelector('div').classList.add('translate-y-10');
        };

        window.handleNicknameLogin = async function() {
            const nickname = document.getElementById('login-nickname').value.trim();
            if (!nickname) {
                showToast("請輸入暱稱！");
                return;
            }

            if (!currentUser) {
                await initAuth();
            }

            if (currentUser) {
                await setDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'data'), {
                    nickname: nickname,
                    updatedAt: new Date().toISOString()
                });
                
                document.getElementById('username-display').innerText = nickname;
                showToast(`歡迎回來，${nickname}！`);
                closeLoginModal();
            }
        };

        window.logout = async function() {
            await signOut(auth);
            await initAuth();
            showToast("已成功登出");
        };

        // ==================== 業務提交 (我要找拼車 / 我要提供座位) ====================

        window.submitRideRequest = async function(e) {
            e.preventDefault();
            if (!currentUser) {
                openLoginModal();
                showToast("請先點擊上方快速登入唷！");
                return;
            }

            const from = document.getElementById('find-from').value.trim();
            const to = document.getElementById('find-to').value.trim();
            const time = document.getElementById('find-time').value;
            const passengers = document.getElementById('find-passengers').value;

            try {
                const userProfile = await getDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'data'));
                const nickname = userProfile.exists() ? userProfile.data().nickname : '匿名同學';

                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'rides'), {
                    type: 'passenger', // 乘客需求
                    from,
                    to,
                    time: new Date(time).toISOString(),
                    capacity: parseInt(passengers),
                    creatorId: currentUser.uid,
                    creatorName: nickname,
                    joinedUsers: [currentUser.uid], // 第一個加入的就是創建者自己
                    joinedUserNames: [nickname],
                    createdAt: new Date().toISOString()
                });

                showToast("需求發布成功！已自動為你加入該拼車");
                document.getElementById('find-ride-form').reset();
                switchTab('view-rides');
            } catch (err) {
                console.error("Post Error", err);
                showToast("發布失敗，請稍後再試！");
            }
        };

        window.submitRideOffer = async function(e) {
            e.preventDefault();
            if (!currentUser) {
                openLoginModal();
                showToast("請先點擊上方快速登入唷！");
                return;
            }

            const from = document.getElementById('offer-from').value.trim();
            const to = document.getElementById('offer-to').value.trim();
            const time = document.getElementById('offer-time').value;
            const seats = document.getElementById('offer-seats').value;

            try {
                const userProfile = await getDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'data'));
                const nickname = userProfile.exists() ? userProfile.data().nickname : '匿名司機';

                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'rides'), {
                    type: 'driver', // 司機座位
                    from,
                    to,
                    time: new Date(time).toISOString(),
                    capacity: parseInt(seats),
                    creatorId: currentUser.uid,
                    creatorName: nickname,
                    joinedUsers: [currentUser.uid],
                    joinedUserNames: [nickname],
                    createdAt: new Date().toISOString()
                });

                showToast("司機座位發布成功！");
                document.getElementById('offer-ride-form').reset();
                switchTab('view-rides');
            } catch (err) {
                console.error("Post Error", err);
                showToast("發布失敗，請稍後再試！");
            }
        };

        // ==================== 渲染拼車清單 ====================

        window.filterRides = function(type) {
            currentFilter = type;
            // 更新 UI 樣式
            ['all', 'passenger', 'driver'].forEach(t => {
                const btn = document.getElementById(`filter-${t}`);
                if (t === type) {
                    btn.className = "text-xs px-3 py-1.5 rounded-full font-semibold bg-black text-white transition-all btn-click";
                } else {
                    btn.className = "text-xs px-3 py-1.5 rounded-full font-semibold bg-neutral-100 text-neutral-500 transition-all btn-click";
                }
            });
            renderRides();
        };

        function renderRides() {
            const listContainer = document.getElementById('rides-list');
            listContainer.innerHTML = '';

            const filtered = currentRides.filter(ride => {
                if (currentFilter === 'all') return true;
                return ride.type === currentFilter;
            });

            if (filtered.length === 0) {
                listContainer.innerHTML = `
                    <div class="text-center py-16 bg-white rounded-3xl border border-neutral-100 p-8 space-y-4">
                        <div class="bg-neutral-50 w-12 h-12 rounded-full flex items-center justify-center mx-auto text-neutral-400">
                            <i data-lucide="info" class="w-6 h-6"></i>
                        </div>
                        <div>
                            <p class="font-bold text-neutral-800">目前沒有相關拼車項目</p>
                            <p class="text-xs text-neutral-400">趕快來點選首頁「發布」第一個行程吧！</p>
                        </div>
                    </div>
                `;
                lucide.createIcons();
                return;
            }

            filtered.forEach(ride => {
                const isCreator = currentUser && ride.creatorId === currentUser.uid;
                const isJoined = currentUser && ride.joinedUsers.includes(currentUser.uid);
                
                // 計算剩餘/已加入人數
                const currentParticipants = ride.joinedUsers ? ride.joinedUsers.length : 1;
                const seatsStatusText = ride.type === 'driver' 
                    ? `可供座位: ${ride.capacity} | 已預訂: ${currentParticipants - 1}人`
                    : `尋求人數: ${ride.capacity} | 已有: ${currentParticipants}人同拼`;

                // 格式化日期時間
                const dateObj = new Date(ride.time);
                const formattedTime = `${dateObj.getMonth() + 1}/${dateObj.getDate()} (${getWeekday(dateObj)}) ${String(dateObj.getHours()).padStart(2, '0')}:${String(dateObj.getMinutes()).padStart(2, '0')}`;

                const card = document.createElement('div');
                card.className = "bg-white border border-neutral-150 rounded-2xl p-5 shadow-sm space-y-4 hover:shadow-md transition-all";
                card.innerHTML = `
                    <div class="flex justify-between items-start">
                        <div class="flex items-center gap-2">
                            <span class="px-2.5 py-1 rounded-full text-[10px] font-bold uppercase tracking-wider ${ride.type === 'driver' ? 'bg-emerald-50 text-emerald-600' : 'bg-blue-50 text-blue-600'}">
                                ${ride.type === 'driver' ? '🚗 有車出發' : '🙋 找人拼車'}
                            </span>
                            <span class="text-[11px] text-neutral-400 font-medium">由 ${ride.creatorName} 發布</span>
                        </div>
                        <span class="text-xs font-semibold text-neutral-700 bg-neutral-100 px-2 py-0.5 rounded">
                            ${seatsStatusText}
                        </span>
                    </div>

                    <!-- 起訖路線 -->
                    <div class="space-y-2">
                        <div class="flex items-center gap-3">
                            <div class="w-2.5 h-2.5 rounded-full bg-black flex-shrink-0"></div>
                            <p class="text-sm font-semibold text-neutral-800">${ride.from}</p>
                        </div>
                        <div class="w-0.5 h-4 bg-neutral-200 ml-[4px]"></div>
                        <div class="flex items-center gap-3">
                            <div class="w-2.5 h-2.5 rounded-full border border-black bg-white flex-shrink-0"></div>
                            <p class="text-sm font-semibold text-neutral-800">${ride.to}</p>
                        </div>
                    </div>

                    <div class="flex items-center justify-between text-xs text-neutral-500 pt-2 border-t border-neutral-100">
                        <div class="flex items-center gap-1.5 font-medium">
                            <i data-lucide="clock" class="w-4 h-4 text-neutral-400"></i>
                            <span>${formattedTime}</span>
                        </div>
                    </div>

                    <!-- 操作按鈕 -->
                    <div class="flex gap-2 pt-2">
                        ${isJoined 
                            ? `<button onclick="openChat('${ride.id}')" class="flex-grow bg-black text-white text-xs font-bold py-3 px-4 rounded-xl flex items-center justify-center gap-1.5 btn-click">
                                    <i data-lucide="message-square" class="w-4 h-4"></i>
                                    進入即時對話 (${ride.joinedUsers.length}人)
                               </button>
                               ${!isCreator ? `
                               <button onclick="leaveRide('${ride.id}')" class="bg-red-50 text-red-600 hover:bg-red-100 text-xs font-bold px-4 py-3 rounded-xl btn-click">
                                    退出
                               </button>` : ''}
                              `
                            : `<button onclick="joinRide('${ride.id}')" class="flex-grow bg-neutral-100 hover:bg-neutral-200 text-neutral-800 text-xs font-bold py-3 rounded-xl flex items-center justify-center gap-1.5 btn-click">
                                    <i data-lucide="user-plus" class="w-4 h-4"></i>
                                    我想加入拼車
                               </button>`
                        }
                    </div>
                `;

                listContainer.appendChild(card);
            });

            lucide.createIcons();
        }

        // ==================== 行程狀態操作 ====================

        window.joinRide = async function(rideId) {
            if (!currentUser) {
                openLoginModal();
                showToast("加入拼車前請先「快速登入」唷！");
                return;
            }

            try {
                const userProfile = await getDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'data'));
                const nickname = userProfile.exists() ? userProfile.data().nickname : '匿名同學';

                const rideRef = doc(db, 'artifacts', appId, 'public', 'data', 'rides', rideId);
                const rideDoc = await getDoc(rideRef);
                if (!rideDoc.exists()) return;

                const rideData = rideDoc.data();
                if (rideData.joinedUsers.includes(currentUser.uid)) {
                    showToast("你已經在成員列表中了！");
                    return;
                }

                const updatedUsers = [...rideData.joinedUsers, currentUser.uid];
                const updatedUserNames = [...(rideData.joinedUserNames || []), nickname];

                await updateDoc(rideRef, {
                    joinedUsers: updatedUsers,
                    joinedUserNames: updatedUserNames
                });

                showToast("成功加入拼車！你可以前往聊天室協調囉 💬");
                switchTab('view-rides');
            } catch (err) {
                console.error(err);
                showToast("加入失敗，請稍後再試。");
            }
        };

        window.leaveRide = async function(rideId) {
            if (!currentUser) return;

            try {
                const rideRef = doc(db, 'artifacts', appId, 'public', 'data', 'rides', rideId);
                const rideDoc = await getDoc(rideRef);
                if (!rideDoc.exists()) return;

                const rideData = rideDoc.data();
                const userIndex = rideData.joinedUsers.indexOf(currentUser.uid);
                if (userIndex === -1) return;

                const updatedUsers = [...rideData.joinedUsers];
                updatedUsers.splice(userIndex, 1);

                const updatedUserNames = [...(rideData.joinedUserNames || [])];
                if (updatedUserNames[userIndex]) {
                    updatedUserNames.splice(userIndex, 1);
                }

                await updateDoc(rideRef, {
                    joinedUsers: updatedUsers,
                    joinedUserNames: updatedUserNames
                });

                showToast("已退出該拼車項目");
            } catch (err) {
                console.error(err);
                showToast("退出失敗");
            }
        };

        // ==================== 即時聊天室邏輯 ====================

        window.openChat = async function(rideId) {
            activeChatRideId = rideId;
            switchTab('chat');

            // 讀取該拼車項目資訊
            const rideDoc = await getDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rides', rideId));
            if (!rideDoc.exists()) return;
            const rideData = rideDoc.data();

            document.getElementById('chat-title-route').innerText = `${rideData.from} ➔ ${rideData.to}`;
            document.getElementById('chat-badge-type').innerText = rideData.type === 'driver' ? '有車拼車' : '徵求拼車';

            // 即時監聽該行程的聊天訊息
            const chatRef = collection(db, 'artifacts', appId, 'public', 'data', 'rides', rideId, 'messages');
            
            chatUnsubscribe = onSnapshot(chatRef, (snapshot) => {
                const messages = [];
                snapshot.forEach(doc => {
                    messages.push({ id: doc.id, ...doc.data() });
                });

                // 按時間排序
                messages.sort((a, b) => new Date(a.timestamp) - new Date(b.timestamp));
                renderChatMessages(messages);
            });
        };

        function renderChatMessages(messages) {
            const container = document.getElementById('chat-messages');
            container.innerHTML = '';

            if (messages.length === 0) {
                container.innerHTML = `
                    <div class="text-center text-xs text-neutral-400 py-10">
                        💬 大家都在車上了！發個訊息跟夥伴打聲招呼吧。
                    </div>
                `;
                return;
            }

            messages.forEach(msg => {
                const isMe = currentUser && msg.senderId === currentUser.uid;
                const wrapper = document.createElement('div');
                wrapper.className = `flex flex-col ${isMe ? 'items-end' : 'items-start'} max-w-full`;
                
                wrapper.innerHTML = `
                    <div class="text-[10px] text-neutral-400 mb-0.5 px-1 font-medium">${msg.senderName}</div>
                    <div class="max-w-[80%] rounded-2xl px-4 py-2.5 text-sm ${isMe ? 'bg-black text-white rounded-br-none' : 'bg-neutral-100 text-neutral-900 rounded-bl-none'}">
                        ${escapeHTML(msg.text)}
                    </div>
                `;
                container.appendChild(wrapper);
            });

            // 捲動到最底下
            container.scrollTop = container.scrollHeight;
        }

        window.sendChatMessage = async function(e) {
            e.preventDefault();
            if (!currentUser || !activeChatRideId) return;

            const input = document.getElementById('chat-input');
            const text = input.value.trim();
            if (!text) return;

            try {
                const userProfile = await getDoc(doc(db, 'artifacts', appId, 'users', currentUser.uid, 'profile', 'data'));
                const nickname = userProfile.exists() ? userProfile.data().nickname : '同學';

                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'rides', activeChatRideId, 'messages'), {
                    text: text,
                    senderId: currentUser.uid,
                    senderName: nickname,
                    timestamp: new Date().toISOString()
                });

                input.value = '';
            } catch (err) {
                console.error("Chat Error", err);
                showToast("發送失敗");
            }
        };

        // ==================== 後台管理 ====================

        function renderAdminRides() {
            const container = document.getElementById('admin-rides-list');
            container.innerHTML = '';

            if (currentRides.length === 0) {
                container.innerHTML = `<p class="text-xs text-neutral-400 text-center py-4">目前無任何行程</p>`;
                return;
            }

            currentRides.forEach(ride => {
                const row = document.createElement('div');
                row.className = "bg-white p-3 rounded-xl flex justify-between items-center text-xs border border-neutral-100 shadow-sm";
                row.innerHTML = `
                    <div class="truncate max-w-[70%]">
                        <span class="font-bold text-neutral-800">[${ride.type === 'driver' ? '司機' : '乘客'}]</span>
                        <p class="truncate font-semibold text-neutral-600">${ride.from} ➔ ${ride.to}</p>
                        <span class="text-[10px] text-neutral-400">發布者: ${ride.creatorName}</span>
                    </div>
                    <button onclick="adminDeleteRide('${ride.id}')" class="bg-red-500 text-white px-3 py-1.5 rounded-lg font-bold btn-click">
                        刪除
                    </button>
                `;
                container.appendChild(row);
            });
        }

        window.adminDeleteRide = async function(rideId) {
            const confirmAction = confirm("您確定要刪除此拼車項目嗎？(這將會同時清理所有聊天內容)");
            if (!confirmAction) return;

            try {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'rides', rideId));
                showToast("已成功從後台刪除該項目");
            } catch (err) {
                console.error("Delete Error", err);
                showToast("刪除失敗");
            }
        };

        // ==================== 輔助工具 ====================

        function getWeekday(date) {
            const weekdays = ["日", "一", "二", "三", "四", "五", "六"];
            return weekdays[date.getDay()];
        }

        function showToast(text) {
            const toast = document.getElementById('toast');
            document.getElementById('toast-text').innerText = text;
            toast.classList.remove('opacity-0', 'translate-y-[-100px]');
            toast.classList.add('opacity-100', 'translate-y-0');
            setTimeout(() => {
                toast.classList.remove('opacity-100', 'translate-y-0');
                toast.classList.add('opacity-0', 'translate-y-[-100px]');
            }, 3000);
        }

        function escapeHTML(str) {
            return str.replace(/[&<>'"]/g, 
                tag => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', "'": '&#39;', '"': '&quot;' }[tag] || tag)
            );
        }
    </script>
</body>
</html>