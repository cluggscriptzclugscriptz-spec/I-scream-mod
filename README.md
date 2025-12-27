<!DOCTYPE html>

<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Freedroid Emulator - The Best Website Emulator</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.5; }
        }
        .animate-pulse {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
        }
    </style>
</head>
<body>
    <div id="app"></div>

```
<script>
    // App State
    let state = {
        isIOS: /iPad|iPhone|iPod/.test(navigator.userAgent),
        showIOSPrompt: false,
        showSurvey: false,
        country: '',
        wantsRecommendations: null,
        currentView: 'home',
        emulatorRunning: false,
        emulatorPoints: 0,
        profile: {
            username: 'User',
            description: 'Freedroid Enthusiast',
            pfp: '👤',
            banner: '#3b82f6'
        },
        editingProfile: false,
        tempProfile: null,
        redeemCode: ''
    };

    const validCodes = {
        'FREEDROID2024': 100,
        'SMOOTH50': 50,
        'WELCOME': 25
    };

    // Initialize
    if (state.isIOS) {
        state.showIOSPrompt = true;
    } else {
        state.showSurvey = true;
    }

    // Icons (using Unicode symbols)
    const icons = {
        smartphone: '📱',
        globe: '🌍',
        gift: '🎁',
        user: '👤',
        download: '⬇️',
        settings: '⚙️',
        home: '🏠',
        chrome: '🌐',
        play: '▶️',
        power: '⚡',
        volume: '🔊',
        rotate: '🔄',
        maximize: '⛶'
    };

    // Render Functions
    function renderIOSPrompt() {
        return `
            <div class="min-h-screen bg-gradient-to-br from-blue-600 to-purple-700 flex items-center justify-center p-4">
                <div class="bg-white rounded-3xl p-8 max-w-md w-full shadow-2xl">
                    <div class="text-center mb-6">
                        <div class="text-6xl mb-4">📱</div>
                        <h1 class="text-3xl font-bold text-gray-900 mb-2">Freedroid Emulator</h1>
                        <p class="text-gray-600">Optimized for iPhone Users</p>
                    </div>
                    <div class="bg-blue-50 rounded-2xl p-6 mb-6">
                        <h2 class="font-semibold text-lg mb-3 text-gray-900">📱 Better Performance</h2>
                        <p class="text-gray-700 mb-4">Add Freedroid to your Home Screen for the smoothest experience possible!</p>
                        <ol class="text-sm text-gray-600 space-y-2">
                            <li>1. Tap the Share button</li>
                            <li>2. Select "Add to Home Screen"</li>
                            <li>3. Enjoy seamless emulation!</li>
                        </ol>
                    </div>
                    <button onclick="handleIOSContinue()" class="w-full bg-blue-600 text-white py-4 rounded-xl font-semibold hover:bg-blue-700 transition">
                        Continue
                    </button>
                </div>
            </div>
        `;
    }

    function renderSurvey() {
        return `
            <div class="min-h-screen bg-gradient-to-br from-blue-600 to-purple-700 flex items-center justify-center p-4">
                <div class="bg-white rounded-3xl p-8 max-w-md w-full shadow-2xl">
                    <div class="text-6xl text-center mb-4">🌍</div>
                    <h2 class="text-2xl font-bold text-center mb-6 text-gray-900">Quick Survey</h2>
                    
                    <div class="mb-6">
                        <label class="block text-sm font-semibold mb-2 text-gray-700">1. What country are you from?</label>
                        <p class="text-xs text-gray-500 mb-3">This helps optimize server regions for faster performance</p>
                        <input type="text" id="countryInput" value="${state.country}" placeholder="Enter your country" class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-blue-500 focus:outline-none">
                    </div>

                    <div class="mb-6">
                        <label class="block text-sm font-semibold mb-3 text-gray-700">2. Would you like personalized recommendations?</label>
                        <div class="flex gap-4">
                            <button onclick="setRecommendations(true)" class="flex-1 py-3 rounded-xl font-semibold transition ${state.wantsRecommendations === true ? 'bg-blue-600 text-white' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'}">
                                Yes
                            </button>
                            <button onclick="setRecommendations(false)" class="flex-1 py-3 rounded-xl font-semibold transition ${state.wantsRecommendations === false ? 'bg-blue-600 text-white' : 'bg-gray-100 text-gray-700 hover:bg-gray-200'}">
                                No
                            </button>
                        </div>
                    </div>

                    <button onclick="handleSurveySubmit()" ${!state.country || state.wantsRecommendations === null ? 'disabled' : ''} class="w-full bg-blue-600 text-white py-4 rounded-xl font-semibold hover:bg-blue-700 transition disabled:opacity-50 disabled:cursor-not-allowed">
                        Start Using Freedroid
                    </button>
                </div>
            </div>
        `;
    }

    function renderEmulator() {
        return `
            <div class="min-h-screen bg-gray-900 flex flex-col">
                <div class="bg-gray-800 border-b border-gray-700 p-4 flex items-center justify-between flex-wrap gap-4">
                    <div class="flex items-center gap-3">
                        <span class="text-2xl">📱</span>
                        <span class="font-semibold text-white text-sm md:text-base">Samsung Galaxy S24 - Android 14</span>
                    </div>
                    <div class="flex items-center gap-3">
                        <span class="text-sm text-green-400 flex items-center gap-1">
                            <span class="inline-block w-2 h-2 bg-green-400 rounded-full animate-pulse"></span>
                            Ultra Smooth ${state.emulatorPoints > 0 ? '(Boosted)' : '(Normal)'}
                        </span>
                        <button onclick="exitEmulator()" class="bg-red-600 text-white px-4 py-2 rounded-lg hover:bg-red-700 transition flex items-center gap-2 text-sm">
                            ⚡ Exit
                        </button>
                    </div>
                </div>

                <div class="flex-1 flex items-center justify-center p-4 md:p-8 overflow-auto">
                    <div class="relative">
                        <div class="w-80 md:w-96 h-[600px] md:h-[700px] bg-black rounded-[3rem] border-8 border-gray-800 shadow-2xl overflow-hidden">
                            <div class="h-8 bg-black flex items-center justify-center">
                                <div class="w-32 h-6 bg-gray-900 rounded-full"></div>
                            </div>
                            
                            <div class="h-full bg-gradient-to-br from-blue-500 to-purple-600 p-4 md:p-6 overflow-hidden">
                                <div class="text-white text-center mb-8">
                                    <div class="text-4xl md:text-6xl mb-2" id="emulatorTime">12:34</div>
                                    <div class="text-base md:text-lg" id="emulatorDate">Friday, December 26</div>
                                </div>

                                <div class="grid grid-cols-4 gap-3 md:gap-4 mt-8 md:mt-16">
                                    ${['Chrome', 'Play Store', 'Settings', 'Camera', 'Gallery', 'Messages', 'Phone', 'Contacts'].map(app => `
                                        <div class="flex flex-col items-center gap-2">
                                            <div class="w-12 h-12 md:w-14 md:h-14 bg-white rounded-2xl shadow-lg flex items-center justify-center text-2xl">
                                                ${app === 'Chrome' ? '🌐' : app === 'Play Store' ? '▶️' : app === 'Settings' ? '⚙️' : '📱'}
                                            </div>
                                            <span class="text-white text-xs">${app}</span>
                                        </div>
                                    `).join('')}
                                </div>

                                <div class="absolute bottom-16 md:bottom-20 left-1/2 transform -translate-x-1/2 bg-white/20 backdrop-blur-lg rounded-full px-4 md:px-6 py-2 md:py-3">
                                    <div class="flex gap-4 md:gap-6 text-xl md:text-2xl">
                                        <span>🏠</span>
                                        <span>🔄</span>
                                        <span>⛶</span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>

                <div class="bg-gray-800 border-t border-gray-700 p-4 text-center text-xs md:text-sm text-gray-400">
                    ⬇️ Open Chrome inside the emulator to download APK files
                </div>
            </div>
        `;
    }

    function renderHome() {
        return `
            <div class="space-y-6">
                <div class="text-center py-8 md:py-12">
                    <h2 class="text-3xl md:text-5xl font-bold bg-gradient-to-r from-blue-600 to-purple-600 bg-clip-text text-transparent mb-4">
                        The Best Website Emulator
                    </h2>
                    <p class="text-lg md:text-xl text-gray-600 mb-2">Everything is crazy smooth ✨</p>
                    <p class="text-sm md:text-base text-gray-500">Latest Android 14 & Samsung Galaxy S24</p>
                </div>

                <div class="grid md:grid-cols-3 gap-6 mb-8">
                    <div class="bg-gradient-to-br from-blue-50 to-blue-100 rounded-2xl p-6 text-center">
                        <div class="text-4xl mb-3">📱</div>
                        <h3 class="font-semibold text-lg mb-2">Full Android Features</h3>
                        <p class="text-sm text-gray-600">All features included, nothing missing</p>
                    </div>
                    <div class="bg-gradient-to-br from-purple-50 to-purple-100 rounded-2xl p-6 text-center">
                        <div class="text-4xl mb-3">⬇️</div>
                        <h3 class="font-semibold text-lg mb-2">Download APKs</h3>
                        <p class="text-sm text-gray-600">Install any app from Chrome</p>
                    </div>
                    <div class="bg-gradient-to-br from-pink-50 to-pink-100 rounded-2xl p-6 text-center">
                        <div class="text-4xl mb-3">⚙️</div>
                        <h3 class="font-semibold text-lg mb-2">Zero Lag</h3>
                        <p class="text-sm text-gray-600">Optimized for your region</p>
                    </div>
                </div>

                <div class="bg-gradient-to-r from-blue-600 to-purple-600 rounded-2xl p-8 text-white text-center">
                    <h3 class="text-2xl font-bold mb-4">Start Emulator</h3>
                    <p class="mb-6 text-blue-100">Launch your ultra-smooth Android experience</p>
                    <button onclick="startEmulator()" class="bg-white text-blue-600 px-8 py-4 rounded-xl font-bold text-lg hover:bg-blue-50 transition transform hover:scale-105">
                        ▶️ Launch Emulator
                    </button>
                </div>
            </div>
        `;
    }

    function renderCodes() {
        return `
            <div class="max-w-2xl mx-auto">
                <div class="text-center mb-8">
                    <div class="text-6xl mb-4">🎁</div>
                    <h2 class="text-3xl font-bold mb-2">Redeem Codes</h2>
                    <p class="text-gray-600">Get Emulator Points for smoother performance!</p>
                </div>

                <div class="bg-gradient-to-br from-purple-50 to-pink-50 rounded-2xl p-8 mb-6">
                    <div class="flex flex-col md:flex-row gap-3 mb-4">
                        <input type="text" id="redeemCodeInput" value="${state.redeemCode}" placeholder="Enter code here" class="flex-1 px-6 py-4 border-2 border-purple-200 rounded-xl focus:border-purple-500 focus:outline-none text-lg font-mono uppercase">
                        <button onclick="handleRedeemCode()" class="bg-gradient-to-r from-purple-600 to-pink-600 text-white px-8 py-4 rounded-xl font-semibold hover:opacity-90 transition">
                            Redeem
                        </button>
                    </div>
                    <p class="text-sm text-gray-600 text-center">Codes give you points for enhanced performance</p>
                </div>

                <div class="bg-white border-2 border-gray-200 rounded-2xl p-6">
                    <h3 class="font-semibold text-lg mb-4">Try these codes:</h3>
                    <div class="space-y-3">
                        <div class="flex justify-between items-center p-4 bg-gray-50 rounded-xl">
                            <code class="font-mono font-bold text-purple-600">FREEDROID2024</code>
                            <span class="text-sm text-gray-600">+100 points</span>
                        </div>
                        <div class="flex justify-between items-center p-4 bg-gray-50 rounded-xl">
                            <code class="font-mono font-bold text-purple-600">SMOOTH50</code>
                            <span class="text-sm text-gray-600">+50 points</span>
                        </div>
                        <div class="flex justify-between items-center p-4 bg-gray-50 rounded-xl">
                            <code class="font-mono font-bold text-purple-600">WELCOME</code>
                            <span class="text-sm text-gray-600">+25 points</span>
                        </div>
                    </div>
                </div>
            </div>
        `;
    }

    function renderProfile() {
        if (state.editingProfile) {
            return `
                <div class="max-w-2xl mx-auto space-y-6">
                    <h2 class="text-2xl font-bold">Edit Profile</h2>
                    
                    <div>
                        <label class="block text-sm font-semibold mb-2">Banner Color</label>
                        <input type="color" id="bannerColor" value="${state.tempProfile.banner}" onchange="updateTempProfile('banner', this.value)" class="w-full h-12 rounded-xl cursor-pointer">
                    </div>

                    <div>
                        <label class="block text-sm font-semibold mb-2">Profile Picture (Emoji)</label>
                        <input type="text" id="pfpInput" value="${state.tempProfile.pfp}" oninput="updateTempProfile('pfp', this.value)" maxlength="2" class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-blue-500 focus:outline-none text-2xl text-center">
                    </div>

                    <div>
                        <label class="block text-sm font-semibold mb-2">Username</label>
                        <input type="text" id="usernameInput" value="${state.tempProfile.username}" oninput="updateTempProfile('username', this.value)" class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-blue-500 focus:outline-none">
                    </div>

                    <div>
                        <label class="block text-sm font-semibold mb-2">Description</label>
                        <textarea id="descInput" oninput="updateTempProfile('description', this.value)" class="w-full px-4 py-3 border-2 border-gray-200 rounded-xl focus:border-blue-500 focus:outline-none" rows="3">${state.tempProfile.description}</textarea>
                    </div>

                    <div class="flex gap-3">
                        <button onclick="saveProfile()" class="flex-1 bg-blue-600 text-white py-3 rounded-xl font-semibold hover:bg-blue-700 transition">
                            Save Changes
                        </button>
                        <button onclick="cancelEdit()" class="flex-1 bg-gray-200 text-gray-700 py-3 rounded-xl font-semibold hover:bg-gray-300 transition">
                            Cancel
                        </button>
                    </div>
                </div>
            `;
        }

        return `
            <div class="max-w-2xl mx-auto">
                <div class="rounded-2xl overflow-hidden mb-6 shadow-lg">
                    <div class="h-32" style="background-color: ${state.profile.banner}"></div>
                    <div class="bg-white p-6 relative">
                        <div class="absolute -top-12 left-6">
                            <div class="w-24 h-24 bg-white rounded-full flex items-center justify-center text-5xl border-4 border-white shadow-lg">
                                ${state.profile.pfp}
                            </div>
                        </div>
                        <div class="ml-32">
                            <h2 class="text-2xl font-bold mb-1">${state.profile.username}</h2>
                            <p class="text-gray-600">${state.profile.description}</p>
                        </div>
                    </div>
                </div>

                <div class="bg-gradient-to-br from-blue-50 to-purple-50 rounded-2xl p-6 mb-4">
                    <div class="grid grid-cols-2 gap-4 text-center">
                        <div>
                            <div class="text-3xl font-bold text-blue-600">${state.emulatorPoints}</div>
                            <div class="text-sm text-gray-600">Emulator Points</div>
                        </div>
                        <div>
                            <div class="text-3xl font-bold text-purple-600">${state.country || 'Global'}</div>
                            <div class="text-sm text-gray-600">Region</div>
                        </div>
                    </div>
                </div>

                <button onclick="editProfile()" class="w-full bg-gradient-to-r from-blue-600 to-purple-600 text-white py-4 rounded-xl font-semibold hover:opacity-90 transition">
                    Edit Profile
                </button>
            </div>
        `;
    }

    function renderMain() {
        return `
            <div class="min-h-screen bg-gradient-to-br from-blue-600 via-purple-600 to-pink-600">
                <div class="max-w-6xl mx-auto p-4 md:p-6">
                    <div class="bg-white rounded-3xl shadow-2xl overflow-hidden">
                        <div class="bg-gradient-to-r from-blue-600 to-purple-600 p-6 md:p-8 text-white">
                            <div class="flex flex-col md:flex-row items-start md:items-center justify-between gap-4">
                                <div>
                                    <h1 class="text-3xl md:text-4xl font-bold mb-2">Freedroid Emulator</h1>
                                    <p class="text-blue-100 text-sm md:text-base">The Best Website Emulator - Ultra Smooth Performance</p>
                                </div>
                                <div class="text-left md:text-right">
                                    <div class="text-sm text-blue-100">Emulator Points</div>
                                    <div class="text-3xl font-bold">${state.emulatorPoints}</div>
                                </div>
                            </div>
                        </div>

                        <div class="flex border-b border-gray-200">
                            <button onclick="setView('home')" class="flex-1 py-4 font-semibold transition text-sm md:text-base ${state.currentView === 'home' ? 'bg-blue-50 text-blue-600 border-b-2 border-blue-600' : 'text-gray-600 hover:bg-gray-50'}">
                                🏠 Home
                            </button>
                            <button onclick="setView('codes')" class="flex-1 py-4 font-semibold transition text-sm md:text-base ${state.currentView === 'codes' ? 'bg-blue-50 text-blue-600 border-b-2 border-blue-600' : 'text-gray-600 hover:bg-gray-50'}">
                                🎁 Codes
                            </button>
                            <button onclick="setView('profile')" class="flex-1 py-4 font-semibold transition text-sm md:text-base ${state.currentView === 'profile' ? 'bg-blue-50 text-blue-600 border-b-2 border-blue-600' : 'text-gray-600 hover:bg-gray-50'}">
                                👤 Profile
                            </button>
                        </div>

                        <div class="p-4 md:p-8">
                            ${state.currentView === 'home' ? renderHome() : state.currentView === 'codes' ? renderCodes() : renderProfile()}
                        </div>
                    </div>

                    <div class="text-center mt-6 text-white text-sm">
                        <p>© 2024 Freedroid Emulator - 100% Free Forever</p>
                    </div>
                </div>
            </div>
        `;
    }

    function render() {
        const app = document.getElementById('app');
        if (state.showIOSPrompt) {
            app.innerHTML = renderIOSPrompt();
        } else if (state.showSurvey) {
            app.innerHTML = renderSurvey();
        } else if (state.emulatorRunning) {
            app.innerHTML = renderEmulator();
            updateEmulatorTime();
        } else {
            app.innerHTML = renderMain();
        }
    }

    // Event Handlers
    function handleIOSContinue() {
        state.showIOSPrompt = false;
        state.showSurvey = true;
        render();
    }

    function setRecommendations(value) {
        state.wantsRecommendations = value;
        render();
    }

    function handleSurveySubmit() {
        const countryInput = document.getElementById('countryInput');
        state.country = countryInput.value;
        if (state.country && state.wantsRecommendations !== null) {
            state.showSurvey = false;
            state.currentView = 'home';
            render();
        }
    }

    function setView(view) {
        state.currentView = view;
        render();
    }

    function startEmulator() {
        state.emulatorRunning = true;
        render();
    }

    function exitEmulator() {
        state.emulatorRunning = false;
        render();
    }

    function handleRedeemCode() {
        const input = document.getElementById('redeemCodeInput');
        const code = input.value.toUpperCase();
        if (validCodes[code]) {
            state.emulatorPoints += validCodes[code];
            alert(`Success! You earned ${validCodes[code]} Emulator Points!`);
            state.redeemCode = '';
            render();
        } else {
            alert('Invalid code. Try again!');
        }
    }

    function editProfile() {
        state.tempProfile = {...state.profile};
        state.editingProfile = true;
        render();
    }

    function updateTempProfile(field, value) {
        state.tempProfile[field] = value;
    }

    function saveProfile() {
        state.profile = {...state.tempProfile};
        state.editingProfile = false;
        render();
    }

    function cancelEdit() {
        state.editingProfile = false;
        render();
    }

    function updateEmulatorTime() {
        const timeEl = document.getElementById('emulatorTime');
        const dateEl = document.getElementById('emulatorDate');
        if (timeEl && dateEl) {
            const now = new Date();
            timeEl.textContent = now.toLocaleTimeString('en-US', {hour: '2-digit', minute: '2-digit'});
            dateEl.textContent = now.toLocaleDateString('en-US', {weekday: 'long', month: 'long', day: 'numeric'});
            setTimeout(updateEmulatorTime, 1000);
        }
    }

    // Initial render
    render();
</script>
```

</body>
</html>
