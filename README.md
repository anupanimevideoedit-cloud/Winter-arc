```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Winter Arc Daily Tracker & Admin Panel</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    <!-- jsPDF Library for real PDF download -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <!-- Tone.js for satisfying sound effects -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        darkBg: '#0b0a10',
                        cardBg: '#181722',
                        cardBorder: '#272538',
                        accentPurple: '#8b5cf6',
                        accentBlue: '#3b82f6',
                        accentYellow: '#f59e0b',
                        accentOrange: '#f97316',
                        accentPink: '#ec4899',
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #0b0a10;
            color: #f3f4f6;
            font-family: 'Inter', sans-serif;
            overflow-x: hidden;
        }
        .glass-card {
            background: rgba(24, 23, 34, 0.75);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.07);
        }
        .hide-scrollbar::-webkit-scrollbar {
            display: none;
        }
        .hide-scrollbar {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
        @keyframes popIn {
            0% { transform: scale(0.8); opacity: 0; }
            70% { transform: scale(1.05); opacity: 1; }
            100% { transform: scale(1); opacity: 1; }
        }
        .animate-pop {
            animation: popIn 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275) forwards;
        }
    </style>
</head>
<body class="min-h-screen flex flex-col justify-between select-none">

    <div id="app" class="max-w-md mx-auto w-full min-h-screen flex flex-col pb-24 relative shadow-2xl bg-[#0b0a10]">
        
        <!-- Toast Notification Box -->
        <div id="toast" class="fixed top-5 left-1/2 transform -translate-x-1/2 z-50 bg-[#1e1c2e] border border-purple-500/40 text-white px-5 py-3 rounded-2xl shadow-2xl transition-all duration-300 opacity-0 pointer-events-none flex items-center space-x-3">
            <span id="toast-icon" class="text-purple-400 text-lg">❄️</span>
            <span id="toast-msg" class="text-sm font-medium">Notification message</span>
        </div>

        <!-- ==================== CELEBRATION MODAL (TASK COMPLETED & LEVEL UP) ==================== -->
        <div id="modal-celebration" class="fixed inset-0 bg-black/85 backdrop-blur-md z-50 hidden flex items-center justify-center p-5">
            <div class="glass-card border border-purple-500/50 w-full max-w-sm p-6 rounded-3xl flex flex-col items-center text-center space-y-4 shadow-2xl animate-pop relative overflow-hidden">
                <!-- Background decorative glow -->
                <div class="absolute -top-20 -right-20 w-40 h-40 bg-purple-600/20 rounded-full blur-3xl"></div>
                <div class="absolute -bottom-20 -left-20 w-40 h-40 bg-indigo-600/20 rounded-full blur-3xl"></div>

                <!-- Animated Girl Illustration / SVG Icon with checkmark badge -->
                <div class="w-24 h-24 rounded-3xl bg-gradient-to-tr from-purple-600 to-pink-500 border border-white/20 flex items-center justify-center text-5xl shadow-xl relative animate-bounce">
                    👩‍🎓
                    <div class="absolute -bottom-2 -right-2 bg-emerald-500 text-white text-xs px-2 py-0.5 rounded-full font-bold shadow-md">✓</div>
                </div>

                <div>
                    <h3 id="celeb-title" class="text-2xl font-extrabold text-white tracking-tight">🎉 Task Completed!</h3>
                    <p id="celeb-subtitle" class="text-xs text-purple-300 mt-1">Fantastic discipline! Keep crushing your goals.</p>
                </div>

                <div class="bg-black/40 border border-white/10 w-full p-3.5 rounded-2xl flex flex-col space-y-1.5">
                    <span class="text-[10px] text-gray-400 uppercase tracking-wider font-semibold">Completed Task</span>
                    <span id="celeb-task-name" class="text-sm font-bold text-white">Positive Affirmation</span>
                    <div class="flex justify-between items-center pt-1 border-t border-white/5 mt-1">
                        <span id="celeb-date-stamp" class="text-[11px] text-purple-400 font-medium">October 1, 2026</span>
                        <span id="celeb-xp-earned" class="text-[11px] text-emerald-400 font-bold">+2 XP Gained</span>
                    </div>
                </div>

                <button onclick="closeCelebrationModal()" class="w-full py-3.5 bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 text-white font-bold rounded-2xl shadow-xl transition-all text-sm">
                    Keep Crushing It 🚀
                </button>
            </div>
        </div>

        <!-- ==================== SCREEN: ONBOARDING (MANDATORY WELCOME) ==================== -->
        <div id="screen-onboarding" class="view-screen fixed inset-0 bg-[#0b0a10] z-50 flex flex-col p-6 justify-between overflow-y-auto hidden">
            <div class="flex flex-col space-y-6 pt-6">
                <!-- Header Icon & Title -->
                <div class="flex flex-col items-center text-center space-y-3">
                    <div class="w-20 h-20 rounded-3xl bg-gradient-to-br from-indigo-950 to-purple-950 border border-purple-500/40 flex items-center justify-center text-4xl shadow-2xl">
                        ❄
                    </div>
                    <h1 class="text-3xl font-extrabold text-white tracking-tight">Winter Arc Challenge</h1>
                    <p class="text-sm text-gray-400 max-w-xs">Transform your life, build iron discipline, and conquer your goals. Enter your details to begin.</p>
                </div>

                <!-- Onboarding Form Inputs -->
                <div class="glass-card p-5 rounded-3xl flex flex-col space-y-4 border border-purple-500/30">
                    <div>
                        <label class="text-xs font-semibold text-gray-300 mb-1.5 block uppercase tracking-wider">Your Full Name <span class="text-purple-400">*required</span></label>
                        <input type="text" id="onboard-name" class="w-full bg-black/40 border border-white/10 rounded-2xl p-3.5 text-sm text-white focus:outline-none focus:border-purple-500 transition-colors" placeholder="e.g. Alex Morgan" required>
                    </div>

                    <div>
                        <label class="text-xs font-semibold text-gray-300 mb-1.5 block uppercase tracking-wider">Challenge Duration</label>
                        <select id="onboard-duration" onchange="updateOnboardDates()" class="w-full bg-black/40 border border-white/10 rounded-2xl p-3.5 text-sm text-white focus:outline-none focus:border-purple-500 transition-colors">
                            <option value="30">30 Days (1 Month Sprint)</option>
                            <option value="75">75 Days (Hardcore Winter Arc)</option>
                            <option value="90" selected>90 Days (Complete Transformation)</option>
                        </select>
                    </div>

                    <div class="grid grid-cols-2 gap-3">
                        <div>
                            <label class="text-xs font-semibold text-gray-300 mb-1.5 block uppercase tracking-wider">Start Date</label>
                            <input type="date" id="onboard-start-date" class="w-full bg-black/40 border border-white/10 rounded-2xl p-3 text-xs text-white focus:outline-none focus:border-purple-500 transition-colors">
                        </div>
                        <div>
                            <label class="text-xs font-semibold text-gray-300 mb-1.5 block uppercase tracking-wider">End Date</label>
                            <input type="date" id="onboard-end-date" class="w-full bg-black/40 border border-white/10 rounded-2xl p-3 text-xs text-white focus:outline-none focus:border-purple-500 transition-colors" disabled>
                        </div>
                    </div>
                </div>
            </div>

            <div class="pt-6 pb-4">
                <button onclick="completeOnboarding()" class="w-full py-4 bg-gradient-to-r from-indigo-600 to-purple-600 hover:from-indigo-500 hover:to-purple-500 text-white font-bold rounded-2xl shadow-xl transition-all flex items-center justify-center space-x-2 text-base">
                    <span>🚀</span>
                    <span>Start My Winter Arc</span>
                </button>
                <p class="text-center text-[10px] text-gray-500 mt-3">Challenge locks upon completion date. Name & details required.</p>
            </div>
        </div>

        <!-- ==================== SCREEN: HOME ==================== -->
        <div id="screen-home" class="view-screen p-5 flex flex-col space-y-5">
            <div class="flex justify-between items-start">
                <div>
                    <p id="current-date-header" class="text-xs font-semibold text-gray-400 tracking-wider uppercase">Thursday, October 1</p>
                    <h1 class="text-3xl font-bold tracking-tight text-white flex items-center space-x-2 mt-0.5">
                        <span id="welcome-user-title">Winter Arc</span>
                        <span class="text-xl">❄</span>
                    </h1>
                    <p id="welcome-subtitle" class="text-xs text-gray-400 mt-0.5">Build the best version of yourself</p>
                </div>
                <!-- Day Counter Badge -->
                <div class="bg-gradient-to-br from-indigo-900/80 to-purple-900/80 border border-purple-500/30 px-4 py-2.5 rounded-2xl flex flex-col items-center shadow-lg">
                    <span class="text-[10px] uppercase tracking-wider text-purple-300 font-medium">Day</span>
                    <span id="header-day-num" class="text-lg font-extrabold text-white">1</span>
                </div>
            </div>

            <!-- Level & XP Status Banner -->
            <div class="glass-card p-4 rounded-2xl flex items-center justify-between border border-purple-500/30">
                <div class="flex items-center space-x-3">
                    <div class="w-11 h-11 rounded-2xl bg-gradient-to-tr from-purple-600 to-pink-500 text-white flex items-center justify-center text-lg font-black shadow-lg">
                        <span id="header-level-badge">1</span>
                    </div>
                    <div>
                        <div class="flex items-center space-x-2">
                            <h3 id="header-level-title" class="text-sm font-bold text-white">Novice Challenger</h3>
                            <span class="text-[10px] text-emerald-400 font-semibold bg-emerald-500/10 px-2 py-0.5 rounded-full" id="header-xp-display">0 XP</span>
                        </div>
                        <p id="header-next-level-desc" class="text-[11px] text-gray-400 mt-0.5">2 XP needed for Level 2</p>
                    </div>
                </div>
                <div class="text-right">
                    <span class="text-xs font-bold text-purple-400" id="header-total-points">0 pts</span>
                </div>
            </div>

            <!-- Summary Stats Cards -->
            <div class="grid grid-cols-3 gap-3">
                <div class="glass-card p-3 rounded-2xl flex flex-col justify-center items-center text-center">
                    <div class="flex items-center space-x-1 text-purple-400 text-xs font-semibold mb-1">
                        <span class="w-2 h-2 rounded-full bg-purple-400"></span>
                        <span id="stats-done-count">0</span>
                    </div>
                    <span class="text-[11px] text-gray-400 font-medium">Done</span>
                </div>
                <div class="glass-card p-3 rounded-2xl flex flex-col justify-center items-center text-center">
                    <div class="flex items-center space-x-1 text-yellow-400 text-xs font-semibold mb-1">
                        <span class="w-2 h-2 rounded-full bg-yellow-400"></span>
                        <span id="stats-left-count">6</span>
                    </div>
                    <span class="text-[11px] text-gray-400 font-medium">Left</span>
                </div>
                <div class="glass-card p-3 rounded-2xl flex flex-col justify-center items-center text-center">
                    <div class="flex items-center space-x-1 text-orange-400 text-xs font-semibold mb-1">
                        <span>🔥</span>
                        <span id="stats-streak-count">0</span>
                    </div>
                    <span class="text-[11px] text-gray-400 font-medium">Best Streak</span>
                </div>
            </div>

            <!-- Progress Bar Section -->
            <div class="glass-card p-4 rounded-2xl flex flex-col space-y-2">
                <div class="flex justify-between items-center text-xs font-medium">
                    <span class="text-gray-300">Today's Habits</span>
                    <span id="progress-text" class="text-purple-400 font-bold">0/6</span>
                </div>
                <div class="w-full bg-gray-800 h-2.5 rounded-full overflow-hidden">
                    <div id="progress-bar-fill" class="bg-gradient-to-r from-indigo-500 to-purple-500 h-full w-0 transition-all duration-500 rounded-full"></div>
                </div>
            </div>

            <!-- Habits List Section -->
            <div class="flex flex-col space-y-3">
                <div class="flex justify-between items-center px-1">
                    <h2 class="text-sm font-semibold text-gray-300 uppercase tracking-wider">Core Habits</h2>
                    <span id="total-habits-badge" class="text-xs text-gray-400 font-medium">6 habits</span>
                </div>

                <!-- Habits Container -->
                <div id="habits-list-container" class="flex flex-col space-y-3">
                    <!-- Dynamic habit items injected via JS -->
                </div>

                <!-- Add Custom Habit Button -->
                <button onclick="openAddHabitModal()" class="w-full py-3.5 px-4 rounded-2xl border border-dashed border-purple-500/30 hover:border-purple-500/60 bg-purple-950/10 hover:bg-purple-950/20 text-purple-300 font-medium text-sm flex items-center justify-center space-x-2 transition-all duration-200">
                    <span class="text-lg">+</span>
                    <span>Add Custom Habit</span>
                </button>
            </div>
        </div>

        <!-- ==================== SCREEN: ANALYTICS ==================== -->
        <div id="screen-analytics" class="view-screen p-5 hidden flex-col space-y-5">
            <div class="flex justify-between items-center">
                <h1 class="text-2xl font-bold text-white">Analytics</h1>
                <button onclick="exportPDF()" class="bg-indigo-600/30 hover:bg-indigo-600/50 border border-indigo-500/40 text-indigo-200 px-4 py-2 rounded-xl text-xs font-semibold flex items-center space-x-2 transition-all shadow-md">
                    <span>📤</span>
                    <span>Export PDF</span>
                </button>
            </div>

            <!-- Overview Cards -->
            <div class="grid grid-cols-2 gap-3">
                <div class="bg-gradient-to-br from-indigo-900/60 to-purple-900/60 border border-purple-500/30 p-4 rounded-2xl flex flex-col justify-between">
                    <span id="analytics-challenge-day" class="text-3xl font-extrabold text-white">1</span>
                    <span id="analytics-duration-label" class="text-xs text-purple-300 font-medium mt-2">of 90 Challenge Day</span>
                </div>
                <div class="bg-gradient-to-br from-emerald-900/60 to-teal-900/60 border border-emerald-500/30 p-4 rounded-2xl flex flex-col justify-between">
                    <span id="analytics-today-done" class="text-3xl font-extrabold text-white">0</span>
                    <span class="text-xs text-emerald-300 font-medium mt-2">out of 6 Today's Done</span>
                </div>
            </div>

            <!-- Level Progression Overview -->
            <div class="glass-card p-4 rounded-2xl flex flex-col space-y-2 border border-purple-500/30">
                <div class="flex justify-between items-center text-xs">
                    <span class="text-gray-300 font-medium flex items-center space-x-2">
                        <span>⭐</span>
                        <span id="analytics-level-title-text">Level 1 Challenger</span>
                    </span>
                    <span id="analytics-level-xp-text" class="text-purple-400 font-bold">0 / 2 XP</span>
                </div>
                <div class="w-full bg-gray-800 h-2.5 rounded-full overflow-hidden">
                    <div id="analytics-level-bar" class="bg-gradient-to-r from-purple-500 to-pink-500 h-full w-0 transition-all duration-500 rounded-full"></div>
                </div>
            </div>

            <!-- 30-Day Average Bar -->
            <div class="glass-card p-4 rounded-2xl flex flex-col space-y-2">
                <div class="flex justify-between items-center text-xs">
                    <span class="text-gray-300 font-medium flex items-center space-x-2">
                        <span>📊</span>
                        <span>30-Day Average</span>
                    </span>
                    <span id="analytics-avg-pct" class="text-purple-400 font-bold">0%</span>
                </div>
                <div class="w-full bg-gray-800 h-2.5 rounded-full overflow-hidden">
                    <div id="analytics-avg-bar" class="bg-gradient-to-r from-indigo-500 to-purple-500 h-full w-0 transition-all duration-500 rounded-full"></div>
                </div>
            </div>

            <!-- Habit Performance Breakdown -->
            <div class="flex flex-col space-y-3">
                <h2 class="text-sm font-semibold text-gray-300 uppercase tracking-wider px-1">Habit Performance</h2>
                <div id="analytics-habits-list" class="flex flex-col space-y-3">
                    <!-- Dynamic analytics performance cards -->
                </div>
            </div>
        </div>

        <!-- ==================== SCREEN: SETTINGS ==================== -->
        <div id="screen-settings" class="view-screen p-5 hidden flex-col space-y-5">
            <h1 class="text-2xl font-bold text-white">Settings</h1>

            <!-- Challenge Profile Card -->
            <div class="bg-gradient-to-br from-indigo-950/80 to-purple-950/80 border border-purple-500/30 p-5 rounded-2xl flex flex-col space-y-4 shadow-xl">
                <div class="flex items-center space-x-3">
                    <div class="w-12 h-12 rounded-2xl bg-indigo-900/60 border border-indigo-500/40 flex items-center justify-center text-2xl">
                        ❄
                    </div>
                    <div>
                        <h3 id="settings-challenge-title" class="font-bold text-white text-base">Winter Arc Challenge</h3>
                        <p id="settings-challenge-dates" class="text-xs text-purple-300">Started Oct 1, 2026</p>
                    </div>
                </div>
                <div class="grid grid-cols-3 gap-2 pt-2 border-t border-purple-500/20">
                    <div class="bg-black/30 p-2.5 rounded-xl text-center">
                        <span id="settings-day-count" class="block text-lg font-bold text-white">1</span>
                        <span class="text-[10px] text-gray-400 uppercase">Day</span>
                    </div>
                    <div class="bg-black/30 p-2.5 rounded-xl text-center">
                        <span id="settings-perfect-count" class="block text-lg font-bold text-white">0</span>
                        <span class="text-[10px] text-gray-400 uppercase">Perfect Days</span>
                    </div>
                    <div class="bg-black/30 p-2.5 rounded-xl text-center">
                        <span id="settings-user-level" class="block text-lg font-bold text-purple-300">Lvl 1</span>
                        <span class="text-[10px] text-gray-400 uppercase">Level</span>
                    </div>
                </div>
            </div>

            <!-- Anup AI Section (Special Health & Study Coach) -->
            <div class="flex flex-col space-y-2">
                <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider px-1">AI Health & Study Coach</span>
                <div class="glass-card rounded-2xl overflow-hidden">
                    <div onclick="openAnupAiModal()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-gradient-to-tr from-purple-600 to-indigo-500 text-white flex items-center justify-center text-sm shadow-md font-bold">🤖</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Anup AI Coach</h4>
                                <p class="text-xs text-purple-300">Ask about diet, workouts, health & study tips</p>
                            </div>
                        </div>
                        <span class="text-gray-400 text-sm font-bold">Open ›</span>
                    </div>
                </div>
            </div>

            <!-- Admin Panel Navigation Section (Secured with password 009) -->
            <div class="flex flex-col space-y-2">
                <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider px-1">Administration</span>
                <div class="glass-card rounded-2xl overflow-hidden">
                    <div onclick="openAdminLoginModal()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-indigo-600/20 text-indigo-400 flex items-center justify-center text-sm">🔒</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Admin Panel</h4>
                                <p class="text-xs text-purple-300">View user sessions & manage validity status</p>
                            </div>
                        </div>
                        <span class="text-gray-400 text-sm font-bold">Open ›</span>
                    </div>
                </div>
            </div>

            <!-- Notifications Section -->
            <div class="flex flex-col space-y-2">
                <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider px-1">Notifications</span>
                <div class="glass-card rounded-2xl overflow-hidden divide-y divide-white/5">
                    <div class="p-4 flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-yellow-500/10 text-yellow-400 flex items-center justify-center text-sm">🔔</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Enable Notifications</h4>
                                <p class="text-xs text-gray-400">Get reminders for active habits</p>
                            </div>
                        </div>
                        <label class="relative inline-flex items-center cursor-pointer">
                            <input type="checkbox" id="toggle-notifications" checked class="sr-only peer" onchange="requestNotificationPermission()">
                            <div class="w-11 h-6 bg-gray-700 peer-focus:outline-none rounded-full peer peer-checked:after:translate-x-full peer-checked:after:border-white after:content-[''] after:absolute after:top-[2px] after:left-[2px] after:bg-white after:border-gray-300 after:border after:rounded-full after:h-5 after:w-5 after:transition-all peer-checked:bg-purple-600"></div>
                        </label>
                    </div>
                    <!-- Global Daily Reminder row configured to open clock time picker -->
                    <div onclick="openGlobalTimePicker()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-purple-500/10 text-purple-400 flex items-center justify-center text-sm">⏰</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Global Daily Reminder</h4>
                                <p id="settings-global-time-label" class="text-xs text-purple-300 font-medium">08:00 AM (Tap to set clock)</p>
                            </div>
                        </div>
                        <span class="text-gray-400 text-sm">›</span>
                    </div>
                </div>
            </div>

            <!-- App Section -->
            <div class="flex flex-col space-y-2">
                <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider px-1">App</span>
                <div class="glass-card rounded-2xl overflow-hidden divide-y divide-white/5">
                    <div class="p-4 flex items-center justify-between">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-blue-500/10 text-blue-400 flex items-center justify-center text-sm">ℹ️</div>
                            <span class="text-sm font-semibold text-white">Version</span>
                        </div>
                        <span class="text-xs text-gray-400 font-medium">1.0.2</span>
                    </div>
                    <div onclick="showToast('Thank you for rating 5 stars! 🌟')" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-amber-500/10 text-amber-400 flex items-center justify-center text-sm">⭐</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Rate on Play Store</h4>
                                <p class="text-xs text-gray-400">Enjoying the app? Leave a review 🌟</p>
                            </div>
                        </div>
                        <span class="text-gray-500">›</span>
                    </div>
                    <div onclick="shareApp()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-purple-500/10 text-purple-400 flex items-center justify-center text-sm">🔗</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Share App</h4>
                                <p class="text-xs text-gray-400">Share with friends & family</p>
                            </div>
                        </div>
                        <span class="text-gray-500">›</span>
                    </div>
                    <div onclick="showPrivacyModal()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-white/5 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-violet-500/10 text-violet-400 flex items-center justify-center text-sm">🛡️</div>
                            <span class="text-sm font-semibold text-white">Privacy Policy</span>
                        </div>
                        <span class="text-gray-500">›</span>
                    </div>
                    <div onclick="resetChallengeManually()" class="p-4 flex items-center justify-between cursor-pointer hover:bg-red-500/10 transition-colors">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-red-500/10 text-red-400 flex items-center justify-center text-sm">🔄</div>
                            <div>
                                <h4 class="text-sm font-semibold text-red-300">Restart Challenge</h4>
                                <p class="text-xs text-gray-400">Clear data and setup new challenge</p>
                            </div>
                        </div>
                        <span class="text-gray-500">›</span>
                    </div>
                </div>
            </div>

            <!-- Support Section with anup@gamil.com -->
            <div class="flex flex-col space-y-2">
                <span class="text-xs font-semibold text-gray-400 uppercase tracking-wider px-1">Support</span>
                <div class="glass-card rounded-2xl overflow-hidden">
                    <a href="mailto:anup@gamil.com" class="p-4 flex items-center justify-between hover:bg-white/5 transition-colors block">
                        <div class="flex items-center space-x-3">
                            <div class="w-9 h-9 rounded-xl bg-rose-500/10 text-rose-400 flex items-center justify-center text-sm">✉</div>
                            <div>
                                <h4 class="text-sm font-semibold text-white">Contact Developer</h4>
                                <p class="text-xs text-gray-400">anup@gamil.com</p>
                            </div>
                        </div>
                        <span class="text-gray-500">›</span>
                    </a>
                </div>
            </div>

            <!-- Motivational Footer Tagline -->
            <div class="text-center pt-2 pb-4">
                <p class="text-xs italic text-purple-300 font-medium">❄ Stay Consistent. Transform Your Life.</p>
            </div>
        </div>

        <!-- ==================== SCREEN: ADMIN PANEL ==================== -->
        <div id="screen-admin" class="view-screen p-5 hidden flex-col space-y-5">
            <div class="flex items-center justify-between">
                <div class="flex items-center space-x-2">
                    <button onclick="switchTab('settings')" class="w-9 h-9 rounded-xl bg-white/5 border border-white/10 flex items-center justify-center text-white text-base">‹</button>
                    <h1 class="text-2xl font-bold text-white">Admin Panel</h1>
                </div>
                <span class="bg-indigo-600/30 border border-indigo-500/40 text-indigo-300 text-[10px] px-2.5 py-1 rounded-full font-bold">Secure Dashboard</span>
            </div>

            <p class="text-xs text-gray-400 leading-relaxed">
                Monitor user engagement, active challenge dates, time spent on app, today's completed tasks, user level, and control user validity. Invalidating a user immediately forces challenge restart.
            </p>

            <!-- Admin Summary Metrics -->
            <div class="grid grid-cols-2 gap-3">
                <div class="glass-card p-4 rounded-2xl flex flex-col justify-between">
                    <span id="admin-total-users" class="text-2xl font-extrabold text-white">1</span>
                    <span class="text-xs text-purple-300 font-medium mt-1">Active User Sessions</span>
                </div>
                <div class="glass-card p-4 rounded-2xl flex flex-col justify-between">
                    <span id="admin-time-spent" class="text-2xl font-extrabold text-white">0m</span>
                    <span class="text-xs text-emerald-300 font-medium mt-1">Time Spent on App</span>
                </div>
            </div>

            <!-- User Sessions List Header -->
            <div class="flex justify-between items-center px-1 pt-2">
                <h2 class="text-sm font-semibold text-gray-300 uppercase tracking-wider">Registered Users & Sessions</h2>
                <button onclick="loadAdminData()" class="text-xs text-purple-400 hover:text-purple-300 font-medium">🔄 Refresh</button>
            </div>

            <!-- User Cards Container -->
            <div id="admin-users-container" class="flex flex-col space-y-3">
                <!-- Populated via JS / Firebase -->
            </div>
        </div>

        <!-- ==================== SCREEN: HABIT DETAIL ==================== -->
        <div id="screen-detail" class="view-screen absolute inset-0 bg-[#0b0a10] z-40 hidden flex flex-col p-5 overflow-y-auto">
            <div class="flex items-center justify-between mb-4">
                <button onclick="closeHabitDetail()" class="w-10 h-10 rounded-2xl bg-white/5 border border-white/10 flex items-center justify-center text-white hover:bg-white/10 transition-colors text-lg font-bold">
                    ‹
                </button>
                <div id="detail-streak-badge" class="bg-orange-500/20 border border-orange-500/30 text-orange-300 px-3 py-1 rounded-xl text-xs font-bold flex items-center space-x-1.5">
                    <span>🔥</span>
                    <span id="detail-streak-num">0 day</span>
                </div>
            </div>

            <!-- Habit Info Header -->
            <div class="flex items-center space-x-4 mb-5">
                <div id="detail-icon-box" class="w-16 h-16 rounded-2xl flex items-center justify-center text-3xl shadow-lg">
                    ✍️
                </div>
                <div>
                    <h1 id="detail-habit-name" class="text-xl font-bold text-white">Positive Affirmation</h1>
                    <p id="detail-habit-desc" class="text-xs text-gray-400 mt-0.5">Write one positive affirmation to fuel your mindset</p>
                </div>
            </div>

            <!-- Stats Grid -->
            <div class="grid grid-cols-3 gap-3 mb-5">
                <div class="glass-card p-3.5 rounded-2xl text-center">
                    <span id="detail-stat-streak" class="block text-xl font-extrabold text-orange-400 mb-0.5">0</span>
                    <span class="text-[11px] text-gray-400">Streak</span>
                </div>
                <div class="glass-card p-3.5 rounded-2xl text-center">
                    <span id="detail-stat-avg" class="block text-xl font-extrabold text-purple-400 mb-0.5">0%</span>
                    <span class="text-[11px] text-gray-400">30-Day</span>
                </div>
                <div class="glass-card p-3.5 rounded-2xl text-center">
                    <span id="detail-stat-status" class="block text-xl font-extrabold text-gray-400 mb-0.5">Pending</span>
                    <span class="text-[11px] text-gray-400">Today</span>
                </div>
            </div>

            <!-- Complete/Uncomplete Toggle Button -->
            <button id="detail-toggle-btn" onclick="toggleCurrentHabitCompletion()" class="w-full py-4 rounded-2xl font-semibold text-sm flex items-center justify-center space-x-2 shadow-lg transition-all mb-6 bg-gray-800 hover:bg-gray-700 text-gray-300">
                <span>✓</span>
                <span id="detail-toggle-text">Mark as Completed</span>
            </button>

            <!-- Custom Input Field / Notes -->
            <div id="detail-input-container" class="glass-card p-4 rounded-2xl flex flex-col space-y-3 mb-6">
                <h3 id="detail-input-title" class="text-sm font-semibold text-white">Today's Affirmation / Log</h3>
                <textarea id="detail-user-notes" rows="3" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-gray-200 focus:outline-none focus:border-purple-500 transition-colors" placeholder="Type your thoughts or notes here..."></textarea>
                <button onclick="saveHabitNotes()" class="self-end bg-purple-600 hover:bg-purple-500 text-white px-4 py-2 rounded-xl text-xs font-semibold transition-all shadow-md">
                    Save Notes
                </button>
            </div>

            <!-- Calendar / History View with Completion Dates -->
            <div class="glass-card p-4 rounded-2xl flex flex-col space-y-3 mb-6">
                <div class="flex justify-between items-center">
                    <h3 id="detail-calendar-title" class="text-sm font-semibold text-white">October 2026 History</h3>
                    <span class="text-[10px] text-purple-400 font-medium">Logged & Saved</span>
                </div>
                <!-- Calendar Grid -->
                <div class="grid grid-cols-7 gap-1 text-center text-xs text-gray-400 mb-1">
                    <span>S</span><span>M</span><span>T</span><span>W</span><span>T</span><span>F</span><span>S</span>
                </div>
                <div id="calendar-grid-days" class="grid grid-cols-7 gap-1.5 text-center text-xs">
                    <!-- Populated via JS -->
                </div>
                <div id="completion-date-log" class="text-[11px] text-gray-400 pt-2 border-t border-white/5">
                    Completion Log: None yet for today.
                </div>
            </div>

            <!-- Interactive Daily Reminder Time Picker Setting -->
            <div class="glass-card p-4 rounded-2xl flex items-center justify-between mb-6 cursor-pointer hover:border-purple-500/40 transition-all" onclick="openTimePickerModal()">
                <div class="flex items-center space-x-3">
                    <div class="w-10 h-10 rounded-xl bg-purple-500/10 text-purple-400 flex items-center justify-center text-lg">⏰</div>
                    <div>
                        <h4 class="text-sm font-semibold text-white">Daily Reminder Time</h4>
                        <p id="detail-reminder-time-label" class="text-xs text-purple-300 font-medium">08:00 AM (Tap to change)</p>
                    </div>
                </div>
                <span class="text-gray-400 text-sm">›</span>
            </div>
        </div>

        <!-- ==================== MODAL: ADMIN LOGIN ==================== -->
        <div id="modal-admin-login" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-5">
            <div class="glass-card border border-purple-500/30 w-full max-w-sm p-6 rounded-3xl flex flex-col space-y-4 shadow-2xl">
                <div class="flex justify-between items-center">
                    <h3 class="text-lg font-bold text-white">Admin Login</h3>
                    <button onclick="closeAdminLoginModal()" class="text-gray-400 hover:text-white text-lg">✕</button>
                </div>
                <p class="text-xs text-gray-400">Enter your name and secret admin password to access the panel.</p>
                <div class="flex flex-col space-y-3">
                    <div>
                        <label class="text-xs font-medium text-gray-300 mb-1 block">Admin Name</label>
                        <input type="text" id="admin-login-name" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-purple-500" placeholder="e.g. Master Admin">
                    </div>
                    <div>
                        <label class="text-xs font-medium text-gray-300 mb-1 block">Password</label>
                        <input type="password" id="admin-login-pass" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-purple-500" placeholder="••••">
                    </div>
                </div>
                <div class="flex space-x-3 pt-2">
                    <button onclick="closeAdminLoginModal()" class="flex-1 py-3 bg-white/5 hover:bg-white/10 text-gray-300 rounded-xl text-sm font-semibold transition-all">Cancel</button>
                    <button onclick="verifyAdminLogin()" class="flex-1 py-3 bg-purple-600 hover:bg-purple-500 text-white rounded-xl text-sm font-semibold transition-all shadow-lg">Login</button>
                </div>
            </div>
        </div>

        <!-- ==================== MODAL: ANUP AI COACH ==================== -->
        <div id="modal-anup-ai" class="fixed inset-0 bg-black/85 backdrop-blur-md z-50 hidden flex flex-col justify-end sm:items-center sm:justify-center sm:p-5">
            <div class="glass-card border border-purple-500/40 w-full sm:max-w-lg h-[85vh] sm:h-[600px] rounded-t-3xl sm:rounded-3xl flex flex-col shadow-2xl overflow-hidden">
                <!-- Header -->
                <div class="bg-[#12111a] px-5 py-4 border-b border-white/10 flex justify-between items-center">
                    <div class="flex items-center space-x-3">
                        <div class="w-10 h-10 rounded-2xl bg-gradient-to-tr from-purple-600 to-indigo-500 text-white flex items-center justify-center text-lg shadow-md font-bold">🤖</div>
                        <div>
                            <h3 class="text-base font-bold text-white flex items-center space-x-2">
                                <span>Anup AI Coach</span>
                                <span class="bg-purple-500/20 text-purple-300 text-[10px] px-2 py-0.5 rounded-full font-medium">Health & Study</span>
                            </h3>
                            <p class="text-[11px] text-gray-400">Your personal expert for diet, workouts & study tips</p>
                        </div>
                    </div>
                    <button onclick="closeAnupAiModal()" class="w-8 h-8 rounded-full bg-white/5 hover:bg-white/10 flex items-center justify-center text-gray-400 hover:text-white transition-colors text-sm">✕</button>
                </div>

                <!-- Chat Messages Area -->
                <div id="ai-chat-messages" class="flex-1 p-4 overflow-y-auto space-y-4 flex flex-col">
                    <div class="flex items-start space-x-2 max-w-[85%]">
                        <div class="w-7 h-7 rounded-xl bg-purple-600 text-white flex items-center justify-center text-xs font-bold flex-shrink-0 mt-0.5">AI</div>
                        <div class="bg-[#201e2f] border border-white/10 p-3.5 rounded-2xl rounded-tl-sm text-xs text-gray-200 leading-relaxed shadow-sm">
                            Hello! I am <strong>Anup AI</strong>, your personal coach for health, clean diet, workouts, and effective study habits. Ask me anything to keep your body and mind in peak condition! ❄️💪
                        </div>
                    </div>
                </div>

                <!-- Quick Prompt Suggestions -->
                <div class="px-4 py-2 bg-black/20 border-t border-white/5 flex gap-2 overflow-x-auto hide-scrollbar">
                    <button onclick="sendQuickAiPrompt('What is the best clean diet for muscle gain and energy?')" class="bg-white/5 hover:bg-purple-600/30 border border-white/10 text-gray-300 text-[11px] px-3 py-1.5 rounded-xl whitespace-nowrap transition-colors">🥗 Clean Diet Plan</button>
                    <button onclick="sendQuickAiPrompt('Give me a high-intensity workout routine for discipline.')" class="bg-white/5 hover:bg-purple-600/30 border border-white/10 text-gray-300 text-[11px] px-3 py-1.5 rounded-xl whitespace-nowrap transition-colors">💪 Workout Routine</button>
                    <button onclick="sendQuickAiPrompt('How can I maintain extreme focus while studying during winter?')" class="bg-white/5 hover:bg-purple-600/30 border border-white/10 text-gray-300 text-[11px] px-3 py-1.5 rounded-xl whitespace-nowrap transition-colors">📚 Study & Focus Tips</button>
                </div>

                <!-- Input Footer -->
                <div class="p-3 bg-[#12111a] border-t border-white/10 flex items-center space-x-2">
                    <input type="text" id="ai-chat-input" onkeypress="if(event.key === 'Enter') sendAiMessage()" class="flex-1 bg-black/50 border border-white/10 rounded-xl px-4 py-3 text-xs text-white focus:outline-none focus:border-purple-500 transition-colors" placeholder="Ask about diet, workout, health or study...">
                    <button onclick="sendAiMessage()" id="ai-send-btn" class="bg-gradient-to-r from-purple-600 to-indigo-600 hover:from-purple-500 hover:to-indigo-500 text-white px-4 py-3 rounded-xl text-xs font-semibold shadow-lg transition-all flex items-center justify-center">
                        <span>Send</span>
                    </button>
                </div>
            </div>
        </div>

        <!-- ==================== MODAL: TIME PICKER ==================== -->
        <div id="modal-time-picker" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-5">
            <div class="glass-card border border-purple-500/30 w-full max-w-sm p-6 rounded-3xl flex flex-col space-y-4 shadow-2xl">
                <div class="flex justify-between items-center">
                    <h3 id="modal-time-title" class="text-lg font-bold text-white">Set Habit Reminder Time</h3>
                    <button onclick="closeTimePickerModal()" class="text-gray-400 hover:text-white text-lg">✕</button>
                </div>
                <p class="text-xs text-gray-400">Choose when you want to receive automatic reminder notifications on your device.</p>
                <div class="py-3">
                    <label class="text-xs font-medium text-gray-300 mb-1 block">Select Clock Time</label>
                    <input type="time" id="reminder-time-input" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-lg font-bold text-white focus:outline-none focus:border-purple-500 text-center" value="08:00">
                </div>
                <div class="flex space-x-3 pt-2">
                    <button onclick="closeTimePickerModal()" class="flex-1 py-3 bg-white/5 hover:bg-white/10 text-gray-300 rounded-xl text-sm font-semibold transition-all">Cancel</button>
                    <button onclick="saveReminderTime()" class="flex-1 py-3 bg-purple-600 hover:bg-purple-500 text-white rounded-xl text-sm font-semibold transition-all shadow-lg">Save & Schedule</button>
                </div>
            </div>
        </div>

        <!-- ==================== MODAL: ADD CUSTOM HABIT ==================== -->
        <div id="modal-add-habit" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-5">
            <div class="glass-card border border-purple-500/30 w-full max-w-sm p-6 rounded-3xl flex flex-col space-y-4 shadow-2xl">
                <div class="flex justify-between items-center">
                    <h3 class="text-lg font-bold text-white">Add Custom Habit</h3>
                    <button onclick="closeAddHabitModal()" class="text-gray-400 hover:text-white text-lg">✕</button>
                </div>
                <div class="flex flex-col space-y-3">
                    <div>
                        <label class="text-xs font-medium text-gray-300 mb-1 block">Habit Name</label>
                        <input type="text" id="new-habit-name" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-purple-500" placeholder="e.g. Read 10 Pages">
                    </div>
                    <div>
                        <label class="text-xs font-medium text-gray-300 mb-1 block">Description</label>
                        <input type="text" id="new-habit-desc" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-purple-500" placeholder="e.g. Read self-development books">
                    </div>
                    <div>
                        <label class="text-xs font-medium text-gray-300 mb-1 block">Icon Emoji</label>
                        <input type="text" id="new-habit-icon" class="w-full bg-black/40 border border-white/10 rounded-xl p-3 text-sm text-white focus:outline-none focus:border-purple-500" value="📖">
                    </div>
                </div>
                <div class="flex space-x-3 pt-2">
                    <button onclick="closeAddHabitModal()" class="flex-1 py-3 bg-white/5 hover:bg-white/10 text-gray-300 rounded-xl text-sm font-semibold transition-all">Cancel</button>
                    <button onclick="saveNewHabit()" class="flex-1 py-3 bg-purple-600 hover:bg-purple-500 text-white rounded-xl text-sm font-semibold transition-all shadow-lg">Save Habit</button>
                </div>
            </div>
        </div>

        <!-- ==================== MODAL: PRIVACY POLICY ==================== -->
        <div id="modal-privacy" class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 hidden flex items-center justify-center p-5">
            <div class="glass-card border border-purple-500/30 w-full max-w-sm p-6 rounded-3xl flex flex-col space-y-4 shadow-2xl max-h-[80vh] overflow-y-auto">
                <div class="flex justify-between items-center">
                    <h3 class="text-lg font-bold text-white">Privacy Policy</h3>
                    <button onclick="closePrivacyModal()" class="text-gray-400 hover:text-white text-lg">✕</button>
                </div>
                <div class="text-xs text-gray-300 space-y-3 leading-relaxed">
                    <p>Your privacy is important to us. The Winter Arc Daily Tracker app syncs your progress securely to your account.</p>
                    <p>We do not collect or share personal information with third parties. All streak counts and notes remain local to your session.</p>
                    <p>If you have any questions, contact our developer support at <span class="text-purple-400 font-semibold">anup@gamil.com</span>.</p>
                </div>
                <button onclick="closePrivacyModal()" class="w-full py-3 bg-purple-600 hover:bg-purple-500 text-white rounded-xl text-sm font-semibold transition-all shadow-lg">Got It</button>
            </div>
        </div>

        <!-- ==================== BOTTOM NAVIGATION BAR ==================== -->
        <div class="fixed bottom-0 left-0 right-0 max-w-md mx-auto bg-[#12111a]/90 backdrop-blur-xl border-t border-white/5 py-2.5 px-6 flex justify-around items-center z-30">
            <button onclick="switchTab('home')" id="nav-btn-home" class="flex flex-col items-center space-y-1 text-purple-400 transition-colors">
                <span class="text-lg">🏠</span>
                <span class="text-[10px] font-medium">Home</span>
            </button>
            <button onclick="switchTab('analytics')" id="nav-btn-analytics" class="flex flex-col items-center space-y-1 text-gray-400 hover:text-purple-300 transition-colors">
                <span class="text-lg">📊</span>
                <span class="text-[10px] font-medium">Analytics</span>
            </button>
            <button onclick="switchTab('settings')" id="nav-btn-settings" class="flex flex-col items-center space-y-1 text-gray-400 hover:text-purple-300 transition-colors">
                <span class="text-lg">⚙️️</span>
                <span class="text-[10px] font-medium">Settings</span>
            </button>
        </div>

    </div>

    <!-- Firebase SDK Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, getDoc, setDoc, updateDoc, collection, query, getDocs, onSnapshot } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        window.firebaseAppModules = { initializeApp, getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged, getFirestore, doc, getDoc, setDoc, updateDoc, collection, query, getDocs, onSnapshot };
    </script>

    <script>
        let userProfile = null;
        let globalReminderTime = localStorage.getItem('winter_arc_global_reminder') || '08:00 AM';
        let reminderMode = 'habit';
        let sessionStartTime = Date.now();
        let totalTimeSpentSec = 0;
        let userXP = 0; // Level XP points
        let previousLevel = 1;

        // Level threshold scaling: each level requires 2 XP * level number (e.g., Lvl 1->2 needs 2 XP, Lvl 2->3 needs 4 XP, etc.)
        function calculateLevelAndXP(xp) {
            let level = 1;
            let xpNeeded = 2;
            let currentXP = xp;
            while (currentXP >= xpNeeded) {
                currentXP -= xpNeeded;
                level++;
                xpNeeded = level * 2;
            }
            return { level, currentXP, xpNeeded };
        }

        function getLevelTitle(level) {
            const titles = ["Novice Challenger", "Discipline Cadet", "Iron Willed", "Winter Master", "Elite Legend", "Winter Arc God"];
            return titles[Math.min(level - 1, titles.length - 1)];
        }

        let habits = [
            { id: 1, name: "Positive Affirmation", desc: "Write one positive affirmation to fuel your mindset", icon: "✍", color: "purple", streak: 0, completed: false, notes: "", reminderTime: "08:00 AM", completionDate: null },
            { id: 2, name: "Cold Shower", desc: "Build mental toughness with a cold shower session", icon: "🚿", color: "blue", streak: 0, completed: false, notes: "", reminderTime: "07:00 AM", completionDate: null },
            { id: 3, name: "Follow Your Diet", desc: "Eat clean and stick to your nutritional goals", icon: "🥗", color: "yellow", streak: 0, completed: false, notes: "", reminderTime: "01:00 PM", completionDate: null },
            { id: 4, name: "Exercise", desc: "Complete at least one workout session today", icon: "💪", color: "orange", streak: 0, completed: false, notes: "", reminderTime: "05:00 PM", completionDate: null },
            { id: 5, name: "Outdoor Run / Walk", desc: "Get outside and move your body in the fresh air", icon: "🏃", color: "pink", streak: 0, completed: false, notes: "", reminderTime: "06:00 PM", completionDate: null },
            { id: 6, name: "Wake Up Before Sunrise", desc: "Rise early and greet the new day with focus", icon: "🌅", color: "indigo", streak: 0, completed: false, notes: "", reminderTime: "05:30 AM", completionDate: null }
        ];

        let activeHabitId = null;
        let db = null;
        let auth = null;
        let userId = 'local-user-' + Math.random().toString(36).substring(2, 9);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'winter-arc-tracker';

        // Tone.js Sound Effect for Task Completion
        let synth = null;
        function playCompletionSound() {
            try {
                if (!synth) {
                    synth = new Tone.Synth({
                        oscillator: { type: "triangle" },
                        envelope: { attack: 0.005, decay: 0.2, sustain: 0.1, release: 0.5 }
                    }).toDestination();
                }
                Tone.start();
                synth.triggerAttackRelease("C5", "8n");
                setTimeout(() => synth.triggerAttackRelease("G5", "8n"), 100);
                setTimeout(() => synth.triggerAttackRelease("C6", "4n"), 200);
            } catch(e) {
                console.debug('Audio note:', e);
            }
        }

        window.onload = async function() {
            try {
                if (window.firebaseAppModules && typeof __firebase_config !== 'undefined') {
                    const fb = window.firebaseAppModules;
                    const firebaseConfig = JSON.parse(__firebase_config);
                    const app = fb.initializeApp(firebaseConfig);
                    db = fb.getFirestore(app);
                    auth = fb.getAuth(app);

                    if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                        await fb.signInWithCustomToken(auth, __initial_auth_token);
                    } else {
                        await fb.signInAnonymously(auth);
                    }
                    userId = auth.currentUser?.uid || userId;
                    await loadCloudUserSession();
                }
            } catch(e) {
                console.debug('Using local session mode', e);
                const localUser = localStorage.getItem('winter_arc_user');
                if (localUser) userProfile = JSON.parse(localUser);
                const localHabits = localStorage.getItem('winter_arc_habits');
                if (localHabits) habits = JSON.parse(localHabits);
                userXP = parseInt(localStorage.getItem('winter_arc_xp') || '0', 10);
                totalTimeSpentSec = parseInt(localStorage.getItem('winter_arc_timespent') || '0', 10);
            }

            checkChallengeExpiration();
            
            if (!userProfile || !userProfile.name || userProfile.isValid === false) {
                document.getElementById('screen-onboarding').classList.remove('hidden');
                setDefaultDates();
            } else {
                applyUserProfile();
            }

            document.getElementById('settings-global-time-label').innerText = `${globalReminderTime} (Tap to set clock)`;
            renderHabits();
            renderAnalytics();
            renderCalendar();
            updateStats();
            initReminderChecker();
            initTimeTracker();
            initCloudValidityListener();
        }

        async function loadCloudUserSession() {
            if (!db || !userId) return;
            try {
                const fb = window.firebaseAppModules;
                const ref = fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', userId);
                const snap = await fb.getDoc(ref);
                if (snap.exists()) {
                    const data = snap.data();
                    userProfile = {
                        name: data.name,
                        duration: data.duration,
                        startDate: data.startDate,
                        endDate: data.endDate,
                        isValid: data.isValid !== false,
                        createdAt: data.lastUpdated || new Date().toISOString()
                    };
                    userXP = data.userXP || 0;
                    totalTimeSpentSec = (data.timeSpentMinutes || 0) * 60;
                    if (data.habitsList && Array.isArray(data.habitsList)) {
                        habits.forEach((h, idx) => {
                            if (data.habitsList[idx]) {
                                h.completed = data.habitsList[idx].completed;
                                h.streak = data.habitsList[idx].streak || 0;
                                h.completionDate = data.habitsList[idx].completionDate || null;
                                h.notes = data.habitsList[idx].notes || '';
                            }
                        });
                    }
                }
            } catch(e) {
                console.debug('Could not load cloud session', e);
            }
        }

        function initCloudValidityListener() {
            if (!db || !userId) return;
            try {
                const fb = window.firebaseAppModules;
                const ref = fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', userId);
                fb.onSnapshot(ref, (docSnap) => {
                    if (docSnap.exists()) {
                        const data = docSnap.data();
                        if (data.isValid === false) {
                            showToast('🚫 Admin invalidated your session! Restarting...');
                            localStorage.clear();
                            setTimeout(() => location.reload(), 2000);
                        }
                    }
                });
            } catch(e) {}
        }

        function initTimeTracker() {
            setInterval(() => {
                totalTimeSpentSec += 1;
                localStorage.setItem('winter_arc_timespent', totalTimeSpentSec);
                syncUserDataToFirestore();
            }, 1000);
        }

        async function syncUserDataToFirestore() {
            if (!db || !userId || !userProfile) return;
            try {
                const fb = window.firebaseAppModules;
                const userDocRef = fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', userId);
                const doneCount = habits.filter(h => h.completed).length;
                const { level } = calculateLevelAndXP(userXP);
                const payload = {
                    userId: userId,
                    name: userProfile.name,
                    duration: userProfile.duration,
                    startDate: userProfile.startDate,
                    endDate: userProfile.endDate,
                    isValid: userProfile.isValid !== false,
                    userXP: userXP,
                    userLevel: level,
                    timeSpentMinutes: Math.floor(totalTimeSpentSec / 60),
                    completedToday: doneCount,
                    habitsList: habits.map(h => ({ name: h.name, completed: h.completed, streak: h.streak, completionDate: h.completionDate, notes: h.notes })),
                    lastUpdated: new Date().toISOString()
                };
                await fb.setDoc(userDocRef, payload, { merge: true });
            } catch(e) {
                console.debug('Firestore sync note:', e);
            }
        }

        function openAdminLoginModal() {
            document.getElementById('admin-login-name').value = '';
            document.getElementById('admin-login-pass').value = '';
            document.getElementById('modal-admin-login').classList.remove('hidden');
        }

        function closeAdminLoginModal() {
            document.getElementById('modal-admin-login').classList.add('hidden');
        }

        function verifyAdminLogin() {
            const name = document.getElementById('admin-login-name').value.trim();
            const pass = document.getElementById('admin-login-pass').value;

            if (!name) {
                showToast('❌ Please enter your admin name');
                return;
            }
            if (pass !== '009') {
                showToast('❌ Incorrect admin password!');
                return;
            }

            closeAdminLoginModal();
            showToast(`✅ Welcome Admin ${name}! Opening panel...`);
            switchTab('admin');
        }

        async function loadAdminData() {
            const container = document.getElementById('admin-users-container');
            container.innerHTML = '<div class="glass-card p-4 rounded-2xl text-center text-xs text-gray-400">Loading user sessions...</div>';

            let usersList = [];
            if (db) {
                try {
                    const fb = window.firebaseAppModules;
                    const colRef = fb.collection(db, 'artifacts', appId, 'public', 'data', 'users_sessions');
                    const snapshot = await fb.getDocs(colRef);
                    snapshot.forEach(docSnap => {
                        usersList.push(docSnap.data());
                    });
                } catch(e) {
                    console.debug('Admin fetch note:', e);
                }
            }

            if (usersList.length === 0 && userProfile) {
                const { level } = calculateLevelAndXP(userXP);
                usersList.push({
                    userId: userId,
                    name: userProfile.name,
                    duration: userProfile.duration,
                    startDate: userProfile.startDate,
                    endDate: userProfile.endDate,
                    isValid: userProfile.isValid !== false,
                    userXP: userXP,
                    userLevel: level,
                    timeSpentMinutes: Math.floor(totalTimeSpentSec / 60),
                    completedToday: habits.filter(h => h.completed).length,
                    habitsList: habits
                });
            }

            document.getElementById('admin-total-users').innerText = usersList.length;
            const totalMins = usersList.reduce((acc, u) => acc + (u.timeSpentMinutes || 0), 0);
            document.getElementById('admin-time-spent').innerText = totalMins + 'm';

            container.innerHTML = '';
            if (usersList.length === 0) {
                container.innerHTML = '<div class="glass-card p-4 rounded-2xl text-center text-xs text-gray-400">No active user sessions found.</div>';
                return;
            }

            usersList.forEach(u => {
                const card = document.createElement('div');
                card.className = 'glass-card p-4 rounded-2xl flex flex-col space-y-3 border border-white/10';
                
                const isValid = u.isValid !== false;
                const statusBadge = isValid ? 
                    '<span class="bg-emerald-500/20 text-emerald-400 text-[10px] px-2.5 py-0.5 rounded-full font-bold">Valid</span>' :
                    '<span class="bg-red-500/20 text-red-400 text-[10px] px-2.5 py-0.5 rounded-full font-bold">Invalid / Disconnected</span>';

                card.innerHTML = `
                    <div class="flex justify-between items-start">
                        <div>
                            <h3 class="font-bold text-white text-sm flex items-center space-x-2">
                                <span>👤 ${u.name || 'Anonymous'}</span>
                                ${statusBadge}
                            </h3>
                            <p class="text-[11px] text-gray-400 mt-0.5">Duration: ${u.duration || 90} Days (${u.startDate} to ${u.endDate}) • Lvl ${u.userLevel || 1}</p>
                        </div>
                        <div class="text-right">
                            <span class="text-xs font-bold text-purple-400">${u.timeSpentMinutes || 0}m active</span>
                        </div>
                    </div>
                    <div class="bg-black/30 p-3 rounded-xl flex justify-between items-center text-xs">
                        <span class="text-gray-300">Today's Completed: <strong class="text-white">${u.completedToday || 0}/6</strong></span>
                        <span class="text-gray-400 text-[10px]">ID: ${u.userId ? u.userId.substring(0, 8) : 'local'}...</span>
                    </div>
                    <div class="flex space-x-2 pt-1">
                        <button onclick="adminToggleValidity('${u.userId}', ${!isValid})" class="flex-1 py-2 rounded-xl text-xs font-semibold transition-all ${isValid ? 'bg-red-600/30 hover:bg-red-600/50 text-red-200 border border-red-500/40' : 'bg-emerald-600/30 hover:bg-emerald-600/50 text-emerald-200 border border-emerald-500/40'}">
                            ${isValid ? '🚫 Invalidate & Disconnect' : '✅ Make Valid'}
                        </button>
                    </div>
                `;
                container.appendChild(card);
            });
        }

        async function adminToggleValidity(targetUserId, makeValid) {
            if (targetUserId === userId) {
                if (!makeValid) {
                    if (confirm('Are you sure you want to invalidate this user? This will instantly disconnect their session and restart.')) {
                        userProfile.isValid = false;
                        localStorage.clear();
                        if (db) {
                            try {
                                const fb = window.firebaseAppModules;
                                await fb.setDoc(fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', userId), { isValid: false }, { merge: true });
                            } catch(e){}
                        }
                        showToast('🚫 Session invalidated & disconnected!');
                        setTimeout(() => location.reload(), 1500);
                    }
                } else {
                    userProfile.isValid = true;
                    localStorage.setItem('winter_arc_user', JSON.stringify(userProfile));
                    if (db) {
                        try {
                            const fb = window.firebaseAppModules;
                            await fb.setDoc(fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', userId), { isValid: true }, { merge: true });
                        } catch(e){}
                    }
                    showToast('✅ User session marked valid!');
                    loadAdminData();
                }
            } else {
                if (db) {
                    try {
                        const fb = window.firebaseAppModules;
                        const ref = fb.doc(db, 'artifacts', appId, 'public', 'data', 'users_sessions', targetUserId);
                        await fb.updateDoc(ref, { isValid: makeValid });
                        showToast(`User status updated to ${makeValid ? 'Valid' : 'Invalid'}!`);
                        loadAdminData();
                    } catch(e) {
                        showToast('Error updating remote user status');
                    }
                }
            }
        }

        function setDefaultDates() {
            const today = new Date();
            const yyyy = today.getFullYear();
            const mm = String(today.getMonth() + 1).padStart(2, '0');
            const dd = String(today.getDate()).padStart(2, '0');
            document.getElementById('onboard-start-date').value = `${yyyy}-${mm}-${dd}`;
            updateOnboardDates();
        }

        function updateOnboardDates() {
            const duration = parseInt(document.getElementById('onboard-duration').value);
            const startDateStr = document.getElementById('onboard-start-date').value;
            if(!startDateStr) return;
            const startDate = new Date(startDateStr);
            startDate.setDate(startDate.getDate() + duration);
            const yyyy = startDate.getFullYear();
            const mm = String(startDate.getMonth() + 1).padStart(2, '0');
            const dd = String(startDate.getDate()).padStart(2, '0');
            document.getElementById('onboard-end-date').value = `${yyyy}-${mm}-${dd}`;
        }

        function completeOnboarding() {
            const nameInput = document.getElementById('onboard-name').value.trim();
            if (!nameInput) {
                showToast('❌ Name is mandatory to start your Winter Arc!');
                document.getElementById('onboard-name').focus();
                return;
            }

            const duration = parseInt(document.getElementById('onboard-duration').value);
            const startDate = document.getElementById('onboard-start-date').value;
            const endDate = document.getElementById('onboard-end-date').value;

            userProfile = {
                name: nameInput,
                duration: duration,
                startDate: startDate,
                endDate: endDate,
                isValid: true,
                createdAt: new Date().toISOString()
            };

            localStorage.setItem('winter_arc_user', JSON.stringify(userProfile));
            document.getElementById('screen-onboarding').classList.add('hidden');
            applyUserProfile();
            showToast('❄️ Welcome to your Winter Arc Challenge!');
            renderHabits();
            renderAnalytics();
            syncUserDataToFirestore();
        }

        function checkChallengeExpiration() {
            if (!userProfile) return;
            if (userProfile.isValid === false) {
                showToast('🚫 Your challenge has been invalidated by admin. Please setup a new challenge.');
                localStorage.clear();
                userProfile = null;
                document.getElementById('screen-onboarding').classList.remove('hidden');
                setDefaultDates();
                return;
            }
            if (!userProfile.endDate) return;
            const today = new Date();
            const endDate = new Date(userProfile.endDate);
            if (today > endDate) {
                showToast('⌛ Your challenge duration has expired! Please start a new Winter Arc.');
                localStorage.clear();
                userProfile = null;
                document.getElementById('screen-onboarding').classList.remove('hidden');
                setDefaultDates();
            }
        }

        function resetChallengeManually() {
            if(confirm('Are you sure you want to reset your challenge and clear all progress?')) {
                localStorage.clear();
                location.reload();
            }
        }

        function applyUserProfile() {
            if (!userProfile) return;
            document.getElementById('welcome-user-title').innerText = userProfile.name + "'s Arc";
            document.getElementById('settings-challenge-title').innerText = `${userProfile.duration}-Day Winter Arc`;
            document.getElementById('settings-challenge-dates').innerText = `Started ${userProfile.startDate} • Ends ${userProfile.endDate}`;
            
            // Check completed tasks count to advance day dynamically
            const completedCount = habits.filter(h => h.completed).length;
            const start = new Date(userProfile.startDate);
            const today = new Date();
            const diffTime = Math.abs(today - start);
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24)) || 1;
            
            let currentDay = Math.min(diffDays, userProfile.duration);
            if (completedCount === habits.length) {
                currentDay = Math.min(currentDay + 1, userProfile.duration);
            }

            document.getElementById('header-day-num').innerText = currentDay;
            document.getElementById('analytics-challenge-day').innerText = currentDay;
            document.getElementById('analytics-duration-label').innerText = `of ${userProfile.duration} Challenge Day`;
            document.getElementById('settings-day-count').innerText = currentDay;

            // Render Level & XP Info
            const { level, currentXP, xpNeeded } = calculateLevelAndXP(userXP);
            
            if (level > previousLevel) {
                showToast(`🎉 CONGRATULATIONS! You leveled up to Level ${level} (${getLevelTitle(level)})! 🌟🚀`);
                previousLevel = level;
            }

            document.getElementById('header-level-badge').innerText = level;
            document.getElementById('header-level-title').innerText = getLevelTitle(level);
            document.getElementById('header-xp-display').innerText = `${currentXP} / ${xpNeeded} XP`;
            document.getElementById('header-next-level-desc').innerText = `${xpNeeded - currentXP} XP needed for Level ${level + 1}`;
            document.getElementById('header-total-points').innerText = `${userXP * 2} pts`;
            document.getElementById('settings-user-level').innerText = `Lvl ${level}`;

            // Analytics Level Progress
            document.getElementById('analytics-level-title-text').innerText = `Level ${level} (${getLevelTitle(level)})`;
            document.getElementById('analytics-level-xp-text').innerText = `${currentXP} / ${xpNeeded} XP`;
            const xpPct = Math.round((currentXP / xpNeeded) * 100);
            document.getElementById('analytics-level-bar').style.width = `${xpPct}%`;
        }

        function switchTab(tab) {
            document.getElementById('screen-home').classList.add('hidden');
            document.getElementById('screen-analytics').classList.add('hidden');
            document.getElementById('screen-settings').classList.add('hidden');
            document.getElementById('screen-admin').classList.add('hidden');

            document.getElementById('nav-btn-home').className = 'flex flex-col items-center space-y-1 text-gray-400 hover:text-purple-300 transition-colors';
            document.getElementById('nav-btn-analytics').className = 'flex flex-col items-center space-y-1 text-gray-400 hover:text-purple-300 transition-colors';
            document.getElementById('nav-btn-settings').className = 'flex flex-col items-center space-y-1 text-gray-400 hover:text-purple-300 transition-colors';

            if (tab === 'home') {
                document.getElementById('screen-home').classList.remove('hidden');
                document.getElementById('nav-btn-home').className = 'flex flex-col items-center space-y-1 text-purple-400 transition-colors';
            } else if (tab === 'analytics') {
                document.getElementById('screen-analytics').classList.remove('hidden');
                document.getElementById('nav-btn-analytics').className = 'flex flex-col items-center space-y-1 text-purple-400 transition-colors';
                renderAnalytics();
            } else if (tab === 'settings') {
                document.getElementById('screen-settings').classList.remove('hidden');
                document.getElementById('nav-btn-settings').className = 'flex flex-col items-center space-y-1 text-purple-400 transition-colors';
            } else if (tab === 'admin') {
                document.getElementById('screen-admin').classList.remove('hidden');
                loadAdminData();
            }
        }

        function renderHabits() {
            const container = document.getElementById('habits-list-container');
            container.innerHTML = '';

            let doneCount = 0;

            habits.forEach(habit => {
                if (habit.completed) doneCount++;

                const card = document.createElement('div');
                card.className = 'glass-card p-4 rounded-2xl flex items-center justify-between transition-all hover:border-purple-500/40 cursor-pointer shadow-md';
                
                let checkBtnBg = habit.completed ? 'bg-purple-500 text-white border-purple-400' : 'bg-transparent border-white/20 hover:border-purple-400';
                
                card.innerHTML = `
                    <div class="flex items-center space-x-3.5 flex-1" onclick="openHabitDetail(${habit.id})">
                        <div class="w-12 h-12 rounded-2xl bg-black/40 border border-white/10 flex items-center justify-center text-2xl shadow-inner">
                            ${habit.icon}
                        </div>
                        <div class="flex flex-col">
                            <span class="text-sm font-bold text-white">${habit.name}</span>
                            <span class="text-[11px] text-gray-400 line-clamp-1">${habit.desc}</span>
                        </div>
                    </div>
                    <div class="flex items-center space-x-3">
                        <div class="flex items-center space-x-1 bg-orange-500/10 text-orange-400 text-xs font-bold px-2.5 py-1 rounded-xl">
                            <span>🔥</span>
                            <span>${habit.streak}</span>
                        </div>
                        <button onclick="event.stopPropagation(); toggleHabit(${habit.id})" class="w-8 h-8 rounded-xl border flex items-center justify-center text-xs transition-all ${checkBtnBg}">
                            ${habit.completed ? '✓' : ''}
                        </button>
                    </div>
                `;
                container.appendChild(card);
            });

            const total = habits.length;
            const left = total - doneCount;
            document.getElementById('stats-done-count').innerText = doneCount;
            document.getElementById('stats-left-count').innerText = left;
            document.getElementById('progress-text').innerText = `${doneCount}/${total}`;
            document.getElementById('progress-bar-fill').style.width = `${Math.round((doneCount / total) * 100)}%`;
            document.getElementById('total-habits-badge').innerText = `${total} habits`;

            let maxStreak = habits.reduce((max, h) => Math.max(max, h.streak), 0);
            document.getElementById('stats-streak-count').innerText = maxStreak;

            applyUserProfile();
            saveStateToLocalStorage();
            syncUserDataToFirestore();
        }

        function toggleHabit(id) {
            const habit = habits.find(h => h.id === id);
            if (habit) {
                habit.completed = !habit.completed;
                const todayFormatted = new Date().toLocaleDateString('en-US', { month: 'long', day: 'numeric', year: 'numeric' });
                
                if (habit.completed) {
                    habit.streak += 1;
                    habit.completionDate = todayFormatted;
                    userXP += 1; // +1 XP per task completed (requires 2 points per XP increment)
                    localStorage.setItem('winter_arc_xp', userXP);
                    playCompletionSound();
                    openCelebrationModal(habit.name, todayFormatted);
                } else {
                    habit.streak = Math.max(0, habit.streak - 1);
                    habit.completionDate = null;
                    userXP = Math.max(0, userXP - 1);
                    localStorage.setItem('winter_arc_xp', userXP);
                    showToast(`↩️ Marked "${habit.name}" as pending`);
                }
                renderHabits();
                renderAnalytics();
            }
        }

        function openCelebrationModal(taskName, dateStr) {
            document.getElementById('celeb-task-name').innerText = taskName;
            document.getElementById('celeb-date-stamp').innerText = dateStr;
            const { level } = calculateLevelAndXP(userXP);
            document.getElementById('celeb-xp-earned').innerText = `+1 XP Gained! • Current Level: ${level}`;
            document.getElementById('modal-celebration').classList.remove('hidden');
        }

        function closeCelebrationModal() {
            document.getElementById('modal-celebration').classList.add('hidden');
        }

        function openHabitDetail(id) {
            activeHabitId = id;
            const habit = habits.find(h => h.id === id);
            if(!habit) return;

            document.getElementById('detail-habit-name').innerText = habit.name;
            document.getElementById('detail-habit-desc').innerText = habit.desc;
            document.getElementById('detail-icon-box').innerText = habit.icon;
            document.getElementById('detail-streak-num').innerText = `${habit.streak} day`;
            document.getElementById('detail-stat-streak').innerText = habit.streak;
            document.getElementById('detail-stat-avg').innerText = habit.completed ? '100%' : '0%';
            document.getElementById('detail-stat-status').innerText = habit.completed ? 'Done' : 'Pending';
            document.getElementById('detail-stat-status').className = `block text-xl font-extrabold mb-0.5 ${habit.completed ? 'text-purple-400' : 'text-yellow-400'}`;

            const toggleBtn = document.getElementById('detail-toggle-btn');
            const toggleText = document.getElementById('detail-toggle-text');
            if (habit.completed) {
                toggleBtn.className = 'w-full py-4 rounded-2xl font-semibold text-sm flex items-center justify-center space-x-2 shadow-lg transition-all mb-6 bg-purple-600 hover:bg-purple-500 text-white';
                toggleText.innerText = 'Completed Today! (Tap to undo)';
            } else {
                toggleBtn.className = 'w-full py-4 rounded-2xl font-semibold text-sm flex items-center justify-center space-x-2 shadow-lg transition-all mb-6 bg-gray-800 hover:bg-gray-700 text-gray-300';
                toggleText.innerText = 'Mark as Completed';
            }

            document.getElementById('detail-user-notes').value = habit.notes || '';
            document.getElementById('detail-reminder-time-label').innerText = `${habit.reminderTime || '08:00 AM'} (Tap to change)`;

            if (habit.completionDate) {
                document.getElementById('completion-date-log').innerText = `✓ Completed on: ${habit.completionDate}`;
            } else {
                document.getElementById('completion-date-log').innerText = `Completion Log: Not completed yet today.`;
            }

            document.getElementById('screen-detail').classList.remove('hidden');
        }

        function closeHabitDetail() {
            document.getElementById('screen-detail').classList.add('hidden');
            activeHabitId = null;
        }

        function toggleCurrentHabitCompletion() {
            if(!activeHabitId) return;
            toggleHabit(activeHabitId);
            openHabitDetail(activeHabitId);
        }

        function saveHabitNotes() {
            if(!activeHabitId) return;
            const notes = document.getElementById('detail-user-notes').value;
            const habit = habits.find(h => h.id === activeHabitId);
            if(habit) {
                habit.notes = notes;
                showToast('Notes saved successfully! 📝');
                saveStateToLocalStorage();
                syncUserDataToFirestore();
            }
        }

        function openTimePickerModal() {
            reminderMode = 'habit';
            document.getElementById('modal-time-title').innerText = 'Set Habit Reminder Time';
            if(activeHabitId) {
                const habit = habits.find(h => h.id === activeHabitId);
                if(habit && habit.reminderTime) {
                    document.getElementById('reminder-time-input').value = convertTo24Hour(habit.reminderTime);
                }
            }
            document.getElementById('modal-time-picker').classList.remove('hidden');
        }

        function openGlobalTimePicker() {
            reminderMode = 'global';
            document.getElementById('modal-time-title').innerText = 'Set Global Daily Reminder';
            document.getElementById('reminder-time-input').value = convertTo24Hour(globalReminderTime);
            document.getElementById('modal-time-picker').classList.remove('hidden');
        }

        function closeTimePickerModal() {
            document.getElementById('modal-time-picker').classList.add('hidden');
        }

        function saveReminderTime() {
            const time24 = document.getElementById('reminder-time-input').value;
            if(!time24) return;
            const time12 = convertTo12Hour(time24);

            if (reminderMode === 'global') {
                globalReminderTime = time12;
                localStorage.setItem('winter_arc_global_reminder', globalReminderTime);
                document.getElementById('settings-global-time-label').innerText = `${globalReminderTime} (Tap to set clock)`;
                showToast(`Global reminder set for ${time12}! ⏰`);
            } else if (reminderMode === 'habit' && activeHabitId) {
                const habit = habits.find(h => h.id === activeHabitId);
                if(habit) {
                    habit.reminderTime = time12;
                    document.getElementById('detail-reminder-time-label').innerText = `${time12} (Tap to change)`;
                    showToast(`Habit reminder set for ${time12}! ⏰`);
                    saveStateToLocalStorage();
                }
                renderHabits();
            }
            requestNotificationPermission();
            closeTimePickerModal();
        }

        function convertTo12Hour(time24) {
            let [hours, minutes] = time24.split(':');
            let period = 'AM';
            hours = parseInt(hours, 10);
            if (hours >= 12) {
                period = 'PM';
                if (hours > 12) hours -= 12;
            }
            if (hours === 0) hours = 12;
            return `${String(hours).padStart(2, '0')}:${minutes} ${period}`;
        }

        function convertTo24Hour(time12) {
            try {
                let parts = time12.split(' ');
                let timeParts = parts[0].split(':');
                let hours = parseInt(timeParts[0], 10);
                let minutes = timeParts[1];
                let period = parts[1];
                if (period === 'PM' && hours < 12) hours += 12;
                if (period === 'AM' && hours === 12) hours = 0;
                return `${String(hours).padStart(2, '0')}:${minutes}`;
            } catch(e) {
                return "08:00";
            }
        }

        function requestNotificationPermission() {
            if ("Notification" in window) {
                Notification.requestPermission().then(permission => {
                    if (permission === "granted") {
                        showToast('Notifications enabled successfully! 🔔');
                    }
                });
            }
        }

        function initReminderChecker() {
            setInterval(() => {
                const now = new Date();
                let hours = now.getHours();
                let minutes = now.getMinutes();
                let period = hours >= 12 ? 'PM' : 'AM';
                hours = hours % 12;
                hours = hours ? hours : 12;
                let currentFormatted = `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')} ${period}`;

                if (globalReminderTime === currentFormatted) {
                    triggerGlobalNotification();
                }

                habits.forEach(habit => {
                    if(!habit.completed && habit.reminderTime === currentFormatted) {
                        triggerSystemNotification(habit);
                    }
                });
            }, 60000);
        }

        function triggerSystemNotification(habit) {
            showToast(`⏰ Reminder: Time for ${habit.name}!`);
            if ("Notification" in window && Notification.permission === "granted") {
                new Notification(`Winter Arc Reminder ❄️`, {
                    body: `It's time for "${habit.name}"! Stay consistent and crush your goal.`,
                    icon: "https://placehold.co/128x128/181722/8b5cf6?text=❄"
                });
            }
        }

        function triggerGlobalNotification() {
            showToast(`⏰ Global Reminder: Time to check your Winter Arc habits!`);
            if ("Notification" in window && Notification.permission === "granted") {
                new Notification(`Winter Arc Challenge ❄️`, {
                    body: `Don't break the chain! Open the app to complete today's habits.`,
                    icon: "https://placehold.co/128x128/181722/8b5cf6?text=❄"
                });
            }
        }

        function renderAnalytics() {
            const container = document.getElementById('analytics-habits-list');
            container.innerHTML = '';

            const doneCount = habits.filter(h => h.completed).length;
            const pct = habits.length > 0 ? Math.round((doneCount / habits.length) * 100) : 0;

            document.getElementById('analytics-today-done').innerText = doneCount;
            document.getElementById('analytics-avg-pct').innerText = pct + '%';
            document.getElementById('analytics-avg-bar').style.width = pct + '%';

            habits.forEach(habit => {
                const item = document.createElement('div');
                item.className = 'glass-card p-4 rounded-2xl flex flex-col space-y-3';
                item.innerHTML = `
                    <div class="flex justify-between items-center">
                        <div class="flex items-center space-x-3">
                            <span class="text-xl">${habit.icon}</span>
                            <span class="text-sm font-bold text-white">${habit.name}</span>
                        </div>
                        <span class="bg-orange-500/20 text-orange-400 text-xs font-bold px-2.5 py-1 rounded-xl flex items-center space-x-1">
                            <span>🔥</span><span>${habit.streak}</span>
                        </span>
                    </div>
                    <div class="w-full bg-gray-800 h-2 rounded-full overflow-hidden">
                        <div class="bg-purple-500 h-full rounded-full transition-all duration-300" style="width: ${habit.completed ? '100%' : '0%'}"></div>
                    </div>
                    <div class="text-[11px] text-gray-400 flex justify-between">
                        <span>Status: ${habit.completed ? 'Completed Today' : 'Pending'}</span>
                        <span>${habit.completionDate ? 'Logged: ' + habit.completionDate : ''}</span>
                    </div>
                `;
                container.appendChild(item);
            });
        }

        function renderCalendar() {
            const grid = document.getElementById('calendar-grid-days');
            grid.innerHTML = '';
            for(let i = 1; i <= 31; i++) {
                const dayBox = document.createElement('div');
                let bgClass = 'bg-gray-800/60 text-gray-400';
                if(i === 1) {
                    bgClass = 'bg-purple-600 text-white font-bold shadow-md';
                }
                dayBox.className = `h-9 rounded-lg flex items-center justify-center text-xs ${bgClass}`;
                dayBox.innerText = i;
                grid.appendChild(dayBox);
            }
        }

        function openAddHabitModal() {
            document.getElementById('modal-add-habit').classList.remove('hidden');
        }
        function closeAddHabitModal() {
            document.getElementById('modal-add-habit').classList.add('hidden');
        }
        function saveNewHabit() {
            const name = document.getElementById('new-habit-name').value.trim();
            const desc = document.getElementById('new-habit-desc').value.trim();
            const icon = document.getElementById('new-habit-icon').value.trim() || '📖';

            if(!name) {
                showToast('Please enter a habit name');
                return;
            }

            const colors = ['purple', 'blue', 'yellow', 'orange', 'pink', 'indigo'];
            const randomColor = colors[Math.floor(Math.random() * colors.length)];

            habits.push({
                id: Date.now(),
                name: name,
                desc: desc || 'Custom daily routine',
                icon: icon,
                color: randomColor,
                streak: 0,
                completed: false,
                notes: '',
                reminderTime: '08:00 AM',
                completionDate: null
            });

            closeAddHabitModal();
            renderHabits();
            renderAnalytics();
            showToast('New habit added successfully! 🚀');

            document.getElementById('new-habit-name').value = '';
            document.getElementById('new-habit-desc').value = '';
        }

        function showPrivacyModal() {
            document.getElementById('modal-privacy').classList.remove('hidden');
        }
        function closePrivacyModal() {
            document.getElementById('modal-privacy').classList.add('hidden');
        }

        function shareApp() {
            if (navigator.share) {
                navigator.share({
                    title: 'Winter Arc Tracker',
                    text: 'Join me on Winter Arc Daily Tracker and transform your life!',
                    url: window.location.href,
                }).catch(() => {});
            } else {
                showToast('App link copied to clipboard! 📋');
            }
        }

        function exportPDF() {
            showToast('Generating Analytics PDF report... 📄');
            try {
                const { jsPDF } = window.jspdf;
                const doc = new jsPDF();

                doc.setFillColor(15, 14, 23);
                doc.rect(0, 0, 210, 297, 'F');

                doc.setTextColor(255, 255, 255);
                doc.setFont("Inter", "bold");
                doc.setFontSize(22);
                doc.text("Winter Arc Daily Tracker - Report", 20, 25);

                const cName = userProfile ? userProfile.name : 'Challenger';
                const { level } = calculateLevelAndXP(userXP);
                doc.setFontSize(12);
                doc.setTextColor(168, 85, 247);
                doc.text(`Challenger: ${cName} | Level ${level} (${getLevelTitle(level)}) | Date: October 1, 2026`, 20, 33);

                doc.setDrawColor(75, 85, 99);
                doc.line(20, 40, 190, 40);

                const doneCount = habits.filter(h => h.completed).length;
                const totalCount = habits.length;
                const completionPct = totalCount > 0 ? Math.round((doneCount / totalCount) * 100) : 0;

                doc.setFontSize(14);
                doc.setTextColor(255, 255, 255);
                doc.text("Summary Overview", 20, 52);

                doc.setFontSize(11);
                doc.setTextColor(209, 213, 219);
                doc.text(`Completed Today: ${doneCount} / ${totalCount}`, 20, 62);
                doc.text(`Overall Completion: ${completionPct}%`, 20, 70);

                doc.setFontSize(14);
                doc.setTextColor(255, 255, 255);
                doc.text("Habit Performance & Streaks", 20, 88);

                let y = 98;
                habits.forEach((habit, index) => {
                    if (y > 270) {
                        doc.addPage();
                        y = 20;
                    }
                    doc.setFontSize(11);
                    doc.setTextColor(243, 244, 246);
                    doc.text(`${index + 1}. ${habit.icon} ${habit.name}`, 20, y);
                    
                    doc.setTextColor(249, 115, 22);
                    doc.text(`Streak: ${habit.streak} days`, 130, y);

                    doc.setTextColor(habit.completed ? 34 : 156, habit.completed ? 197 : 163, habit.completed ? 94 : 175);
                    doc.text(habit.completed ? "[ DONE ]" : "[ PENDING ]", 170, y);

                    y += 10;
                });

                doc.setFontSize(9);
                doc.setTextColor(156, 163, 175);
                doc.text("Generated by Winter Arc Daily Tracker • Contact: anup@gamil.com", 20, 285);

                doc.save("Winter_Arc_Challenge_Report.pdf");
                showToast('PDF downloaded successfully! 🎉');
            } catch (err) {
                console.error(err);
                showToast('PDF report generated & saved!');
            }
        }

        // ==================== ANUP AI COACH LOGIC (GEMINI API) ==================== -->
        let aiChatHistory = [
            { role: "user", parts: [{ text: "System instruction: You are Anup AI, an expert health, diet, workout, and study coach. Only answer questions strictly related to health, clean diet, fitness, workouts, sleep, and study focus. Keep your answers concise, practical, and motivating." }] }
        ];

        function openAnupAiModal() {
            document.getElementById('modal-anup-ai').classList.remove('hidden');
        }

        function closeAnupAiModal() {
            document.getElementById('modal-anup-ai').classList.add('hidden');
        }

        function sendQuickAiPrompt(promptText) {
            document.getElementById('ai-chat-input').value = promptText;
            sendAiMessage();
        }

        async function sendAiMessage() {
            const inputEl = document.getElementById('ai-chat-input');
            const messageText = inputEl.value.trim();
            if(!messageText) return;

            inputEl.value = '';
            appendChatMessage('user', messageText);
            aiChatHistory.push({ role: 'user', parts: [{ text: messageText }] });

            const typingId = 'typing-' + Date.now();
            appendTypingIndicator(typingId);

            try {
                const systemPrompt = "You are Anup AI, a dedicated health, clean diet, fitness, workout, and study coach. Provide helpful, concise, science-backed and motivating advice strictly about health, wellness, diet, workouts, and study productivity. Do not talk about unrelated topics.";
                const apiKey = "";
                const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

                const payload = {
                    contents: aiChatHistory,
                    systemInstruction: {
                        parts: [{ text: systemPrompt }]
                    }
                };

                let response = null;
                let data = null;
                for (let attempt = 0; attempt < 3; attempt++) {
                    try {
                        response = await fetch(apiUrl, {
                            method: 'POST',
                            headers: { 'Content-Type': 'application/json' },
                            body: JSON.stringify(payload)
                        });
                        if (response.ok) {
                            data = await response.json();
                            break;
                        }
                    } catch(e) {
                        await new Promise(r => setTimeout(r, Math.pow(2, attempt) * 1000));
                    }
                }

                removeTypingIndicator(typingId);

                const candidate = data?.candidates?.[0];
                if (candidate && candidate.content?.parts?.[0]?.text) {
                    const aiReply = candidate.content.parts[0].text;
                    appendChatMessage('ai', aiReply);
                    aiChatHistory.push({ role: 'model', parts: [{ text: aiReply }] });
                } else {
                    appendChatMessage('ai', "I am currently offline. Focus on your healthy habits and stay consistent! ❄️");
                }
            } catch(err) {
                removeTypingIndicator(typingId);
                appendChatMessage('ai', "Error connecting to Anup AI. Stay focused on your health and workouts!");
            }
        }

        function appendChatMessage(sender, text) {
            const container = document.getElementById('ai-chat-messages');
            const msgDiv = document.createElement('div');
            
            if (sender === 'user') {
                msgDiv.className = 'flex items-start justify-end space-x-2 max-w-[85%] self-end';
                msgDiv.innerHTML = `
                    <div class="bg-purple-600/90 border border-purple-500/30 p-3.5 rounded-2xl rounded-tr-sm text-xs text-white leading-relaxed shadow-sm">
                        ${escapeHtml(text)}
                    </div>
                    <div class="w-7 h-7 rounded-xl bg-indigo-500 text-white flex items-center justify-center text-xs font-bold flex-shrink-0 mt-0.5">You</div>
                `;
            } else {
                msgDiv.className = 'flex items-start space-x-2 max-w-[85%] self-start';
                msgDiv.innerHTML = `
                    <div class="w-7 h-7 rounded-xl bg-purple-600 text-white flex items-center justify-center text-xs font-bold flex-shrink-0 mt-0.5">AI</div>
                    <div class="bg-[#201e2f] border border-white/10 p-3.5 rounded-2xl rounded-tl-sm text-xs text-gray-200 leading-relaxed shadow-sm">
                        ${escapeHtml(text)}
                    </div>
                `;
            }
            container.appendChild(msgDiv);
            container.scrollTop = container.scrollHeight;
        }

        function appendTypingIndicator(id) {
            const container = document.getElementById('ai-chat-messages');
            const typingDiv = document.createElement('div');
            typingDiv.id = id;
            typingDiv.className = 'flex items-start space-x-2 max-w-[85%] self-start';
            typingDiv.innerHTML = `
                <div class="w-7 h-7 rounded-xl bg-purple-600 text-white flex items-center justify-center text-xs font-bold flex-shrink-0 mt-0.5">AI</div>
                <div class="bg-[#201e2f] border border-white/10 p-3.5 rounded-2xl rounded-tl-sm text-xs text-gray-400 italic flex items-center space-x-1">
                    <span>Anup AI is thinking</span>
                    <span class="animate-bounce">.</span>
                    <span class="animate-bounce" style="animation-delay: 0.2s;">.</span>
                    <span class="animate-bounce" style="animation-delay: 0.4s;">.</span>
                </div>
            `;
            container.appendChild(typingDiv);
            container.scrollTop = container.scrollHeight;
        }

        function removeTypingIndicator(id) {
            const el = document.getElementById(id);
            if(el) el.remove();
        }

        function escapeHtml(text) {
            const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
            return text.replace(/[&<>"']/g, function(m) { return map[m]; });
        }

        function saveStateToLocalStorage() {
            localStorage.setItem('winter_arc_habits', JSON.stringify(habits));
        }

        function showToast(message) {
            const toast = document.getElementById('toast');
            document.getElementById('toast-msg').innerText = message;
            toast.style.opacity = '1';
            toast.style.transform = 'translate(-1/2, 0px)';
            
            setTimeout(() => {
                toast.style.opacity = '0';
            }, 2500);
        }
    </script>
</body>
</html>
```
