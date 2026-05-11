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
            justify-content: center; /* จัดให้อยู่กึ่งกลางหน้าจอ */
            background-color: #0b0f1a;
            font-family: system-ui, -apple-system, sans-serif;
        }

        /* ปรับความกว้างเป็น 680px ตามสั่ง */
        #main-app {
            width: 100%;
            max-width: 680px; 
            background-color: #0f172a;
            display: flex;
            flex-direction: column;
            box-shadow: 0 0 40px rgba(0,0,0,0.6);
        }

        .fixed-panel { flex-shrink: 0; }

        .log-container {
            flex-grow: 1;
            overflow-y: auto;
            -webkit-overflow-scrolling: touch;
            background-color: #f1f5f9;
            min-height: 200px;
            border-top: 2px solid #e2e8f0;
        }

        .drug-btn {
            display: flex; align-items: center; justify-content: center; text-align: center;
            height: 44px; font-size: 11px; font-weight: 800;
            border-radius: 8px; transition: all 0.1s;
        }
        .drug-btn:active { transform: scale(0.96); }

        /* หน้าจอ Summary ปรับความกว้างให้เท่ากัน */
        #summary-screen {
            display: none;
            position: fixed;
            top: 0;
            left: 50%;
            transform: translateX(-50%);
            width: 100%;
            max-width: 680px;
            height: 100%;
            background: white;
            z-index: 50;
            overflow-y: auto;
            padding: 24px;
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }
    </style>
