<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>ACLS Pro 2025 - Summary View</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        :root { --app-height: 100dvh; }
        body {
            height: var(--app-height);
            margin: 0;
            overflow: hidden;
            display: flex;
            justify-content: center; /* จัดให้อยู่กึ่งกลางแนวนอน */
            background-color: #1e293b; /* พื้นหลังด้านนอกให้เข้มขึ้นเล็กน้อย */
            font-family: system-ui, -apple-system, sans-serif;
        }

        /* ปรับความกว้างหลักที่นี่ */
        #main-app {
            width: 100%;
            max-width: 768px; /* ขยายความกว้างให้โปร่งขึ้น (ระดับ Tablet) */
            background-color: #0f172a;
            box-shadow: 0 0 50px rgba(0,0,0,0.5);
        }

        .fixed-panel { flex-shrink: 0; }

        /* ส่วน Log: ปรับความสูงให้สมดุลกับความกว้างที่เพิ่มขึ้น */
        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f8fafc;
            min-height: 200px;
            border-top: 2px solid #e2e8f0;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 48px; /* เพิ่มความสูงปุ่มนิดหน่อยให้กดง่ายขึ้น */
            font-size: 12px; /* ขยับขนาดตัวอักษรขึ้น */
            font-weight: 800;
            border-radius: 10px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); }

        #summary-screen {
            display: none;
            position: fixed;
            top: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 100%;
            max-width: 768px;
            height: 100%;
            background: white;
            z-index: 50;
            overflow-y: auto;
            padding: 30px;
        }

        /* ซ่อน Scrollbar */
        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body>

    <div id="main-app" class="flex flex-col h-full">
        <div class="fixed-panel p-5 bg-slate-900 text-white">
            <div class="flex justify-between items-center mb-4">
                <div>
                    <h1 class="text-2xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                    <p class="text-[10px] text-slate-500 font-bold mt-1 uppercase">Emergency Recording System</p>
                </div>
                <div class="text-right">
                    <p class="text-[10px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-3xl font-mono font-bold text-emerald-400">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-4 mb-4">
                <div class="bg-slate-800 p-4 rounded-2xl border-2 border-blue-500 text-center">
                    <p class="text-[10px] text-blue-400 font-bold uppercase mb-1">Rhythm Check In</p>
                    <p id="cpr-timer" class="text-4xl font-mono font-black">02:00</p>
                </div>
                <div class="bg-slate-800 p-4 rounded-2xl border border-slate-700 flex flex-col justify-center px-4">
                    <p class="text-yellow-500 text-[10px] font-bold uppercase mb-1">Current Guidance</p>
                    <div id="guidance-text" class="text-slate-300 text-sm italic font-medium leading-tight">Waiting for initiation...</div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-3">
                <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3.5 rounded-xl font-black text-xs uppercase shadow-lg">Start</button>
                <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3.5 rounded-xl font-black text-xs uppercase shadow-lg">Shockable</button>
                <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3.5 rounded-xl font-black text-xs uppercase shadow-lg">Non-Shock</button>
            </div>
        </div>

        <div class="fixed-panel p-4 bg-white border-b border-slate-200">
            <div class="grid grid-cols-3 gap-2.5 mb-2.5">
                <button onclick="recordAction(' CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700 hover:bg-emerald-50">START CPR</button>
                <button onclick="recordAction(' Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700 hover:bg-sky-50">IV/IO</button>
                <button onclick="recordAction(' IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700 hover:bg-blue-50">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-2.5 mb-2.5">
                <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md hover:bg-blue-700">EPINEPHRINE</button>
                <button onclick="recordAction(' Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600 hover:bg-purple-50">AMIO 300</button>
            </div>
            <div class="grid grid-cols-2 gap-2.5">
                <button onclick="recordAction(' Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">Amio 150</button>
                <button onclick="recordAction(' Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase tracking-tight">Adv. Airway</button>
            </div>
        </div>

        <div class="log-container no-scrollbar">
            <table class="w-full text-left border-separate border-spacing-y-2 px-5 py-2">
                <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                    <tr class="text-[10px] text-slate-400 font-black uppercase">
                        <th class="py-2 w-20 text-center border-r">Time</th>
                        <th class="py-2 px-4">Action Recorded</th>
                    </tr>
                </thead>
                <tbody id="log-body" class="text-xs font-bold text-slate-700">
                    </tbody>
            </table>
        </div>

        <div class="fixed-panel bg-white border-t p-4 flex items-center gap-4">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-5 rounded-2xl font-black text-sm uppercase shadow-lg active:bg-emerald-600">ROSC Achieved</button>
            <div class="flex gap-2 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-2 rounded-xl text-center shadow-inner">
                    <p class="text-[8px] uppercase font-bold leading-none mt-1">Shock</p>
                    <p id="dash-shock" class="text-2xl font-black">0</p>
                </div>
                <div class="bg-blue-600 text-white flex-1 py-2 rounded-xl text-center shadow-inner">
                    <p class="text-[8px] uppercase font-bold leading-none mt-1">Epi</p>
                    <p id="dash-epi" class="text-2xl font-black">0</p>
                </div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-5 rounded-2xl font-black text-[10px] uppercase active:bg-slate-900">Close Case</button>
        </div>
    </div>

    <div id="summary-screen" class="no-scrollbar">
        <div class="flex justify-between items-center border-b-4 border-slate-900 pb-5 mb-6">
            <h2 class="text-3xl font-black italic text-slate-900">CASE SUMMARY</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-6 py-2 rounded-full font-bold text-xs uppercase shadow-md">New Case</button>
        </div>

        <div class="grid grid-cols-2 gap-5 mb-8">
            <div class="bg-slate-100 p-5 rounded-3xl">
                <p class="text-slate-500 text-[11px] font-bold uppercase mb-1">Total Duration</p>
                <p id="sum-duration" class="text-3xl font-mono font-black text-slate-800">00:00</p>
            </div>
            <div class="bg-slate-100 p-5 rounded-3xl">
                <p class="text-slate-500 text-[11px] font-bold uppercase mb-1">Final Outcome</p>
                <p id="sum-outcome" class="text-2xl font-black text-emerald-600 italic">--</p>
            </div>
            <div class="bg-rose-100 p-5 rounded-3xl">
                <p class="text-rose-600 text-[11px] font-bold uppercase mb-1">Total Defib</p>
                <p id="sum-shock" class="text-4xl font-black text-rose-700">0</p>
            </div>
            <div class="bg-blue-100 p-5 rounded-3xl">
                <p class="text-blue-600 text-[11px] font-bold uppercase mb-1">Total Epi</p>
                <p id="sum-epi" class="text-4xl font-black text-blue-700">0</p>
            </div>
        </div>

        <h3 class="font-black text-slate-900 uppercase text-sm mb-4 flex items-center gap-2">
            <span class="w-3 h-6 bg-amber-500 block"></span> Timeline & Rhythm Log
        </h3>
        <div id="sum-timeline" class="space-y-3 mb-10"></div>

        <button onclick="document.getElementById('summary-screen').style.display='none'; document.getElementById('main-app').style.opacity = '1';" class="w-full py-5 border-4 border-slate-900 rounded-2xl font-black text-slate-900 uppercase text-sm hover:bg-slate-900 hover:text-white transition-colors">Back to Records</button>
    </div>

    <script>
        /* JavaScript คงเดิมตาม Logic ของคุณ */
        let totalSec = 0, cprSec = 120, isRunning = false, mainInterval = null;
        let defibCount = 0, epiCount = 0;
        let timelineData = [];

        function formatTime(s) {
            const m = Math.floor(s / 60).toString().padStart(2, '0');
            const sec = (s % 60).toString().padStart(2, '0');
            return `${m}:${sec}`;
        }

        function toggleStart() {
            const btn = document.getElementById('btn-start');
            if (!isRunning) {
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'bg-amber-600 py-3.5 rounded-xl font-black text-xs uppercase shadow-lg';
                if (totalSec === 0) recordAction(' Code Blue Activated');
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    if (cprSec <= 0) { cprSec = 120; recordAction(' Rhythm Check Due!'); }
                }, 1000);
            } else {
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'bg-emerald-600 py-3.5 rounded-xl font-black text-xs uppercase shadow-lg';
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            const g = document.getElementById('guidance-text');
            g.innerHTML = type === 'Shockable' ? '<b class="text-rose-500 uppercase text-lg">Shock 200J</b><br>then CPR' : '<b class="text-blue-500 uppercase text-lg">Give Epi</b><br>then CPR';
            recordAction(`Rhythm: ${type}`);
            cprSec = 120;
        }

        function recordDefib() {
            defibCount++; document.getElementById('dash-shock').innerText = defibCount;
            recordAction(` Defib #${defibCount} (200J)`);
            cprSec = 120;
        }

        function recordEpi() {
            epiCount++; document.getElementById('dash-epi').innerText = epiCount;
            recordAction(` Epinephrine #${epiCount}`);
        }

        function recordAction(msg) {
            const timeStr = formatTime(totalSec);
            timelineData.push({ time: timeStr, action: msg });
            
            const body = document.getElementById('log-body');
            const row = document.createElement('tr');
            row.className = "bg-white border-l-4 border-slate-300 shadow-sm";
            row.innerHTML = `<td class="text-center font-mono py-3 bg-slate-50 border-r text-slate-500">${timeStr}</td><td class="px-4 text-sm">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            document.getElementById('main-app').style.opacity = "0.1";
            const screen = document.getElementById('summary-screen');
            screen.style.display = 'block';

            document.getElementById('sum-duration').innerText = formatTime(totalSec);
            document.getElementById('sum-outcome').innerText = outcome === 'ROSC' ? 'ROSC ACHIEVED' : 'CASE CLOSED';
            document.getElementById('sum-outcome').className = outcome === 'ROSC' ? 'text-2xl font-black text-emerald-600 italic' : 'text-2xl font-black text-slate-500 italic';
            document.getElementById('sum-shock').innerText = defibCount;
            document.getElementById('sum-epi').innerText = epiCount;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-4 items-start border-b border-slate-100 pb-3";
                let textClass = "text-slate-700";
                if(item.action.includes('Rhythm')) textClass = "text-blue-700 font-black bg-blue-50 px-2 py-0.5 rounded";
                if(item.action.includes('Defib')) textClass = "text-rose-700 font-black bg-rose-50 px-2 py-0.5 rounded";
                div.innerHTML = `<span class="font-mono text-slate-400 text-sm w-14 pt-0.5">${item.time}</span><span class="${textClass}">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
