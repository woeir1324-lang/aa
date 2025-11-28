<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CODE5IVE - 경남 해양 환경 예측 플랫폼</title>
    <!-- Tailwind CSS CDN --><script src="https://cdn.tailwindcss.com/"></script>
    <!-- Inter Font --><style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100..900&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f7f9; /* Dashboard background color */
        }
        /* CSS for smooth updates */
        #map-time-ref {
            transition: opacity 0.1s ease-in-out, transform 0.1s ease-in-out;
        }

        /* Risk Level Colors and Animations */
        .risk-level-high { background-color: #f87171; animation: pulse-red 1.5s infinite; }
        .risk-level-medium { background-color: #facc15; }
        .risk-level-low { background-color: #4ade80; }
        .risk-level-mhigh { background-color: #f97316; }

        @keyframes pulse-red {
            0%, 100% { box-shadow: 0 0 0 0 rgba(248, 113, 113, 0.7); }
            50% { box-shadow: 0 0 0 10px rgba(248, 113, 113, 0); }
        }
        @keyframes pulse-slow {
            0%, 100% { transform: scale(1); opacity: 0.9; }
            50% { transform: scale(1.1); opacity: 1; }
        }
        .animate-pulse-slow { animation: pulse-slow 3s infinite ease-in-out; }

        /* Map Zone Styles */
        .map-zone {
            transition: all 0.3s;
            cursor: pointer;
            text-align: center;
            padding: 0.2rem 0.5rem;
            color: white; 
            font-size: 0.65rem;
            font-weight: bold;
            border-radius: 9999px;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -2px rgba(0, 0, 0, 0.06);
            line-height: 1.2;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 3px solid transparent; 
        }
        .map-zone.selected {
            border: 3px solid #3b82f6; 
            box-shadow: 0 0 0 5px rgba(59, 130, 246, 0.5);
        }

        /* Map Container Background Image (Responsive) */
        #map-container {
            background-image: url('https://cdn.kntoday.co.kr/news/photo/202105/158846_159295_1042.jpg'); 
            background-size: cover; 
            background-position: center; 
            background-repeat: no-repeat;
            background-color: transparent; 
            /* 모바일에서 지도의 최소 높이를 보장 */
            min-height: 300px; 
        }
        /* Map Placeholder to ensure full coverage over the image */
        .map-placeholder {
            background-color: rgba(255, 255, 255, 0.0);
        }
    </style>
    <!-- Firebase Imports --><script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, onSnapshot, query, setDoc, doc, Timestamp, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        import { setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";
        
        // Firestore Log Level Debug (Mandatory)
        setLogLevel('Debug');

        const userIdDisplay = document.getElementById('user-id-display');
        const statusDisplay = document.getElementById('auth-status');
        
        // Global variables already provided by the environment (Mandatory)
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db, auth, userId;

        // NEW: Global variable to track if the user is *not* anonymous (e.g., signed up/logged in)
        window.isUserLoggedIn = false; 

        async function initializeFirebase() {
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    statusDisplay.textContent = '인증 오류: Firebase 설정이 누락되었습니다.';
                    console.error("Firebase config is missing.");
                    return;
                }
                
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                
                // 1. Authenticate the user
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                // 2. Set up Auth State Listener
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        userIdDisplay.textContent = `User ID: ${userId}`;
                        
                        // Check if the user is authenticated (not anonymous)
                        window.isUserLoggedIn = !user.isAnonymous;
                        
                        if (user.isAnonymous) {
                             statusDisplay.textContent = '인증 완료 (익명 사용자)';
                        } else {
                             statusDisplay.textContent = '인증 완료 (회원)';
                        }
                       
                        // Start data listeners and app logic after successful authentication
                        startAppLogic(userId);
                    } else {
                        userId = crypto.randomUUID(); // Fallback for unauthenticated
                        window.isUserLoggedIn = false; // User is definitively not logged in
                        userIdDisplay.textContent = `User ID (Fallback): ${userId.substring(0, 8)}...`;
                        statusDisplay.textContent = '인증 실패 또는 익명 사용자';
                    }
                });

            } catch (error) {
                console.error("Firebase Initialization or Auth Failed:", error);
                statusDisplay.textContent = `인증 실패: ${error.message}`;
            }
        }

        // --- App Specific Logic (Mock Data Fetching) ---
        function startAppLogic(currentUserId) {
            // For this mock-up, we'll demonstrate setting up a listener for mock "Alerts" data
            const alertsCollectionPath = `/artifacts/${appId}/users/${currentUserId}/alerts`;
            
            // Mock Data Structure
            const mockAlert = { 
                level: 'HIGH', 
                location: '사천시 인근 해역', 
                time: Timestamp.now(), 
                message: '탁도 급증으로 오염물질 유입 가능성 높음. 조사 요망' 
            };
            
            // Check if mock data exists, if not, set it once (optional, for demo)
            const mockDocRef = doc(db, alertsCollectionPath, 'current_risk');
            getDoc(mockDocRef).then(docSnap => {
                if (!docSnap.exists()) {
                    setDoc(mockDocRef, mockAlert);
                }
            });

            // Real-time listener for alerts (Updates the UI)
            const q = query(collection(db, alertsCollectionPath));
            onSnapshot(q, (snapshot) => {
                let latestAlert = null;
                snapshot.forEach((doc) => {
                    // In a real app, this logic would process multiple alerts
                    const data = doc.data();
                    if (!latestAlert || data.time.toDate() > latestAlert.time.toDate()) {
                        latestAlert = { id: doc.id, ...data };
                    }
                });

                if (latestAlert) {
                    const alertDiv = document.getElementById('latest-alert');
                    // Check for HIGH risk specifically for the strong visual cue
                    const isHighRisk = latestAlert.level === 'HIGH' || latestAlert.level === 'M-HIGH'; 
                    
                    alertDiv.textContent = `[${latestAlert.level}] ${latestAlert.location}: ${latestAlert.message} (${latestAlert.time.toDate().toLocaleTimeString('ko-KR')})`;
                    
                    if (isHighRisk) {
                        alertDiv.className = 'p-3 rounded-lg text-white font-bold transition-all duration-300 risk-level-high shadow-lg';
                    } else {
                        alertDiv.className = 'p-3 rounded-lg text-white font-bold transition-all duration-300 bg-green-500 shadow-lg';
                    }
                }
            }, (error) => {
                console.error("Error listening to alerts:", error);
                document.getElementById('latest-alert').textContent = '데이터 로딩 중 오류 발생';
            });
        }
        
        document.addEventListener('DOMContentLoaded', initializeFirebase);
    </script>