</head>
<body>

    <div id="main-app">
        <div class="fixed-panel p-4 bg-slate-900 text-white">
            <div class="flex justify-between items-center mb-3">
                <div>
                    <h1 class="text-2xl font-black text-red-500 italic leading-none">ACLS 2025</h1>
                    <p class="text-[9px] text-slate-500 font-bold mt-1 uppercase">Emergency Monitor</p>
                </div>
                <div class="text-right">
                    <p class="text-[8px] text-slate-500 font-bold uppercase">Total Time</p>
                    <p id="total-timer" class="text-2xl font-mono font-bold text-emerald-400">00:00</p>
                </div>
            </div>
            
            <div class="grid grid-cols-2 gap-3 mb-3">
                <div class="bg-slate-800 p-3 rounded-xl border-2 border-blue-500 text-center">
                    <p class="text-[9px] text-blue-400 font-bold uppercase">Rhythm Check</p>
                    <p id="cpr-timer" class="text-3xl font-mono font-black">02:00</p>
                </div>
                <div class="bg-slate-800 p-3 rounded-xl border border-slate-700 flex flex-col justify-center px-3">
                    <p class="text-yellow-500 text-[9px] font-bold uppercase">Guidance</p>
                    <div id="guidance-text" class="text-slate-300 text-[11px] italic leading-tight">System Ready</div>
                </div>
            </div>

            <div class="grid grid-cols-3 gap-2">
                <button onclick="toggleStart()" id="btn-start" class="bg-emerald-600 py-3 rounded-lg font-black text-[11px] uppercase shadow-lg">Start</button>
                <button onclick="setRhythm('Shockable')" class="bg-rose-700 py-3 rounded-lg font-black text-[11px] uppercase shadow-lg">Shockable</button>
                <button onclick="setRhythm('Non-Shockable')" class="bg-blue-700 py-3 rounded-lg font-black text-[11px] uppercase shadow-lg">Non-Shock</button>
            </div>
        </div>

        <div class="fixed-panel p-3 bg-white border-b border-slate-200">
            <div class="grid grid-cols-3 gap-2 mb-2">
                <button onclick="recordAction(' CPR Started')" class="drug-btn border-2 border-emerald-500 text-emerald-700">START CPR</button>
                <button onclick="recordAction(' Access IV/IO')" class="drug-btn border-2 border-sky-600 text-sky-700">IV/IO</button>
                <button onclick="recordAction(' IV Fluid')" class="drug-btn border-2 border-blue-400 text-blue-700">FLUID</button>
            </div>
            <div class="grid grid-cols-3 gap-2 mb-2">
                <button onclick="recordDefib()" class="drug-btn border-2 border-rose-500 text-rose-600 bg-rose-50 italic">DEFIB 200J</button>
                <button onclick="recordEpi()" class="drug-btn bg-blue-600 text-white shadow-md">EPINEPHRINE</button>
                <button onclick="recordAction(' Amiodarone 300mg')" class="drug-btn border-2 border-purple-500 text-purple-600">AMIO 300</button>
            </div>
            <div class="grid grid-cols-2 gap-2">
                <button onclick="recordAction(' Amiodarone 150mg')" class="drug-btn bg-slate-100 border border-slate-300 text-slate-700 uppercase">Amio 150</button>
                <button onclick="recordAction(' Adv. Airway')" class="drug-btn bg-orange-50 border border-orange-200 text-orange-700 italic uppercase">Adv. Airway</button>
            </div>
        </div>

        <div class="log-container no-scrollbar">
            <table class="w-full text-left border-separate border-spacing-y-1 px-4 py-2">
                <thead class="sticky top-0 bg-slate-50 z-10 shadow-sm">
                    <tr class="text-[9px] text-slate-400 font-black uppercase">
                        <th class="py-1 w-16 text-center border-r">Time</th>
                        <th class="py-1 px-3">Action Recorded</th>
                    </tr>
                </thead>
                <tbody id="log-body" class="text-[11px] font-bold text-slate-700"></tbody>
            </table>
        </div>

        <div class="fixed-panel bg-white border-t p-3 flex items-center gap-3">
            <button onclick="showSummary('ROSC')" class="flex-[2] bg-emerald-500 text-white py-4 rounded-xl font-black text-xs uppercase shadow-lg">ROSC Achieved</button>
            <div class="flex gap-2 flex-1">
                <div class="bg-rose-600 text-white flex-1 py-1.5 rounded-xl text-center"><p class="text-[7px] uppercase leading-none mt-1">Shock</p><p id="dash-shock" class="text-xl font-black">0</p></div>
                <div class="bg-blue-600 text-white flex-1 py-1.5 rounded-xl text-center"><p class="text-[7px] uppercase leading-none mt-1">Epi</p><p id="dash-epi" class="text-xl font-black">0</p></div>
            </div>
            <button onclick="showSummary('DEAD')" class="flex-1 bg-slate-800 text-white py-4 rounded-xl font-black text-[9px] uppercase">Close Case</button>
        </div>
    </div>

    <div id="summary-screen" class="no-scrollbar">
        <div class="flex justify-between items-center border-b-2 border-slate-900 pb-3 mb-5">
            <h2 class="text-2xl font-black italic text-slate-900">CASE SUMMARY</h2>
            <button onclick="location.reload()" class="bg-rose-500 text-white px-5 py-1.5 rounded-full font-bold text-xs uppercase">New Case</button>
        </div>

        <div class="grid grid-cols-2 gap-4 mb-6">
            <div class="bg-slate-100 p-4 rounded-2xl">
                <p class="text-slate-500 text-[10px] font-bold uppercase">Duration</p>
                <p id="sum-duration" class="text-2xl font-mono font-black text-slate-800">00:00</p>
            </div>
            <div class="bg-slate-100 p-4 rounded-2xl">
                <p class="text-slate-500 text-[10px] font-bold uppercase">Outcome</p>
                <p id="sum-outcome" class="text-xl font-black text-emerald-600 italic">--</p>
            </div>
            <div class="bg-rose-100 p-4 rounded-2xl text-center">
                <p class="text-rose-600 text-[10px] font-bold uppercase">Total Shock</p>
                <p id="sum-shock" class="text-3xl font-black text-rose-700">0</p>
            </div>
            <div class="bg-blue-100 p-4 rounded-2xl text-center">
                <p class="text-blue-600 text-[10px] font-bold uppercase">Total Epi</p>
                <p id="sum-epi" class="text-3xl font-black text-blue-700">0</p>
            </div>
        </div>

        <h3 class="font-black text-slate-900 uppercase text-xs mb-3 flex items-center gap-2">
            <span class="w-2 h-5 bg-amber-500 block"></span> Timeline
        </h3>
        <div id="sum-timeline" class="space-y-2 mb-8"></div>

        <button onclick="document.getElementById('summary-screen').style.display='none'; document.getElementById('main-app').style.opacity = '1';" class="w-full py-4 border-2 border-slate-900 rounded-xl font-black text-slate-900 uppercase text-xs">Back to View Records</button>
    </div>

    <script>
        /* JavaScript คงเดิมทุกประการ */
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
                isRunning = true; btn.innerText = 'PAUSE'; btn.className = 'bg-amber-600 py-3 rounded-lg font-black text-[11px] uppercase shadow-lg';
                if (totalSec === 0) recordAction(' Code Blue Activated');
                mainInterval = setInterval(() => {
                    totalSec++; cprSec--;
                    document.getElementById('total-timer').innerText = formatTime(totalSec);
                    document.getElementById('cpr-timer').innerText = formatTime(cprSec);
                    if (cprSec <= 0) { cprSec = 120; recordAction(' Rhythm Check Due!'); }
                }, 1000);
            } else {
                isRunning = false; btn.innerText = 'RESUME'; btn.className = 'bg-emerald-600 py-3 rounded-lg font-black text-[11px] uppercase shadow-lg';
                clearInterval(mainInterval);
            }
        }

        function setRhythm(type) {
            const g = document.getElementById('guidance-text');
            g.innerHTML = type === 'Shockable' ? '<b class="text-rose-500 uppercase">Shock 200J</b> -> CPR' : '<b class="text-blue-500 uppercase">Give Epi</b> -> CPR';
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
            row.innerHTML = `<td class="text-center font-mono py-2 bg-slate-50 border-r">${timeStr}</td><td class="px-3">${msg}</td>`;
            body.insertBefore(row, body.firstChild);
        }

        function showSummary(outcome) {
            if (isRunning) toggleStart();
            document.getElementById('main-app').style.opacity = "0.1";
            const screen = document.getElementById('summary-screen');
            screen.style.display = 'block';

            document.getElementById('sum-duration').innerText = formatTime(totalSec);
            document.getElementById('sum-outcome').innerText = outcome === 'ROSC' ? 'ROSC ACHIEVED' : 'CASE CLOSED';
            document.getElementById('sum-shock').innerText = defibCount;
            document.getElementById('sum-epi').innerText = epiCount;

            const timelineBox = document.getElementById('sum-timeline');
            timelineBox.innerHTML = '';
            timelineData.forEach(item => {
                const div = document.createElement('div');
                div.className = "flex gap-3 items-start border-b border-slate-100 pb-2";
                let textClass = "text-slate-700 text-xs";
                if(item.action.includes('Rhythm')) textClass = "text-blue-700 font-black bg-blue-50 px-1 rounded text-xs";
                if(item.action.includes('Defib')) textClass = "text-rose-700 font-black bg-rose-50 px-1 rounded text-xs";
                div.innerHTML = `<span class="font-mono text-slate-400 text-[11px] w-12 pt-1">${item.time}</span><span class="${textClass}">${item.action}</span>`;
                timelineBox.appendChild(div);
            });
        }
    </script>
</body>
</html>
