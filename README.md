# table
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>원소 배치 퍼즐</title>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;500;700&family=Space+Mono:wght@400;700&display=swap" rel="stylesheet">
    <style>
        :root {
            --bg: #111827;
            --panel: #1f2937;
            --panel-border: #374151;
            --slot-empty: #1a2332;
            --slot-empty-border: #2d4a7a;
            --filled-bg: #1e293b;
            --filled-color: #6b8ab5;
            --filled-border: #2a3f5f;
            --card-bg: #1d4ed8;
            --card-border: #3b82f6;
            --card-color: #ffffff;
            --card-placed-bg: #065f46;
            --card-placed-border: #10b981;
            --card-placed-color: #6ee7b7;
            --accent: #60a5fa;
            --gold: #fbbf24;
            --success: #10b981;
            --error: #ef4444;
            --text: #f1f5f9;
            --text-sub: #94a3b8;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; }

        body {
            font-family: 'Noto Sans KR', sans-serif;
            background-color: var(--bg);
            color: var(--text);
            min-height: 100vh;
            padding: 28px 20px 48px;
            overflow-x: auto;
        }

        /* ── 헤더 ── */
        .header {
            text-align: center;
            margin-bottom: 22px;
        }
        h1 {
            font-size: clamp(1.4rem, 2.5vw, 1.9rem);
            font-weight: 700;
            color: #fff;
            margin-bottom: 8px;
            letter-spacing: -0.01em;
        }
        .subtitle {
            font-size: 13.5px;
            color: var(--text-sub);
        }

        /* ── 진행 바 ── */
        .progress-wrap {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 12px;
            margin-bottom: 20px;
        }
        .progress-label {
            font-family: 'Space Mono', monospace;
            font-size: 12px;
            color: var(--text-sub);
        }
        .progress-track {
            width: 220px;
            height: 6px;
            background: #374151;
            border-radius: 99px;
            overflow: hidden;
        }
        .progress-fill {
            height: 100%;
            width: 0%;
            background: linear-gradient(90deg, #3b82f6, #10b981);
            border-radius: 99px;
            transition: width 0.35s ease;
        }
        .progress-count {
            font-family: 'Space Mono', monospace;
            font-size: 12px;
            color: #60a5fa;
            min-width: 44px;
        }

        .game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 16px;
        }

        /* ── 주기율표 ── */
        .periodic-table {
            display: grid;
            grid-template-columns: repeat(18, 42px);
            grid-template-rows: repeat(7, 42px);
            gap: 4px;
            background: var(--panel);
            border: 1px solid var(--panel-border);
            padding: 14px;
            border-radius: 12px;
            position: relative;
            box-shadow: 0 4px 24px rgba(0,0,0,0.4);
        }

        .slot {
            width: 42px;
            height: 42px;
            border-radius: 5px;
            display: flex;
            justify-content: center;
            align-items: center;
            font-family: 'Space Mono', monospace;
        }

        /* 배치해야 할 빈 슬롯 — 밝은 노란빛으로 눈에 띄게 */
        .slot.hint {
            background: #2d2a14;
            border: 1.5px dashed #a37c20;
            flex-direction: column;
            gap: 1px;
            transition: border-color 0.15s, background 0.15s;
        }
        .slot.hint:hover {
            border-color: #f59e0b;
            background: #3a3210;
        }
        .slot.hint.dragover {
            border-color: #fcd34d;
            background: #453b0d;
            border-style: solid;
        }
        .slot.hint .hint-num {
            font-size: 12px;
            line-height: 1;
            color: #c9a040;
            font-weight: 700;
        }
        .slot.hint .hint-name {
            font-size: 7px;
            line-height: 1;
            color: #8a6a28;
            font-family: 'Noto Sans KR', sans-serif;
        }

        /* 채워진 배경 원소 — 배경과 유사하게 눈에 안 띄게 */
        .slot.filled {
            background: #181f2e;
            border: 1px solid #1e2a3f;
            color: #36466a;
            font-size: 12px;
            font-weight: 700;
        }

        /* ── 카드 뱅크 ── */
        .card-bank-wrap { max-width: 820px; width: 100%; }
        .bank-label {
            font-size: 11px;
            letter-spacing: 0.12em;
            text-transform: uppercase;
            color: var(--text-sub);
            margin-bottom: 8px;
            padding-left: 2px;
        }
        .card-bank {
            display: flex;
            flex-wrap: wrap;
            justify-content: center;
            gap: 8px;
            min-height: 96px;
            background: var(--panel);
            border: 1px solid var(--panel-border);
            padding: 14px;
            border-radius: 10px;
        }

        /* ── 드래그 카드 ── */
        .card {
            width: 42px;
            height: 42px;
            background: var(--card-bg);
            border: 2px solid var(--card-border);
            color: var(--card-color);
            font-family: 'Space Mono', monospace;
            font-weight: 700;
            font-size: 12px;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 6px;
            cursor: grab;
            user-select: none;
            box-shadow: 0 2px 6px rgba(59,130,246,0.3);
            transition: transform 0.15s, box-shadow 0.15s, border-color 0.15s;
        }
        .card:hover {
            transform: translateY(-3px) scale(1.08);
            border-color: #93c5fd;
            box-shadow: 0 6px 16px rgba(59,130,246,0.45);
        }
        .card:active { cursor: grabbing; transform: scale(0.95); }

        /* 배치 완료 카드 */
        .card.placed-card {
            background: var(--card-placed-bg);
            border-color: var(--card-placed-border);
            color: var(--card-placed-color);
            cursor: default;
            box-shadow: 0 0 10px rgba(16,185,129,0.25);
        }
        .card.placed-card:hover { transform: none; }

        /* 오답 피드백 */
        .error-shake {
            animation: shake 0.35s ease;
            background: #7f1d1d !important;
            border-color: var(--error) !important;
            color: #fca5a5 !important;
        }
        @keyframes shake {
            0%,100% { transform: translateX(0); }
            20%      { transform: translateX(-6px); }
            40%      { transform: translateX(6px); }
            60%      { transform: translateX(-4px); }
            80%      { transform: translateX(4px); }
        }

        /* ── 성공 오버레이 ── */
        #success-screen {
            display: none;
            position: absolute;
            inset: 0;
            background: rgba(17,24,39,0.94);
            backdrop-filter: blur(4px);
            border-radius: 11px;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            gap: 10px;
            animation: fadeIn 0.4s ease;
        }
        @keyframes fadeIn {
            from { opacity: 0; } to { opacity: 1; }
        }
        .success-icon { font-size: 56px; animation: pop 0.5s 0.1s cubic-bezier(0.34,1.56,0.64,1) both; }
        @keyframes pop {
            from { opacity:0; transform: scale(0.3); }
            to   { opacity:1; transform: scale(1); }
        }
        #success-screen h2 {
            font-size: clamp(1.5rem, 3vw, 2.2rem);
            font-weight: 700;
            color: #fff;
        }
        #success-screen h2 em {
            font-style: normal;
            color: var(--gold);
        }
        .success-line {
            width: 48px; height: 2px;
            background: linear-gradient(90deg, transparent, var(--gold), transparent);
        }
        #success-screen p {
            font-size: 14px;
            color: var(--text-sub);
            text-align: center;
            line-height: 1.7;
            max-width: 300px;
        }
        .btn-restart {
            margin-top: 10px;
            padding: 10px 28px;
            background: #1d4ed8;
            border: 2px solid #3b82f6;
            color: #fff;
            font-family: 'Noto Sans KR', sans-serif;
            font-size: 14px;
            font-weight: 500;
            border-radius: 8px;
            cursor: pointer;
            transition: background 0.2s, transform 0.15s;
        }
        .btn-restart:hover { background: #2563eb; transform: translateY(-1px); }
    </style>
</head>
<body>

    <div class="header">
        <h1>🧪 원소 배치 퍼즐</h1>
        <p class="subtitle">원소 기호 카드를 드래그하여 주기율표의 올바른 위치에 놓으세요</p>
    </div>

    <div class="progress-wrap">
        <span class="progress-label">진행도</span>
        <div class="progress-track"><div class="progress-fill" id="progress-fill"></div></div>
        <span class="progress-count" id="progress-count">0 / 32</span>
    </div>

    <div class="game-container">
        <div class="periodic-table" id="periodic-table">
            <div id="success-screen">
                <div class="success-icon">🏆</div>
                <h2>퍼즐 <em>완성!</em></h2>
                <div class="success-line"></div>
                <p>32개 원소를 모두 정확하게<br>배치했습니다. 훌륭해요!</p>
                <button class="btn-restart" onclick="restartGame()">↺ 다시 도전하기</button>
            </div>
        </div>

        <div class="card-bank-wrap">
            <div class="bank-label">원소 카드 — 드래그하여 배치하세요</div>
            <div class="card-bank" id="card-bank"></div>
        </div>
    </div>

    <script>
        const targetSymbols = [
            'H','He','Li','Be','B','C','N','O','F','Ne',
            'Na','Mg','Al','Si','P','S','Cl','Ar','K','Ca',
            'Sc','Ti','Mn','Fe','Cu','Zn','Ag','I','Pt','Au','Hg','Pb'
        ];

        const elementInfo = {
            'H':  {z:1,  kr:'수소'},      'He': {z:2,  kr:'헬륨'},
            'Li': {z:3,  kr:'리튬'},      'Be': {z:4,  kr:'베릴륨'},
            'B':  {z:5,  kr:'붕소'},      'C':  {z:6,  kr:'탄소'},
            'N':  {z:7,  kr:'질소'},      'O':  {z:8,  kr:'산소'},
            'F':  {z:9,  kr:'플루오린'},  'Ne': {z:10, kr:'네온'},
            'Na': {z:11, kr:'나트륨'},    'Mg': {z:12, kr:'마그네슘'},
            'Al': {z:13, kr:'알루미늄'},  'Si': {z:14, kr:'규소'},
            'P':  {z:15, kr:'인'},        'S':  {z:16, kr:'황'},
            'Cl': {z:17, kr:'염소'},      'Ar': {z:18, kr:'아르곤'},
            'K':  {z:19, kr:'칼륨'},      'Ca': {z:20, kr:'칼슘'},
            'Sc': {z:21, kr:'스칸듐'},    'Ti': {z:22, kr:'타이타늄'},
            'V':  {z:23, kr:'바나듐'},    'Cr': {z:24, kr:'크로뮴'},
            'Mn': {z:25, kr:'망가니즈'},  'Fe': {z:26, kr:'철'},
            'Co': {z:27, kr:'코발트'},    'Ni': {z:28, kr:'니켈'},
            'Cu': {z:29, kr:'구리'},      'Zn': {z:30, kr:'아연'},
            'Ga': {z:31, kr:'갈륨'},      'Ge': {z:32, kr:'저마늄'},
            'As': {z:33, kr:'비소'},      'Se': {z:34, kr:'셀레늄'},
            'Br': {z:35, kr:'브로민'},    'Kr': {z:36, kr:'크립톤'},
            'Rb': {z:37, kr:'루비듐'},    'Sr': {z:38, kr:'스트론튬'},
            'Y':  {z:39, kr:'이트륨'},    'Zr': {z:40, kr:'지르코늄'},
            'Nb': {z:41, kr:'나이오븀'},  'Mo': {z:42, kr:'몰리브덴'},
            'Tc': {z:43, kr:'테크네튬'},  'Ru': {z:44, kr:'루테늄'},
            'Rh': {z:45, kr:'로듐'},      'Pd': {z:46, kr:'팔라듐'},
            'Ag': {z:47, kr:'은'},        'Cd': {z:48, kr:'카드뮴'},
            'In': {z:49, kr:'인듐'},      'Sn': {z:50, kr:'주석'},
            'Sb': {z:51, kr:'안티모니'},  'Te': {z:52, kr:'텔루륨'},
            'I':  {z:53, kr:'아이오딘'},  'Xe': {z:54, kr:'제논'},
            'Cs': {z:55, kr:'세슘'},      'Ba': {z:56, kr:'바륨'},
            'La': {z:57, kr:'란타넘'},    'Hf': {z:72, kr:'하프늄'},
            'Ta': {z:73, kr:'탄탈럼'},    'W':  {z:74, kr:'텅스텐'},
            'Re': {z:75, kr:'레늄'},      'Os': {z:76, kr:'오스뮴'},
            'Ir': {z:77, kr:'이리듐'},    'Pt': {z:78, kr:'백금'},
            'Au': {z:79, kr:'금'},        'Hg': {z:80, kr:'수은'},
            'Tl': {z:81, kr:'탈륨'},      'Pb': {z:82, kr:'납'},
            'Bi': {z:83, kr:'비스무트'},  'Po': {z:84, kr:'폴로늄'},
            'At': {z:85, kr:'아스타틴'},  'Rn': {z:86, kr:'라돈'},
            'Fr': {z:87, kr:'프랑슘'},    'Ra': {z:88, kr:'라듐'},
            'Ac': {z:89, kr:'악티늄'},    'Rf': {z:104,kr:'러더포듐'},
            'Db': {z:105,kr:'더브늄'},    'Sg': {z:106,kr:'시보귬'},
            'Bh': {z:107,kr:'보륨'},      'Hs': {z:108,kr:'하슘'},
            'Mt': {z:109,kr:'마이트너륨'},'Ds': {z:110,kr:'다름슈타튬'},
            'Rg': {z:111,kr:'뢴트게늄'}, 'Cn': {z:112,kr:'코페르니슘'},
            'Nh': {z:113,kr:'니호늄'},    'Fl': {z:114,kr:'플레로븀'},
            'Mc': {z:115,kr:'모스코븀'},  'Lv': {z:116,kr:'리버모륨'},
            'Ts': {z:117,kr:'테네신'},    'Og': {z:118,kr:'오가네손'},
        };

        const allElements = [
            {symbol:'H', r:1,c:1},{symbol:'He',r:1,c:18},
            {symbol:'Li',r:2,c:1},{symbol:'Be',r:2,c:2},{symbol:'B', r:2,c:13},{symbol:'C', r:2,c:14},{symbol:'N', r:2,c:15},{symbol:'O', r:2,c:16},{symbol:'F', r:2,c:17},{symbol:'Ne',r:2,c:18},
            {symbol:'Na',r:3,c:1},{symbol:'Mg',r:3,c:2},{symbol:'Al',r:3,c:13},{symbol:'Si',r:3,c:14},{symbol:'P', r:3,c:15},{symbol:'S', r:3,c:16},{symbol:'Cl',r:3,c:17},{symbol:'Ar',r:3,c:18},
            {symbol:'K', r:4,c:1},{symbol:'Ca',r:4,c:2},{symbol:'Sc',r:4,c:3},{symbol:'Ti',r:4,c:4},{symbol:'V', r:4,c:5},{symbol:'Cr',r:4,c:6},{symbol:'Mn',r:4,c:7},{symbol:'Fe',r:4,c:8},{symbol:'Co',r:4,c:9},{symbol:'Ni',r:4,c:10},{symbol:'Cu',r:4,c:11},{symbol:'Zn',r:4,c:12},{symbol:'Ga',r:4,c:13},{symbol:'Ge',r:4,c:14},{symbol:'As',r:4,c:15},{symbol:'Se',r:4,c:16},{symbol:'Br',r:4,c:17},{symbol:'Kr',r:4,c:18},
            {symbol:'Rb',r:5,c:1},{symbol:'Sr',r:5,c:2},{symbol:'Y', r:5,c:3},{symbol:'Zr',r:5,c:4},{symbol:'Nb',r:5,c:5},{symbol:'Mo',r:5,c:6},{symbol:'Tc',r:5,c:7},{symbol:'Ru',r:5,c:8},{symbol:'Rh',r:5,c:9},{symbol:'Pd',r:5,c:10},{symbol:'Ag',r:5,c:11},{symbol:'Cd',r:5,c:12},{symbol:'In',r:5,c:13},{symbol:'Sn',r:5,c:14},{symbol:'Sb',r:5,c:15},{symbol:'Te',r:5,c:16},{symbol:'I', r:5,c:17},{symbol:'Xe',r:5,c:18},
            {symbol:'Cs',r:6,c:1},{symbol:'Ba',r:6,c:2},{symbol:'La',r:6,c:3},{symbol:'Hf',r:6,c:4},{symbol:'Ta',r:6,c:5},{symbol:'W', r:6,c:6},{symbol:'Re',r:6,c:7},{symbol:'Os',r:6,c:8},{symbol:'Ir',r:6,c:9},{symbol:'Pt',r:6,c:10},{symbol:'Au',r:6,c:11},{symbol:'Hg',r:6,c:12},{symbol:'Tl',r:6,c:13},{symbol:'Pb',r:6,c:14},{symbol:'Bi',r:6,c:15},{symbol:'Po',r:6,c:16},{symbol:'At',r:6,c:17},{symbol:'Rn',r:6,c:18},
            {symbol:'Fr',r:7,c:1},{symbol:'Ra',r:7,c:2},{symbol:'Ac',r:7,c:3},{symbol:'Rf',r:7,c:4},{symbol:'Db',r:7,c:5},{symbol:'Sg',r:7,c:6},{symbol:'Bh',r:7,c:7},{symbol:'Hs',r:7,c:8},{symbol:'Mt',r:7,c:9},{symbol:'Ds',r:7,c:10},{symbol:'Rg',r:7,c:11},{symbol:'Cn',r:7,c:12},{symbol:'Nh',r:7,c:13},{symbol:'Fl',r:7,c:14},{symbol:'Mc',r:7,c:15},{symbol:'Lv',r:7,c:16},{symbol:'Ts',r:7,c:17},{symbol:'Og',r:7,c:18}
        ];

        const tableContainer = document.getElementById('periodic-table');
        const cardBank = document.getElementById('card-bank');
        const progressFill = document.getElementById('progress-fill');
        const progressCount = document.getElementById('progress-count');
        let correctPlacements = 0;
        const total = targetSymbols.length;

        function shuffleArray(arr) {
            for (let i = arr.length-1; i>0; i--) {
                const j = Math.floor(Math.random()*(i+1));
                [arr[i],arr[j]] = [arr[j],arr[i]];
            }
        }

        function updateProgress() {
            const pct = (correctPlacements / total) * 100;
            progressFill.style.width = pct + '%';
            progressCount.textContent = correctPlacements + ' / ' + total;
        }

        function initGame() {
            const successScreen = document.getElementById('success-screen');
            tableContainer.innerHTML = '';
            tableContainer.appendChild(successScreen);
            cardBank.innerHTML = '';
            correctPlacements = 0;
            updateProgress();
            successScreen.style.display = 'none';

            const cardsToPlay = [];

            allElements.forEach(el => {
                const slot = document.createElement('div');
                slot.style.gridRow = el.r;
                slot.style.gridColumn = el.c;

                if (targetSymbols.includes(el.symbol)) {
                    slot.classList.add('slot','hint');
                    slot.dataset.symbol = el.symbol;

                    const info = elementInfo[el.symbol];
                    if (info) {
                        const numSpan = document.createElement('span');
                        numSpan.classList.add('hint-num');
                        numSpan.textContent = info.z;
                        slot.appendChild(numSpan);
                        if (info.z > 20) {
                            const nameSpan = document.createElement('span');
                            nameSpan.classList.add('hint-name');
                            nameSpan.textContent = info.kr;
                            slot.appendChild(nameSpan);
                        }
                    }

                    slot.addEventListener('dragover', e => { e.preventDefault(); slot.classList.add('dragover'); });
                    slot.addEventListener('dragleave', () => slot.classList.remove('dragover'));
                    slot.addEventListener('drop', handleDrop);
                    cardsToPlay.push(el);
                } else {
                    slot.classList.add('slot','filled');
                    slot.textContent = el.symbol;
                }
                tableContainer.appendChild(slot);
            });

            shuffleArray(cardsToPlay);
            cardsToPlay.forEach(el => {
                const card = document.createElement('div');
                card.classList.add('card');
                card.textContent = el.symbol;
                card.draggable = true;
                card.id = 'card-' + el.symbol;
                card.addEventListener('dragstart', e => {
                    e.dataTransfer.setData('text/plain', el.symbol);
                    setTimeout(() => card.style.opacity = '0.35', 0);
                });
                card.addEventListener('dragend', () => card.style.opacity = '1');
                cardBank.appendChild(card);
            });
        }

        function handleDrop(e) {
            e.preventDefault();
            const slot = e.currentTarget;
            slot.classList.remove('dragover');
            const dragged = e.dataTransfer.getData('text/plain');
            const target = slot.dataset.symbol;
            const card = document.getElementById('card-' + dragged);

            if (dragged === target) {
                slot.innerHTML = '';
                card.classList.add('placed-card');
                card.draggable = false;
                card.style.opacity = '1';
                slot.appendChild(card);
                correctPlacements++;
                updateProgress();
                if (correctPlacements === total) {
                    setTimeout(() => { document.getElementById('success-screen').style.display = 'flex'; }, 400);
                }
            } else {
                card.style.opacity = '1';
                card.classList.add('error-shake');
                setTimeout(() => card.classList.remove('error-shake'), 400);
            }
        }

        function restartGame() { initGame(); }
        window.onload = initGame;
    </script>
</body>
</html>
