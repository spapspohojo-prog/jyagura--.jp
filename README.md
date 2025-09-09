<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>スロットマシンゲーム</title>
    <!-- Tailwind CSS を読み込み -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts からゲーム風フォントを読み込み -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
    <style>
        /* カスタムスタイル */
        body {
            font-family: 'Press+Start+2P', cursive, sans-serif;
            background-color: #1a202c; /* ダークテーマの背景色 */
            color: #e2e8f0; /* 明るいテキスト色 */
        }

        /* リールが回転しているときのアニメーション */
        .spinning {
            animation: spin-effect 0.1s linear infinite;
            filter: blur(4px);
        }

        @keyframes spin-effect {
            0% { transform: translateY(-10%); }
            50% { transform: translateY(10%); }
            100% { transform: translateY(-10%); }
        }
        
        /* ７とBARのスタイル */
        .seven { color: red; }
        .bar { 
            font-size: 50%; /* 文字サイズを親要素の半分に */
            line-height: 1; /* 行の高さを調整 */
            font-weight: bold;
            color: black;
            background: linear-gradient(to right, #444, #888, #444);
            padding: 0 5px;
            border-radius: 4px;
        }

        /* GOGOTHANCE ランプのスタイル */
        #gogo-lamp {
            width: 150px;
            height: 60px;
            background-color: #111;
            color: #333;
            border: 2px solid #444;
            border-radius: 10px;
            font-size: 1.2rem;
            font-weight: bold;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: all 0.2s ease-in-out;
            text-shadow: none;
        }

        .gogo-lamp-on {
            background-color: #fefcbf; /* Light yellow */
            color: #ef4444; /* Red */
            text-shadow: 0 0 5px #ef4444, 0 0 10px #ef4444;
            box-shadow: 0 0 10px #fefcbf, 0 0 20px #fefcbf, 0 0 40px #facc15;
            animation: pulse 1s infinite;
        }

        .gogo-lamp-on.in-bonus {
            animation: pulse-fast 0.5s infinite;
        }

        @keyframes pulse {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.05); opacity: 0.85; }
            100% { transform: scale(1); opacity: 1; }
        }

        @keyframes pulse-fast {
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.08); opacity: 0.9; }
            100% { transform: scale(1); opacity: 1; }
        }

        /* ボーナス中の演出 */
        .bonus-mode {
            animation: bonus-glow 1.5s infinite alternate;
        }

        @keyframes bonus-glow {
            from {
                box-shadow: 0 0 15px #fde047, 0 0 25px #fde047, 0 0 40px #fde047;
                border-color: #fde047;
            }
            to {
                box-shadow: 0 0 25px #facc15, 0 0 40px #facc15, 0 0 60px #facc15;
                border-color: #facc15;
            }
        }

        /* ボタンのスタイル */
        .btn {
            transition: all 0.2s ease-in-out;
        }
        .btn:hover:not(:disabled) {
            transform: translateY(-2px);
            box-shadow: 0 4px 8px rgba(0,0,0,0.3);
        }
        .btn:active:not(:disabled) {
            transform: translateY(0);
            box-shadow: none;
        }
        .btn:disabled {
            opacity: 0.5;
            cursor: not-allowed;
        }
        select {
             background-color: #2d3748;
             border-color: #4a5568;
        }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div id="game-container" class="w-full max-w-lg mx-auto bg-gray-800 rounded-2xl shadow-2xl p-4 md:p-6 border-4 border-purple-500">
        
        <h1 class="text-2xl md:text-3xl text-center font-bold mb-4 text-yellow-400 tracking-wider">
            SLOT MACHINE
        </h1>
        
        <!-- リール部分 -->
        <div class="flex justify-center space-x-2 md:space-x-4 mb-4 bg-gray-900 p-4 rounded-lg border-2 border-gray-700">
            <div id="reel1" class="text-6xl md:text-8xl p-2 bg-white rounded-lg text-black w-24 h-24 md:w-28 md:h-28 flex items-center justify-center">?</div>
            <div id="reel2" class="text-6xl md:text-8xl p-2 bg-white rounded-lg text-black w-24 h-24 md:w-28 md:h-28 flex items-center justify-center">?</div>
            <div id="reel3" class="text-6xl md:text-8xl p-2 bg-white rounded-lg text-black w-24 h-24 md:w-28 md:h-28 flex items-center justify-center">?</div>
        </div>

        <!-- GOGO ランプと情報表示部分 -->
        <div class="flex justify-between items-center mb-4">
            <div id="gogo-lamp">GOGOTHANCE</div>
            <div class="text-right text-sm md:text-lg">
                <div>設定: 
                    <select id="setting-select" class="rounded p-1">
                        <option value="1">1</option>
                        <option value="2">2</option>
                        <option value="3">3</option>
                        <option value="4">4</option>
                        <option value="5">5</option>
                        <option value="6">6</option>
                    </select>
                </div>
                <div>クレジット: <span id="credits-display" class="font-bold text-green-400">100</span></div>
            </div>
        </div>
        <p id="message-display" class="text-center text-lg h-8 text-yellow-300 mb-4"></p>

        <!-- 操作ボタン -->
        <div class="flex flex-col items-center space-y-3">
            <button id="spin-button" class="btn w-full bg-purple-600 hover:bg-purple-700 text-white font-bold py-3 px-4 rounded-lg text-xl shadow-lg">
                スピン (3)
            </button>
            <div class="flex w-full space-x-3">
                <button id="add-credits-button" class="btn w-1/2 bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-lg">
                    クレジットを追加
                </button>
                <button id="reset-button" class="btn w-1/2 bg-gray-500 hover:bg-gray-600 text-white font-bold py-2 px-4 rounded-lg">
                    リセット
                </button>
            </div>
        </div>
        
        <!-- ペイアウト表 -->
        <div class="mt-6 border-t-2 border-gray-600 pt-4 text-xs md:text-sm">
            <h2 class="text-center text-yellow-400 mb-2">PAYOUTS</h2>
            <div class="grid grid-cols-2 gap-x-4 gap-y-1">
                <div><span class="seven">７ ７ ７</span> : BIG (250)</div>
                <div><span class="seven">７ ７</span> <span class="bar">BAR</span> : REG (90)</div>
                <div>🍇 🍇 🍇 : 8</div>
                <div>🍒 🍒 🍒 : 10</div>
                <div>🔔 🔔 🔔 : 20</div>
                <div>🤡 🤡 🤡 : 20</div>
                <div>🦏 🦏 🦏 : もう一回</div>
            </div>
        </div>
    </div>

    <!-- クレジット購入モーダル -->
    <div id="purchase-modal" class="hidden fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center p-4 z-50">
        <div class="bg-gray-800 rounded-2xl shadow-2xl p-6 border-2 border-yellow-400 w-full max-w-sm text-center relative">
            <h2 class="text-2xl text-yellow-400 mb-4">クレジット購入</h2>
            <p class="mb-6">クレジットがなくなりました。<br>追加しますか？</p>
            <div class="space-y-3">
                <button class="btn w-full bg-blue-500 hover:bg-blue-600 text-white font-bold py-2 px-4 rounded-lg" data-amount="100">100 クレジット追加</button>
                <button class="btn w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded-lg" data-amount="500">500 クレジット追加</button>
                <button class="btn w-full bg-blue-700 hover:bg-blue-800 text-white font-bold py-2 px-4 rounded-lg" data-amount="1000">1000 クレジット追加</button>
            </div>
            <button id="close-modal-button" class="mt-6 text-gray-400 hover:text-white absolute top-2 right-4 text-2xl">&times;</button>
        </div>
    </div>


    <script>
        const reels = [
            document.getElementById('reel1'),
            document.getElementById('reel2'),
            document.getElementById('reel3')
        ];
        const spinButton = document.getElementById('spin-button');
        const resetButton = document.getElementById('reset-button');
        const creditsDisplay = document.getElementById('credits-display');
        const messageDisplay = document.getElementById('message-display');
        const settingSelect = document.getElementById('setting-select');
        const gogoLamp = document.getElementById('gogo-lamp');

        // 課金システム関連
        const addCreditsButton = document.getElementById('add-credits-button');
        const purchaseModal = document.getElementById('purchase-modal');
        const closeModalButton = document.getElementById('close-modal-button');
        const purchaseButtons = document.querySelectorAll('[data-amount]');

        // シンボル定義
        const symbols = ['🍇', '🍒', '🔔', '🦏', '🤡'];
        const specialSymbols = {
            seven: '<span class="seven">７</span>',
            bar: '<span class="bar">BAR</span>'
        };
        const allSymbolsForRandom = [...symbols, specialSymbols.seven, specialSymbols.bar];

        // 設定ごとの確率データ
        const settingsData = {
            1: { big: 1 / 273.1, reg: 1 / 439.8 },
            2: { big: 1 / 269.7, reg: 1 / 399.6 },
            3: { big: 1 / 269.7, reg: 1 / 331.0 },
            4: { big: 1 / 259.0, reg: 1 / 315.1 },
            5: { big: 1 / 259.0, reg: 1 / 255.0 },
            6: { big: 1 / 255.0, reg: 1 / 255.0 },
        };
        const smallWinProb = 1 / 7.5;

        // ゲームの状態
        let credits = 100;
        let isSpinning = false;
        const spinCost = 3;
        let currentSetting = 1;
        
        // ボーナスゲームの状態
        let inBonusMode = false;
        let bonusPayoutTarget = 0;
        let bonusPayoutAccumulated = 0;

        // サウンド関連
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        
        function playReelSpinSound() {
            if (!audioCtx || audioCtx.state === 'suspended') { audioCtx.resume(); }
            if (!audioCtx) return null;

            // ホワイトノイズを生成して回転音を模倣
            const bufferSize = audioCtx.sampleRate * 0.5; // 0.5秒のバッファ
            const buffer = audioCtx.createBuffer(1, bufferSize, audioCtx.sampleRate);
            const output = buffer.getChannelData(0);
            for (let i = 0; i < bufferSize; i++) {
                output[i] = Math.random() * 2 - 1;
            }

            const source = audioCtx.createBufferSource();
            source.buffer = buffer;
            source.loop = true;

            const gainNode = audioCtx.createGain();
            gainNode.gain.setValueAtTime(0.05, audioCtx.currentTime); // 音量を低めに設定
            
            source.connect(gainNode);
            gainNode.connect(audioCtx.destination);
            source.start();

            return { source, gainNode };
        }

        function stopReelSpinSound(nodes) {
            if (!nodes) return;
            // 音をスムーズにフェードアウトさせて停止
            const { source, gainNode } = nodes;
            gainNode.gain.exponentialRampToValueAtTime(0.0001, audioCtx.currentTime + 0.2);
            source.stop(audioCtx.currentTime + 0.2);
        }

        function playAnnounceSound() {
            if (!audioCtx || audioCtx.state === 'suspended') {
                 audioCtx.resume();
            }
            if (!audioCtx) return;
            const oscillator = audioCtx.createOscillator();
            const gainNode = audioCtx.createGain();
            oscillator.connect(gainNode);
            gainNode.connect(audioCtx.destination);

            oscillator.type = 'sine';
            oscillator.frequency.setValueAtTime(987.77, audioCtx.currentTime); // B5 note
            gainNode.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.5);
            oscillator.start(audioCtx.currentTime);
            oscillator.stop(audioCtx.currentTime + 0.5);
        }

        function playBonusSpinSound() {
            if (!audioCtx || audioCtx.state === 'suspended') { audioCtx.resume(); }
            if (!audioCtx) return;
            const o = audioCtx.createOscillator();
            const g = audioCtx.createGain();
            o.connect(g);
            g.connect(audioCtx.destination);
            o.type = 'square';
            g.gain.setValueAtTime(0.15, audioCtx.currentTime);
            
            [
                { freq: 523.25, time: 0 },    // C5
                { freq: 659.25, time: 0.1 },  // E5
                { freq: 783.99, time: 0.2 }   // G5
            ].forEach(note => {
                o.frequency.setValueAtTime(note.freq, audioCtx.currentTime + note.time);
            });

            o.start(audioCtx.currentTime);
            o.stop(audioCtx.currentTime + 0.3);
        }

        // 初期表示
        updateDisplay();

        function updateDisplay() {
            creditsDisplay.textContent = credits;
        }

        function showMessage(text, isWin = false) {
            messageDisplay.textContent = text;
            messageDisplay.classList.toggle('text-green-400', isWin);
            messageDisplay.classList.toggle('text-yellow-300', !isWin);
        }
        
        function determineOutcome() {
            if (inBonusMode) {
                return { type: 'Grape', reels: ['🍇', '🍇', '🍇'], payout: 8, message: "ボーナス中！" };
            }

            const setting = settingsData[currentSetting];
            const rand = Math.random();

            if (rand < setting.big) {
                return { type: 'BIG', reels: [specialSymbols.seven, specialSymbols.seven, specialSymbols.seven], payout: 250, message: "BIG BONUS!!" };
            }
            if (rand < setting.big + setting.reg) {
                const regReels = [specialSymbols.seven, specialSymbols.seven, specialSymbols.bar].sort(() => 0.5 - Math.random());
                return { type: 'REG', reels: regReels, payout: 90, message: "REGULAR BONUS!" };
            }

            const smallWins = [
                { type: 'Grape', reels: ['🍇', '🍇', '🍇'], payout: 8, message: "ぶどう！ 8クレジット GET!" },
                { type: 'Cherry', reels: ['🍒', '🍒', '🍒'], payout: 10, message: "チェリー！ 10クレジット GET!" },
                { type: 'Bell', reels: ['🔔', '🔔', '🔔'], payout: 20, message: "ベル！ 20クレジット GET!" },
                { type: 'Clown', reels: ['🤡', '🤡', '🤡'], payout: 20, message: "ピエロ！ 20クレジット GET!" },
                { type: 'Rhino', reels: ['🦏', '🦏', '🦏'], payout: 0, message: "サイ！ もう一回！", respin: true },
            ];

            if (Math.random() < smallWinProb) {
                return smallWins[Math.floor(Math.random() * smallWins.length)];
            }

            return { type: 'Loss', reels: generateLosingCombination(), payout: 0, message: "残念！" };
        }

        function generateLosingCombination() {
            let attempt;
            while (true) {
                attempt = [
                    allSymbolsForRandom[Math.floor(Math.random() * allSymbolsForRandom.length)],
                    allSymbolsForRandom[Math.floor(Math.random() * allSymbolsForRandom.length)],
                    allSymbolsForRandom[Math.floor(Math.random() * allSymbolsForRandom.length)]
                ];

                // 3つ揃いはハズレとする
                const isThreeOfAKind = attempt[0] === attempt[1] && attempt[1] === attempt[2];
                if (isThreeOfAKind) {
                    continue; // 3つ揃いなら再抽選
                }
                
                // REGの組み合わせ（7, 7, BAR）もハズレとする
                const sevens = attempt.filter(s => s === specialSymbols.seven).length;
                const bars = attempt.filter(s => s === specialSymbols.bar).length;
                if (sevens === 2 && bars === 1) {
                    continue; // REGの組み合わせなら再抽選
                }

                // 上記の当たりパターンでなければ、ハズレの組み合わせとして返す
                return attempt;
            }
        }
        
        function checkForGameOver() {
            if (credits < spinCost) {
                showMessage("クレジットがなくなりました！");
                spinButton.disabled = true;
                return true;
            }
            return false;
        }

        async function runSpin(isFreeSpin = false) {
            if (isSpinning) return;
            if (!inBonusMode && !isFreeSpin && checkForGameOver()) {
                return;
            }

            isSpinning = true;
            spinButton.disabled = true;
            settingSelect.disabled = true;
            
            showMessage("");
            if (!inBonusMode) {
                gogoLamp.classList.remove('gogo-lamp-on');
                gogoLamp.classList.remove('in-bonus');
            }

            if (!inBonusMode && !isFreeSpin) {
                credits -= spinCost;
                updateDisplay();
            }

            // 抽選
            const outcome = determineOutcome();

            // 先バレ抽選 (BIGかREG当選時、75%の確率で告知)
            if (!inBonusMode && (outcome.type === 'BIG' || outcome.type === 'REG') && Math.random() < 0.75) {
                gogoLamp.classList.add('gogo-lamp-on');
                playAnnounceSound();
            }

            // 回転音を開始
            const spinSound = playReelSpinSound();

            // アニメーション開始
            reels.forEach(reel => reel.classList.add('spinning'));
            const intervals = reels.map((reel) => setInterval(() => {
                reel.innerHTML = allSymbolsForRandom[Math.floor(Math.random() * allSymbolsForRandom.length)];
            }, 50)); // 回転中の絵柄の切り替え速度をアップ

            // リールを停止
            for (let i = 0; i < reels.length; i++) {
                // 各リールが停止するまでの時間を短縮
                await new Promise(resolve => setTimeout(resolve, 300 + i * 150));
                clearInterval(intervals[i]);
                reels[i].classList.remove('spinning');
                reels[i].innerHTML = outcome.reels[i];
            }

            // 回転音を停止
            stopReelSpinSound(spinSound);

            processResult(outcome);
        }

        function processResult(outcome) {
            if (inBonusMode) {
                // --- ボーナスゲーム中の処理 ---
                credits += outcome.payout;
                bonusPayoutAccumulated += outcome.payout;
                playBonusSpinSound();
                updateDisplay();

                const remaining = Math.max(0, bonusPayoutTarget - bonusPayoutAccumulated);

                if (remaining <= 0) {
                    // ボーナス終了
                    inBonusMode = false;
                    document.getElementById('game-container').classList.remove('bonus-mode');
                    gogoLamp.classList.remove('gogo-lamp-on', 'in-bonus');
                    showMessage(`ボーナス終了！ ${bonusPayoutAccumulated}枚獲得！`, true);

                    isSpinning = false;
                    spinButton.disabled = false;
                    settingSelect.disabled = false;
                    resetButton.disabled = false;
                    checkForGameOver(); // ボーナス後にクレジットが足りない場合をチェック
                } else {
                    // ボーナス継続
                    showMessage(`ボーナス中！ あと ${remaining} 枚`, true);
                    isSpinning = false;
                    spinButton.disabled = false; // 次のボーナススピンのためにボタンを有効化
                }
            } else if (outcome.type === 'BIG' || outcome.type === 'REG') {
                // --- ボーナスゲーム開始処理 ---
                inBonusMode = true;
                bonusPayoutTarget = outcome.payout;
                bonusPayoutAccumulated = 0;

                document.getElementById('game-container').classList.add('bonus-mode');
                gogoLamp.classList.add('gogo-lamp-on', 'in-bonus');
                showMessage(outcome.message + " ボーナス突入！", true);
                playAnnounceSound();
                updateDisplay();

                isSpinning = false;
                spinButton.disabled = false; // 最初のボーナススピンのためにボタンを有効化
                settingSelect.disabled = true;
                resetButton.disabled = true;
            } else {
                // --- 通常時の処理 ---
                credits += outcome.payout;
                showMessage(outcome.message, outcome.payout > 0 || outcome.respin);
                updateDisplay();

                if (outcome.respin) {
                    setTimeout(() => { isSpinning = false; runSpin(true); }, 1000);
                } else {
                    isSpinning = false;
                    settingSelect.disabled = false;
                    if (!checkForGameOver()) {
                        spinButton.disabled = false;
                    }
                }
            }
        }
        
        // --- イベントリスナー ---
        spinButton.addEventListener('click', () => runSpin());
        settingSelect.addEventListener('change', (e) => {
            currentSetting = e.target.value;
        });

        resetButton.addEventListener('click', () => {
            credits = 100;
            isSpinning = false;
            inBonusMode = false;
            bonusPayoutTarget = 0;
            bonusPayoutAccumulated = 0;
            document.getElementById('game-container').classList.remove('bonus-mode');
            spinButton.disabled = false;
            purchaseModal.classList.add('hidden');
            settingSelect.disabled = false;
            resetButton.disabled = false;
            gogoLamp.classList.remove('gogo-lamp-on', 'in-bonus');
            showMessage("リセットしました！");
            updateDisplay();
            reels.forEach(reel => reel.innerHTML = "?");
        });
        
        // 課金モーダル関連のイベントリスナー
        addCreditsButton.addEventListener('click', () => {
            purchaseModal.classList.remove('hidden');
        });
        closeModalButton.addEventListener('click', () => {
            purchaseModal.classList.add('hidden');
        });
        purchaseButtons.forEach(button => {
            button.addEventListener('click', () => {
                const amount = parseInt(button.dataset.amount, 10);
                credits += amount;
                updateDisplay();
                purchaseModal.classList.add('hidden');
                if (credits >= spinCost) {
                    spinButton.disabled = false;
                }
                showMessage(`${amount}クレジット追加しました！`, true);
            });
        });
        
        // ユーザー操作がないとAudioContextが開始できないブラウザ対策
        document.body.addEventListener('click', () => {
            if (audioCtx && audioCtx.state === 'suspended') {
                audioCtx.resume();
            }
        }, { once: true });

    </script>
</body>
</html>