</head>
<body>

    <!-- Header & User Info (Always full width, fixed) --><header class="bg-white shadow-md p-4 sticky top-0 z-10">
        <div class="max-w-7xl mx-auto flex justify-between items-center">
            <h1 class="text-2xl md:text-3xl font-extrabold text-blue-700">
                <span class="text-green-500">CODE5IVE</span> RISA 예측 플랫폼
            </h1>
            <div class="flex items-center space-x-2 md:space-x-4">
                <button id="auth-button" onclick="showAuthModal('auth-modal')" class="px-3 py-1 md:px-4 md:py-2 bg-blue-600 text-white font-bold text-sm rounded-lg shadow-md hover:bg-blue-700 transition duration-150">
                    로그인 / 회원가입
                </button>
                <div class="text-right text-xs hidden sm:block">
                    <p id="user-id-display" class="font-medium text-gray-700 truncate max-w-xs"></p>
                    <p id="auth-status" class="text-xs text-gray-500 mt-1">인증 상태 확인 중...</p>
                </div>
            </div>
        </div>
    </header>

    <main class="max-w-7xl mx-auto p-4 md:p-8">
        <!-- Main Title and Description --><section class="mb-6 md:mb-8">
            <h2 class="text-3xl md:text-4xl font-bold text-gray-800 mb-2">
                경남 해역 실시간 모니터링
            </h2>
            <p class="text-gray-600 text-base md:text-lg">
                AI 분석을 통해 오염 악화 전 위험을 미리 예측하고, 어민과 지자체가 즉각 대응할 수 있는 정보를 제공합니다.
            </p>
        </section>

        <!-- Risk Alert Card --><div class="mb-6 md:mb-8">
            <div id="latest-alert" class="p-3 rounded-lg text-white font-bold transition-all duration-300 risk-level-high shadow-lg text-sm md:text-base">
                최신 해양 위험 알림이 없습니다. (데이터 로딩 중...)
            </div>
        </div>

        <!-- Main Dashboard Layout (Map + Stats) -->
        <!-- 모바일: 세로 배치 (col-span-1), PC: 좌우 2:1 배치 (lg:grid-cols-3) --><div class="grid grid-cols-1 lg:grid-cols-3 gap-6 md:gap-8">
            
            <!-- Map Visualization Area (PC: 2/3 width, Mobile: Full width) --><div id="map-container" class="lg:col-span-2 rounded-xl shadow-2xl overflow-hidden aspect-video">
                <div class="w-full h-full map-placeholder flex flex-col p-4 rounded-xl relative">
                    
                    <!-- Map Title, Time & TOGGLE BUTTONS --><div class="z-10 bg-white bg-opacity-90 p-3 rounded-lg shadow-md mb-4 self-start">
                        <!-- Dynamic Title --><p id="map-title" class="text-lg md:text-xl font-bold text-gray-800">경남 해역 위험도 (현재 상황)</p>
                        <!-- Dynamic Timestamp/Reference --><p id="map-time-ref" class="text-xs md:text-sm text-gray-600">실시간 갱신 중...</p>
                        
                        <!-- Toggle Buttons -->
                        <div class="flex space-x-2 mt-3">
                            <button id="mode-current" data-mode="current" onclick="toggleMapView('current')" class="px-3 py-1 text-xs md:text-sm rounded-full font-medium transition-colors duration-150 bg-blue-500 text-white shadow-md">
                                현재 상황
                            </button>
                            <button id="mode-predicted" data-mode="predicted" onclick="toggleMapView('predicted')" class="px-3 py-1 text-xs md:text-sm rounded-full font-medium transition-colors duration-150 bg-gray-200 text-gray-700 hover:bg-gray-300">
                                6시간 예측 현황
                            </button>
                        </div>
                    </div>

                    <!-- Risk Zones (Mock Data Overlay) - ADDED onclick HANDLER -->
                    
                    <!-- 1. 거제시 해역 (Geoje) -->
                    <div data-zone="geoje" data-current-level="HIGH" data-predicted-level="M-HIGH" onclick="handleZoneClick(this)"
                         class="absolute w-[80px] h-[60px] bg-red-600 bg-opacity-80 animate-pulse-slow map-zone" 
                         style="top: 82%; left: 88%; transform: translate(-50%, -50%);">
                        거제시<br/><span class="zone-level">(HIGH)</span>
                    </div>
                    
                    <!-- 2. 통영시 해역 (Tongyeong) -->
                    <div data-zone="tongyeong" data-current-level="M-HIGH" data-predicted-level="HIGH" onclick="handleZoneClick(this)"
                         class="absolute w-[80px] h-[60px] bg-orange-500 bg-opacity-80 map-zone" 
                         style="top: 78%; left: 70%; transform: translate(-50%, -50%);">
                        통영시<br/><span class="zone-level">(M-HIGH)</span>
                    </div>

                    <!-- 3. 남해군 해역 (Namhae) -->
                    <div data-zone="namhae" data-current-level="LOW" data-predicted-level="MEDIUM" onclick="handleZoneClick(this)"
                         class="absolute w-[70px] h-[50px] bg-green-500 bg-opacity-70 map-zone" 
                         style="top: 85%; left: 22%; transform: translate(-50%, -50%);">
                        남해군<br/><span class="zone-level">(LOW)</span>
                    </div>

                    <!-- 4. 사천시 해역 (Sacheon) - Default selection on load -->
                    <div data-zone="sacheon" data-current-level="MEDIUM" data-predicted-level="M-HIGH" onclick="handleZoneClick(this)"
                         class="absolute w-[70px] h-[60px] bg-yellow-500 bg-opacity-70 map-zone selected" 
                         style="top: 70%; left: 35%; transform: translate(-50%, -50%);">
                        사천시<br/><span class="zone-level">(MEDIUM)</span>
                    </div>

                    <!-- 5. 고성군 해역 (Goseong) -->
                    <div data-zone="goseong" data-current-level="LOW" data-predicted-level="MEDIUM" onclick="handleZoneClick(this)"
                         class="absolute w-[70px] h-[50px] bg-green-500 bg-opacity-70 map-zone" 
                         style="top: 68%; left: 52%; transform: translate(-50%, -50%);">
                        고성군<br/><span class="zone-level">(LOW)</span>
                    </div>

                    <!-- 6. 창원시 해역 (Changwon) -->
                    <div data-zone="changwon" data-current-level="M-HIGH" data-predicted-level="HIGH" onclick="handleZoneClick(this)"
                         class="absolute w-[80px] h-[60px] bg-orange-500 bg-opacity-80 map-zone" 
                         style="top: 50%; left: 65%; transform: translate(-50%, -50%);">
                        창원시<br/><span class="zone-level">(M-HIGH)</span>
                    </div>

                    <!-- 7. 진주시 (inland, but affected by river outflow) -->
                    <div data-zone="jinju" data-current-level="LOW" data-predicted-level="LOW" onclick="handleZoneClick(this)"
                         class="absolute w-[70px] h-[50px] bg-green-500 bg-opacity-70 map-zone" 
                         style="top: 55%; left: 40%; transform: translate(-50%, -50%);">
                        진주시<br/><span class="zone-level">(LOW)</span>
                    </div>

                    <!-- 8. 하동군 (Hadong) -->
                    <div data-zone="hadong" data-current-level="MEDIUM" data-predicted-level="LOW" onclick="handleZoneClick(this)"
                         class="absolute w-[70px] h-[50px] bg-yellow-500 bg-opacity-70 map-zone" 
                         style="top: 65%; left: 15%; transform: translate(-50%, -50%);">
                        하동군<br/><span class="zone-level">(MEDIUM)</span>
                    </div>
                    
                    <!-- Prediction Layer Indicator (Mock) --><div class="absolute bottom-4 left-4 bg-gray-800 bg-opacity-80 p-2 md:p-3 rounded-lg text-white text-xs md:text-sm">
                        <p class="font-bold mb-1">🌊 예측 경로 레이어</p>
                        <p class="text-xs">적조 이동: <span class="text-red-300">➜➜➜</span> | 쓰레기 이동: <span class="text-yellow-300">---</span></p>
                    </div>

                    <!-- Legend (White background for readability) --><div class="absolute top-4 right-4 bg-white bg-opacity-90 p-2 md:p-3 rounded-lg shadow-md z-10">
                        <h4 class="font-semibold text-gray-700 mb-2 text-sm md:text-base">위험 등급</h4>
                        <!-- Use class names for easy lookup in JS --><div class="flex items-center space-x-2 mb-1">
                            <span class="w-3 h-3 rounded-full risk-level-high"></span>
                            <span class="text-xs md:text-sm text-gray-600">매우 높음 (HIGH)</span>
                        </div>
                        <div class="flex items-center space-x-2 mb-1">
                            <span class="w-3 h-3 rounded-full risk-level-mhigh"></span>
                            <span class="text-xs md:text-sm text-gray-600">높음 (M-HIGH)</span>
                        </div>
                         <div class="flex items-center space-x-2 mb-1">
                            <span class="w-3 h-3 rounded-full risk-level-medium"></span>
                            <span class="text-xs md:text-sm text-gray-600">보통 (MEDIUM)</span>
                        </div>
                        <div class="flex items-center space-x-2">
                            <span class="w-3 h-3 rounded-full risk-level-low"></span>
                            <span class="text-xs md:text-sm text-gray-600">낮음 (LOW)</span>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Key Metrics & Features (PC: 1/3 width, Mobile: Full width below map) --><div class="lg:col-span-1 space-y-4 md:space-y-6">
                
                <!-- AI Prediction Status Card (CHANGED to Selected Region Status) --><div id="region-status-card" class="bg-white p-5 md:p-6 rounded-xl shadow-xl border-t-4 border-blue-500">
                    <div class="flex items-center justify-between mb-2 md:mb-3">
                        <h3 class="text-lg md:text-xl font-semibold text-gray-700">선택 지역 상세 현황</h3>
                        <span id="ai-status-badge" class="text-xs md:text-sm font-medium text-green-600 bg-green-100 px-3 py-1 rounded-full">사천시</span>
                    </div>
                    <p class="text-2xl md:text-3xl font-extrabold text-gray-900">
                        <span id="region-risk-display" class="text-yellow-500">보통 위험</span>
                    </p>
                    <p class="text-xs md:text-sm text-gray-500 mt-1">현재 지표를 기반으로 분석된 위험 등급입니다.</p>
                </div>

                <!-- Metrics Cards - RESPONSIVE GRID (Mobile 1 column, Tablet 2 columns, PC 2 columns) --><div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-2 gap-4">
                    <div class="bg-white p-4 md:p-5 rounded-xl shadow-lg border-b-2 border-green-500">
                        <p class="text-sm font-medium text-gray-500">수온 (ºC)</p>
                        <p id="temp-value" class="text-xl md:text-2xl font-bold text-gray-900 mt-1">21.5</p>
                        <p id="temp-trend" class="text-xs text-green-500">▲ 0.2 (24H)</p>
                    </div>
                    <div class="bg-white p-4 md:p-5 rounded-xl shadow-lg border-b-2 border-yellow-500">
                        <p class="text-sm font-medium text-gray-500">탁도 (NTU)</p>
                        <p id="turbidity-value" class="text-xl md:text-2xl font-bold text-gray-900 mt-1">3.1</p>
                        <p id="turbidity-trend" class="text-xs text-red-500">▲ 1.5 (급증)</p>
                    </div>
                    <div class="bg-white p-4 md:p-5 rounded-xl shadow-lg border-b-2 border-red-500">
                        <p class="text-sm font-medium text-gray-500">클로로필-a (㎍/L)</p>
                        <p id="chlorophyll-value" class="text-xl md:text-2xl font-bold text-gray-900 mt-1">15.8</p>
                        <p id="chlorophyll-trend" class="text-xs text-red-500">AI 위험 감지</p>
                    </div>
                    <div class="bg-white p-4 md:p-5 rounded-xl shadow-lg border-b-2 border-blue-500">
                        <p class="text-sm font-medium text-gray-500">데이터 갱신 시간</p>
                        <p id="data-refresh-value" class="text-xl md:text-2xl font-bold text-gray-900 mt-1">11:00:00</p>
                        <p id="data-refresh-source" class="text-xs text-gray-500">RISA API 기준</p>
                    </div>
                </div>

                <!-- Call to Action / Custom Alert -->
                <div class="p-5 md:p-6 bg-blue-600 rounded-xl shadow-2xl text-white">
                    <p class="text-base md:text-lg font-semibold mb-2">어민/지자체 맞춤 알림 설정</p>
                    <p class="text-xs md:text-sm opacity-90 mb-4">필요한 해역 정보만 골라 받고 즉각 대응하세요.</p>
                    <button id="cta-button" class="w-full bg-white text-blue-600 font-bold py-3 rounded-lg shadow-md hover:bg-gray-100 transition duration-150 text-sm md:text-base">
                        알림 설정하기
                    </button>
                </div>
            </div>
        </div>
    </main>

    <!-- Footer --><footer class="mt-8 md:mt-12 p-6 text-center text-gray-500 text-xs md:text-sm border-t border-gray-200">
        &copy; 2025 CODE5IVE Project | 마산대학교 X RISE 해커톤 기획 | 데이터 기반 청정 해역 유지
    </footer>

    <!-- Authentication Modal (Hidden by default) -->
    <div id="auth-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 hidden items-center justify-center p-4 z-50">
        <div class="bg-white p-6 md:p-8 rounded-xl shadow-2xl max-w-sm md:max-w-lg w-full">
            <div class="flex justify-between items-center mb-6">
                <h3 id="auth-modal-title" class="text-xl md:text-2xl font-bold text-blue-700">로그인</h3>
                <button onclick="closeAuthModal('auth-modal')" class="text-gray-500 hover:text-gray-700 text-2xl font-light">&times;</button>
            </div>

            <!-- Tab Buttons -->
            <div class="flex border-b border-gray-200 mb-6">
                <button id="tab-login" onclick="switchAuthView('login')" class="flex-1 py-3 text-center font-semibold transition-colors border-b-2 border-blue-600 text-blue-600 text-sm md:text-base">로그인</button>
                <button id="tab-signup" onclick="switchAuthView('signup')" class="flex-1 py-3 text-center font-semibold transition-colors border-b-2 border-transparent text-gray-500 hover:text-blue-600 hover:border-blue-600 text-sm md:text-base">회원가입</button>
            </div>

            <!-- Login Form (Default View) -->
            <div id="login-view">
                <form onsubmit="handleAuthSubmit(event, 'login')">
                    <div class="mb-4">
                        <label for="login-email" class="block text-sm font-medium text-gray-700 mb-1">이메일</label>
                        <input type="email" id="login-email" name="email" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-sm" placeholder="user@example.com" required>
                    </div>
                    <div class="mb-6">
                        <label for="login-password" class="block text-sm font-medium text-gray-700 mb-1">비밀번호</label>
                        <input type="password" id="login-password" name="password" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-sm" placeholder="비밀번호" required>
                    </div>
                    <button type="submit" class="w-full bg-blue-600 text-white font-bold py-3 rounded-lg shadow-lg hover:bg-blue-700 transition duration-150 text-base">로그인 (UI 모형)</button>
                </form>
            </div>

            <!-- Sign Up Form (Hidden by default) -->
            <div id="signup-view" class="hidden">
                <form onsubmit="handleAuthSubmit(event, 'signup')">
                    <div class="mb-4">
                        <label for="signup-email" class="block text-sm font-medium text-gray-700 mb-1">이메일</label>
                        <input type="email" id="signup-email" name="email" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-sm" placeholder="user@example.com" required>
                    </div>
                    <div class="mb-6">
                        <label for="signup-password" class="block text-sm font-medium text-gray-700 mb-1">비밀번호 (6자 이상)</label>
                        <input type="password" id="signup-password" name="password" class="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500 text-sm" placeholder="새 비밀번호" minlength="6" required>
                    </div>
                    <button type="submit" class="w-full bg-green-500 text-white font-bold py-3 rounded-lg shadow-lg hover:bg-green-600 transition duration-150 text-base">회원가입 (UI 모형)</button>
                </form>
            </div>

            <!-- Mock Message Display -->
            <p id="auth-message" class="mt-4 text-xs md:text-sm text-center text-gray-600"></p>
        </div>
    </div>

    <!-- Simple Modal/Message Box for Action Mock (Replaces alert()) -->
    <div id="action-modal" class="fixed inset-0 bg-gray-900 bg-opacity-75 hidden items-center justify-center p-4 z-50">
        <div class="bg-white p-6 rounded-xl shadow-2xl max-w-xs md:max-w-sm w-full">
            <p id="modal-title" class="text-xl font-bold text-blue-700 mb-4">접근 제한 알림</p>
            <p id="modal-content" class="text-gray-700 text-sm">해당 기능은 회원만 이용 가능합니다. 로그인 또는 회원가입 후 이용해주세요.</p>
            <button id="close-modal" class="mt-6 w-full bg-blue-500 text-white py-2 rounded-lg hover:bg-blue-600 transition">확인</button>
        </div>
    </div>

    <script>
        // 현재 맵 뷰 모드 추적 (기본값: current)
        let currentMapViewMode = 'current';
        let currentSelectedRegion = 'sacheon'; // 초기 선택 지역

        // --- 지역별 모의 데이터 (수온, 탁도, 클로로필-a) ---
        // 'predicted-level'은 이제 6시간 후 예측 상황을 나타냅니다.
        const regionData = {
            'geoje': { name: '거제시', temp: 20.8, turbidity: 4.5, chlorophyll: 18.2, trendTemp: '▲ 0.1', trendTurbidity: '▲ 1.0 (주의)', trendChlorophyll: 'AI 위험 감지', riskLevel: 'HIGH' },
            'tongyeong': { name: '통영시', temp: 21.2, turbidity: 3.8, chlorophyll: 16.5, trendTemp: '▼ 0.3', trendTurbidity: '▲ 0.5', trendChlorophyll: 'AI 위험 감지', riskLevel: 'M-HIGH' },
            'namhae': { name: '남해군', temp: 22.0, turbidity: 1.2, chlorophyll: 5.1, trendTemp: '▲ 0.5', trendTurbidity: '▼ 0.2', trendChlorophyll: '정상', riskLevel: 'LOW' },
            'sacheon': { name: '사천시', temp: 21.5, turbidity: 3.1, chlorophyll: 15.8, trendTemp: '▲ 0.2', trendTurbidity: '▲ 1.5 (급증)', trendChlorophyll: 'AI 위험 감지', riskLevel: 'MEDIUM' },
            'goseong': { name: '고성군', temp: 20.5, turbidity: 1.8, chlorophyll: 7.2, trendTemp: '▼ 0.1', trendTurbidity: '▼ 0.1', trendChlorophyll: '정상', riskLevel: 'LOW' },
            'changwon': { name: '창원시', temp: 19.9, turbidity: 5.2, chlorophyll: 21.0, trendTemp: '▲ 0.8 (급상승)', trendTurbidity: '▲ 2.0 (심각)', trendChlorophyll: 'AI 위험 감지', riskLevel: 'M-HIGH' },
            'jinju': { name: '진주시', temp: 20.1, turbidity: 2.5, chlorophyll: 6.8, trendTemp: '▼ 0.5', trendTurbidity: '▲ 0.1', trendChlorophyll: '정상', riskLevel: 'LOW' },
            'hadong': { name: '하동군', temp: 21.8, turbidity: 2.9, chlorophyll: 10.5, trendTemp: '▲ 0.3', trendTurbidity: '▲ 0.4', trendChlorophyll: '주의', riskLevel: 'MEDIUM' }
        };

        const riskLevelKorean = {
            'HIGH': '매우 높음 위험',
            'M-HIGH': '높음 위험',
            'MEDIUM': '보통 위험',
            'LOW': '낮음 위험'
        };

        // --- Real-time Clock Logic ---
        function updateRealTime() {
            const now = new Date();
            const timeString = now.toLocaleTimeString('ko-KR', { hour12: false });
            const mapTimeRef = document.getElementById('map-time-ref');
            const dataRefreshValue = document.getElementById('data-refresh-value');

            if (mapTimeRef && currentMapViewMode === 'current') {
                mapTimeRef.style.opacity = 0.5;
                mapTimeRef.textContent = `최종 갱신: ${timeString} / 실시간 데이터 기준`;
                setTimeout(() => { mapTimeRef.style.opacity = 1; }, 80); 
            }
            
            // 데이터 갱신 시간 (우측 카드) 업데이트
            if (dataRefreshValue) {
                dataRefreshValue.textContent = timeString;
            }
        }
        
        // 1초마다 실시간 시간 업데이트
        setInterval(updateRealTime, 1000);

        // --- Map Toggle Logic ---
        const riskClassMap = {
            'HIGH': 'bg-red-600 bg-opacity-80 animate-pulse-slow',
            'M-HIGH': 'bg-orange-500 bg-opacity-80',
            'MEDIUM': 'bg-yellow-500 bg-opacity-70',
            'LOW': 'bg-green-500 bg-opacity-70'
        };

        window.toggleMapView = function(mode) {
            currentMapViewMode = mode; // 전역 모드 변수 업데이트
            
            const mapTitle = document.getElementById('map-title');
            const mapTimeRef = document.getElementById('map-time-ref');
            const modeCurrentBtn = document.getElementById('mode-current');
            const modePredictedBtn = document.getElementById('mode-predicted');
            const riskZones = document.querySelectorAll('[data-zone]');

            // 1. Update Title and Reference Text
            if (mode === 'predicted') {
                // 예측 시점을 6시간으로 변경
                mapTitle.textContent = '경남 해역 위험도 (6시간 예측 현황)';
                mapTimeRef.textContent = 'AI 예측 시점: 6시간 후 기준';
                
                // Button Styling
                modeCurrentBtn.classList.remove('bg-blue-500', 'text-white', 'shadow-md');
                modeCurrentBtn.classList.add('bg-gray-200', 'text-gray-700', 'hover:bg-gray-300');
                
                modePredictedBtn.classList.remove('bg-gray-200', 'text-gray-700', 'hover:bg-gray-300');
                modePredictedBtn.classList.add('bg-blue-500', 'text-white', 'shadow-md');
                
            } else { // 'current' mode
                mapTitle.textContent = '경남 해역 위험도 (현재 상황)';
                updateRealTime(); 

                // Button Styling
                modePredictedBtn.classList.remove('bg-blue-500', 'text-white', 'shadow-md');
                modePredictedBtn.classList.add('bg-gray-200', 'text-gray-700', 'hover:bg-gray-300');
                
                modeCurrentBtn.classList.remove('bg-gray-200', 'text-gray-700', 'hover:bg-gray-300');
                modeCurrentBtn.classList.add('bg-blue-500', 'text-white', 'shadow-md');
            }

            // 2. Update Risk Zone Styles and Labels
            riskZones.forEach(zone => {
                // 'data-predicted-level'은 이제 6시간 후 예측 레벨입니다.
                const currentLevel = zone.getAttribute(`data-${mode}-level`);
                const levelDisplay = zone.querySelector('.zone-level');
                
                // Clear existing background classes
                const existingClasses = Object.values(riskClassMap).join(' ');
                zone.classList.remove(...existingClasses.split(' '));
                
                // Apply new background class and pulse if needed
                const newClass = riskClassMap[currentLevel] || 'bg-gray-500 bg-opacity-70';
                const classList = newClass.split(' ');
                zone.classList.add(...classList);
                
                // Update text label
                levelDisplay.textContent = `(${currentLevel})`;

                // Apply pulse only to HIGH risk in the CURRENT view
                if (currentLevel === 'HIGH' && mode === 'current') {
                    zone.classList.add('animate-pulse-slow');
                } else {
                    zone.classList.remove('animate-pulse-slow');
                }
            });
            
            // 3. Force update metrics with the current selected region data
            // This ensures the right-hand side updates if we switch prediction mode
            const selectedElement = document.querySelector(`.map-zone[data-zone="${currentSelectedRegion}"]`);
            if (selectedElement) {
                // If we switch view mode, we update the data based on the *current* state of the map zone
                const currentZoneData = regionData[currentSelectedRegion];
                const riskAttribute = mode === 'current' ? 'data-current-level' : 'data-predicted-level';
                const currentRisk = selectedElement.getAttribute(riskAttribute);

                // Create a temporary data object to pass to updateMetrics
                const displayData = {
                    ...currentZoneData,
                    riskLevel: currentRisk // Use the risk level corresponding to the current map view
                };
                updateMetrics(displayData);
            }
        }
        
        // --- Metric Update Function ---
        function updateMetrics(data) {
            // Update individual metric cards
            document.getElementById('temp-value').textContent = data.temp.toFixed(1);
            // 24H trend remains as a standard metric display
            document.getElementById('temp-trend').textContent = `${data.trendTemp} (24H)`; 
            document.getElementById('turbidity-value').textContent = data.turbidity.toFixed(1);
            document.getElementById('turbidity-trend').textContent = `${data.trendTurbidity} (24H)`;
            document.getElementById('chlorophyll-value').textContent = data.chlorophyll.toFixed(1);
            document.getElementById('chlorophyll-trend').textContent = data.trendChlorophyll;
            
            // Update the Selected Region Status Card
            const riskLevel = data.riskLevel;
            const regionStatusCard = document.getElementById('region-status-card');
            
            document.getElementById('ai-status-badge').textContent = data.name;
            document.getElementById('region-risk-display').textContent = riskLevelKorean[riskLevel];
            
            // Update card border color based on risk level
            regionStatusCard.classList.remove('border-red-500', 'border-orange-500', 'border-yellow-500', 'border-green-500', 'border-blue-500');
            
            if (riskLevel === 'HIGH') {
                document.getElementById('region-risk-display').className = 'text-red-500';
                regionStatusCard.classList.add('border-red-500');
            } else if (riskLevel === 'M-HIGH') {
                document.getElementById('region-risk-display').className = 'text-orange-500';
                regionStatusCard.classList.add('border-orange-500');
            } else if (riskLevel === 'MEDIUM') {
                document.getElementById('region-risk-display').className = 'text-yellow-500';
                regionStatusCard.classList.add('border-yellow-500');
            } else { // LOW
                document.getElementById('region-risk-display').className = 'text-green-500';
                regionStatusCard.classList.add('border-green-500');
            }
        }
        
        // --- Map Zone Click Handler ---
        window.handleZoneClick = function(element, shouldHighlight = true) {
            const zoneKey = element.getAttribute('data-zone');
            
            if (shouldHighlight) {
                // 1. Remove 'selected' class from all zones
                document.querySelectorAll('.map-zone').forEach(zone => {
                    zone.classList.remove('selected');
                });

                // 2. Add 'selected' class to the clicked zone
                element.classList.add('selected');
            
                // 3. Update current selected region tracking
                currentSelectedRegion = zoneKey;
            }
            
            // 4. Get base data for the selected zone
            const baseData = regionData[zoneKey];
            
            // 5. Determine the risk level based on the current map view mode
            const riskAttribute = currentMapViewMode === 'current' ? 'data-current-level' : 'data-predicted-level';
            const currentRisk = element.getAttribute(riskAttribute);

            const displayData = {
                ...baseData,
                riskLevel: currentRisk 
            };
            
            // 6. Update the metric cards
            if (displayData) {
                updateMetrics(displayData);
            } else {
                console.error(`Data not found for zone: ${zoneKey}`);
            }
        }

        // --- Authentication Modal Logic (Mock) ---
        window.showAuthModal = function(id) {
            const modal = document.getElementById(id);
            if (modal) {
                modal.classList.remove('hidden');
                modal.classList.add('flex');
                document.getElementById('auth-message').textContent = '';
                window.switchAuthView('login');
            }
        }

        window.closeAuthModal = function(id) {
            const modal = document.getElementById(id);
            if (modal) {
                modal.classList.add('hidden');
                modal.classList.remove('flex');
            }
        }

        window.switchAuthView = function(view) {
            const loginView = document.getElementById('login-view');
            const signupView = document.getElementById('signup-view');
            const title = document.getElementById('auth-modal-title');
            const tabLogin = document.getElementById('tab-login');
            const tabSignup = document.getElementById('tab-signup');
            
            if (view === 'login') {
                loginView.classList.remove('hidden');
                signupView.classList.add('hidden');
                title.textContent = '로그인';
                
                tabLogin.classList.add('border-blue-600', 'text-blue-600');
                tabLogin.classList.remove('border-transparent', 'text-gray-500', 'hover:text-blue-600', 'hover:border-blue-600');
                
                tabSignup.classList.remove('border-blue-600', 'text-blue-600');
                tabSignup.classList.add('border-transparent', 'text-gray-500', 'hover:text-blue-600', 'hover:border-blue-600');
            } else {
                loginView.classList.add('hidden');
                signupView.classList.remove('hidden');
                title.textContent = '회원가입';
                
                tabSignup.classList.add('border-blue-600', 'text-blue-600');
                tabSignup.classList.remove('border-transparent', 'text-gray-500', 'hover:text-blue-600', 'hover:border-blue-600');
                
                tabLogin.classList.remove('border-blue-600', 'text-blue-600');
                tabLogin.classList.add('border-transparent', 'text-gray-500', 'hover:text-blue-600', 'hover:border-blue-600');
            }
            document.getElementById('auth-message').textContent = ''; // Clear message on switch
        }

        window.handleAuthSubmit = function(event, type) {
            event.preventDefault(); // Prevent form submission
            
            const messageDisplay = document.getElementById('auth-message');
            let email, password;
            
            if (type === 'login') {
                email = document.getElementById('login-email').value;
                password = document.getElementById('login-password').value;
                messageDisplay.textContent = `[로그인 UI 모형] 이메일: ${email}로 로그인 시도... (실제 작동하지 않습니다)`;
                
                // Since this is a mock, we simulate a successful login for the next action
                window.isUserLoggedIn = true; 
            } else if (type === 'signup') {
                email = document.getElementById('signup-email').value;
                password = document.getElementById('signup-password').value;
                messageDisplay.textContent = `[회원가입 UI 모형] 이메일: ${email}로 회원가입 시도... (실제 작동하지 않습니다)`;

                // Simulate successful sign-up and login
                window.isUserLoggedIn = true; 
            }
            
            // Clear form fields after mock submission
            document.getElementById(`${type}-email`).value = '';
            document.getElementById(`${type}-password`).value = '';

            // Automatically close the modal after a short delay for simulated success
            setTimeout(() => {
                closeAuthModal('auth-modal');
                // OPTIONAL: Optionally show the user status update on the header for a moment
                document.getElementById('auth-button').textContent = '로그아웃 (모형)';
                setTimeout(() => {
                     document.getElementById('auth-button').textContent = '로그인 / 회원가입';
                }, 2000); // Reset button text after 2 seconds
            }, 1000);
        }

        // --- Custom Action Modal Logic (Restriction Message) ---
        window.showActionModal = function(title, content) {
            const modal = document.getElementById('action-modal');
            document.getElementById('modal-title').textContent = title;
            document.getElementById('modal-content').textContent = content;
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        }

        // --- DOMContentLoaded Initialization ---
        document.addEventListener('DOMContentLoaded', () => {
            // 1. Initial setting of the map view and time
            window.toggleMapView('current');
            updateRealTime();

            // 2. Initial load of metrics for the default selected region ('sacheon')
            const defaultZoneElement = document.querySelector(`[data-zone="${currentSelectedRegion}"]`);
            if (defaultZoneElement) {
                // Manually trigger click to load default data and highlight
                window.handleZoneClick(defaultZoneElement, true); 
            }

            // 3. Modal logic (Restriction message)
            const ctaButton = document.getElementById('cta-button'); 
            const actionModal = document.getElementById('action-modal'); 
            const closeModalButton = actionModal.querySelector('#close-modal');

            if (ctaButton) {
                ctaButton.addEventListener('click', () => {
                    // Always show the restriction message, as requested by the user.
                    window.showActionModal(
                        '접근 제한 알림',
                        '해당 기능은 회원만 이용 가능합니다. 로그인 또는 회원가입 후 이용해주세요.'
                    );
                });
            }

            if (closeModalButton) {
                closeModalButton.addEventListener('click', () => {
                    actionModal.classList.add('hidden');
                    actionModal.classList.remove('flex');
                });
            }
            
            if (actionModal) {
                actionModal.addEventListener('click', (e) => {
                    if (e.target === actionModal) {
                        actionModal.classList.add('hidden');
                        actionModal.classList.remove('flex');
                    }
                });
            }
        });
    </script>
</body>
</html>
