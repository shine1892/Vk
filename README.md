<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Precision Search Dashboard</title>
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&family=Poppins:wght@400;600;700&display=swap" rel="stylesheet">
    
    <style>
        /* --- THEME --- */
        :root { --primary: #00ff88; --secondary: #00ccff; --bg-color: #050505; --card-bg: rgba(20, 20, 20, 0.98); --border: rgba(255, 255, 255, 0.1); }
        * { margin: 0; padding: 0; box-sizing: border-box; font-family: 'Poppins', sans-serif; }
        body { 
            background-color: var(--bg-color);
            background-image: radial-gradient(circle at 10% 20%, rgba(0, 255, 136, 0.05) 0%, transparent 40%), radial-gradient(circle at 90% 80%, rgba(0, 204, 255, 0.05) 0%, transparent 40%);
            color: #fff; min-height: 100vh; display: flex; justify-content: center; align-items: center; padding: 15px; 
        }

        .dashboard { width: 100%; max-width: 500px; background: var(--card-bg); border: 1px solid var(--border); border-radius: 24px; padding: 25px; box-shadow: 0 10px 40px rgba(0,0,0,0.6); display: flex; flex-direction: column; gap: 15px; }
        header h1 { font-size: 24px; font-weight: 700; text-align: center; margin-bottom: 5px; }
        header span { background: linear-gradient(90deg, var(--primary), var(--secondary)); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }

        /* Fake Browser */
        .fake-browser { background: #000; border: 1px solid #333; border-radius: 12px; padding: 12px 15px; display: flex; align-items: center; box-shadow: inset 0 2px 5px rgba(0,0,0,0.5); }
        .typing-text { font-family: 'JetBrains Mono', monospace; font-size: 13px; color: var(--primary); white-space: nowrap; overflow: hidden; width: 100%; }

        /* Controls */
        .control-group { background: rgba(255,255,255,0.03); border: 1px solid var(--border); border-radius: 16px; padding: 15px; display: flex; flex-direction: column; gap: 12px; }
        input, select { width: 100%; background: #111; border: 1px solid #333; color: #fff; padding: 12px; border-radius: 8px; outline: none; font-size: 14px; }
        input:focus, select:focus { border-color: var(--primary); }
        .topic-input { color: var(--secondary) !important; font-weight: 600; }
        .settings-row { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; }
        .setting-item label { display: block; font-size: 10px; color: #888; text-transform: uppercase; margin-bottom: 4px; font-weight: 600; }
        .setting-item input { text-align: center; }

        /* Status */
        .status-container { background: rgba(0,0,0,0.2); border-radius: 8px; padding: 12px; text-align: center; border: 1px solid #222; }
        .progress-bar { height: 4px; background: #222; border-radius: 2px; margin-bottom: 8px; overflow: hidden; }
        .progress-fill { height: 100%; background: linear-gradient(90deg, var(--primary), var(--secondary)); width: 0%; transition: width 0.3s; }
        .status-text { font-size: 11px; color: #bbb; font-family: 'JetBrains Mono', monospace; min-height: 16px; }

        /* Buttons */
        .btn-large { width: 100%; padding: 16px; border: none; border-radius: 12px; background: linear-gradient(135deg, var(--primary), var(--secondary)); color: #000; font-weight: 700; font-size: 16px; cursor: pointer; display: flex; align-items: center; justify-content: center; gap: 8px; transition: transform 0.1s; }
        .btn-large:active { transform: scale(0.98); }
        .btn-stop { background: #ff4444; color: white; display: none; }
        
        .btn-indian { width: 100%; padding: 12px; background: transparent; border: 1px dashed #555; color: #888; border-radius: 8px; cursor: pointer; font-size: 12px; display: flex; align-items: center; justify-content: center; gap: 8px; transition: 0.3s; }
        .btn-indian.active { background: linear-gradient(90deg, #ff993333, #ffffff33, #13880833); border: 1px solid #138808; color: #fff; font-weight: bold; }

        #warning { display: none; background: #331111; color: #ff6666; padding: 10px; font-size: 11px; border-radius: 8px; border: 1px solid #ff4444; text-align: center; margin-bottom: 10px; }
    </style>
</head>
<body>

    <div class="dashboard">
        <header><h1>Research<span>Pro</span></h1></header>

        <div id="warning">⚠️ <strong>Popup Blocked:</strong> Please Allow Popups in your address bar.</div>

        <div class="fake-browser">
            <div class="typing-text" id="typing-area">System Idle...</div>
        </div>

        <div class="control-group">
            <input type="text" class="topic-input" id="topic" placeholder="Enter Topic (e.g. iPhone 16)">
            <select id="category">
                <option value="general">Category: General / Random</option>
                <option value="movies">Category: Movies & TV</option>
                <option value="tech">Category: Tech & Gadgets</option>
                <option value="gaming">Category: Gaming</option>
                <option value="finance">Category: Finance & Stocks</option>
                <option value="health">Category: Health & Fitness</option>
            </select>

            <div class="settings-row">
                <div class="setting-item">
                    <label>Limit</label>
                    <input type="number" id="limit" value="35">
                </div>
                <div class="setting-item">
                    <label>Min Sec</label>
                    <input type="number" id="min" value="5">
                </div>
                <div class="setting-item">
                    <label>Max Sec</label>
                    <input type="number" id="max" value="10">
                </div>
            </div>
        </div>

        <div class="status-container">
            <div class="progress-bar"><div class="progress-fill" id="progress"></div></div>
            <div class="status-text" id="status">Ready to Start</div>
        </div>

        <button class="btn-large" id="start-btn">START AUTOMATION</button>
        <button class="btn-large btn-stop" id="stop-btn">STOP & RESET</button>

        <button class="btn-indian" id="indian-btn">
            <img src="https://flagcdn.com/w40/in.png" style="height:12px"> INDIAN SEARCH MODE: OFF
        </button>
    </div>

    <script>
        // --- DATASETS (100+ Words) ---
        const dictionary = {
            movies: ["trailer","cast","review","release date","ending explained","box office","wiki","poster","tickets","director","sequel","plot","rating","rotten tomatoes","imdb","streaming","netflix","hulu","prime video","watch online","download","4k","soundtrack","filming locations","budget","easter eggs","fan theories","deleted scenes","bloopers","timeline","villain","hero","quotes","memes","wallpaper","merchandise","steelbook","pre order","showtimes","imax","dolby","runtime","parental guide","awards","oscars","red carpet","interview","cameos","post credit scene","analysis","video essay","script","concept art","vfx","stunts","costumes","flop","blockbuster","indie","short film","documentary","biopic","animated","remake","reboot","spinoff","crossover","universe","franchise","trilogy","saga","anthology","season 2","episode 1","finale"],
            tech: ["specs","price","review","vs","battery life","camera test","release date","unboxing","features","manual","issues","bugs","fix","screen replacement","battery cost","teardown","durability","water test","drop test","colors","storage","ram","processor","benchmark","gaming test","overheating","charging speed","wireless","case","cover","screen protector","best buy","amazon","sale","discount","trade in","warranty","service center","drivers","firmware","update","software","hardware","cpu","gpu","ssd","monitor","keyboard","mouse","headset","earbuds","smartwatch","router","5g","wifi 7","usb c","thunderbolt","bluetooth","setup guide","tips and tricks","hidden features"],
            gaming: ["gameplay","walkthrough","guide","wiki","tips","cheats","mods","hacks","glitches","speedrun","world record","achievements","trophies","collectibles","secrets","lore","ending","boss fight","best weapons","build","tier list","meta","patch notes","update","dlc","season pass","skins","loot","server status","ping","fps boost","optimization","requirements","download","install","size","steam","epic","xbox","ps5","switch","controller","settings","sensitivity","crosshair","graphics","ray tracing","esports","tournament","twitch","discord","reviews","release date","beta","early access"],
            finance: ["stock price","market cap","chart","forecast","prediction","buy or sell","rating","news","earnings","revenue","profit","report","balance sheet","debt","dividends","yield","payout","split","buyback","ipo","listing","competitors","industry","sector","index","sp500","crypto","bitcoin","ethereum","bull run","bear market","crash","recession","inflation","rates","taxes","trading","investing","portfolio","broker","fees","account","scam","sec filing","insider","ceo","salary","jobs","hiring","careers","analysis","technical","fundamental"],
            health: ["symptoms","causes","treatment","cure","medicine","side effects","dosage","precautions","warnings","interactions","benefits","risks","diagnosis","test","cost","insurance","hospital","doctor","appointment","natural","herbal","yoga","exercise","diet","nutrition","vitamins","supplements","calories","weight loss","bmi","calculator","guidelines","research","statistics","news","pandemic","virus","infection","pain","relief","recovery","rehab","mental health","anxiety","depression","sleep","energy","immunity","skin","hair"],
            general: ["news","latest","images","video","wiki","biography","age","net worth","family","history","facts","trivia","quiz","meaning","definition","synonyms","translate","examples","types","list","top 10","best","ranking","comparison","difference","guide","how to","diy","ideas","design","trends","popular","viral","meme","joke","quotes","lyrics","song","book","app","login","contact","address","phone","map","directions","reviews","ratings"]
        };

        const indianSuffixes = ["price in India","news India","hindi dubbed","box office India","flipkart","amazon india","jio","cricket","ipl","bollywood","bangalore","delhi","mumbai"];

        // --- ENGINE LOGIC ---
        const params = new URLSearchParams(window.location.search);
        const isRunning = params.get('run') === '1';
        const currentCount = parseInt(params.get('c')) || 0;
        
        // --- PRECISE VALUES (No Defaults/Tuning) ---
        const limit = parseInt(params.get('l')) || 35;
        const minDelay = parseInt(params.get('mi')) || 5; 
        const maxDelay = parseInt(params.get('ma')) || 10;
        
        const topicVal = params.get('t') || "";
        const catVal = params.get('cat') || "general";
        const isIndian = params.get('ind') === '1';

        // Elements
        const ui = {
            topic: document.getElementById('topic'),
            cat: document.getElementById('category'),
            limit: document.getElementById('limit'),
            min: document.getElementById('min'),
            max: document.getElementById('max'),
            status: document.getElementById('status'),
            progress: document.getElementById('progress'),
            typing: document.getElementById('typing-area'),
            start: document.getElementById('start-btn'),
            stop: document.getElementById('stop-btn'),
            indianBtn: document.getElementById('indian-btn'),
            warning: document.getElementById('warning')
        };

        function generateQuery() {
            const list = dictionary[catVal] || dictionary.general;
            let suffix = list[Math.floor(Math.random() * list.length)];
            if (isIndian && Math.random() > 0.4) {
                suffix = indianSuffixes[Math.floor(Math.random() * indianSuffixes.length)];
            }
            if (!topicVal) {
                const randomTopics = ["Weather", "Tech", "Movies", "Crypto", "Health", "Sports"];
                const rand = randomTopics[Math.floor(Math.random() * randomTopics.length)];
                return `${rand} ${suffix} ${Math.floor(Math.random()*100)}`;
            }
            return `${topicVal} ${suffix}`;
        }

        // --- CORE EXECUTION ---
        function runAutomation() {
            // 1. Lock UI
            ui.topic.value = topicVal; ui.cat.value = catVal;
            ui.limit.value = limit; ui.min.value = minDelay; ui.max.value = maxDelay;
            [ui.limit, ui.min, ui.max, ui.topic, ui.cat].forEach(e => e.disabled = true);
            ui.start.style.display = 'none';
            ui.stop.style.display = 'flex';
            if(isIndian) {
                ui.indianBtn.classList.add('active');
                ui.indianBtn.innerHTML = `<img src="https://flagcdn.com/w40/in.png" style="height:12px"> INDIAN SEARCH MODE: ON`;
            }

            // 2. Update Status
            ui.progress.style.width = `${(currentCount / limit) * 100}%`;
            
            if (currentCount >= limit) {
                ui.status.innerText = "✅ TARGET COMPLETE";
                return;
            }

            // 3. GENERATE & OPEN IMMEDIATELY (No Blocking Waits)
            const query = generateQuery();
            document.title = `(${currentCount}/${limit}) Running...`;
            ui.typing.innerText = `Searching: ${query}`; // Instant visual

            const tab = window.open(`https://www.bing.com/search?q=${encodeURIComponent(query)}`, '_blank');
            
            if (!tab) {
                ui.warning.style.display = "block";
                ui.status.innerText = "❌ BLOCKED";
                return;
            }

            // 4. PRECISE TIMER (Date-Based for Accuracy)
            // Calculate delay strictly between Min and Max
            const delayMs = Math.floor(Math.random() * (maxDelay - minDelay + 1) + minDelay) * 1000;
            const targetTime = Date.now() + delayMs;

            const interval = setInterval(() => {
                const now = Date.now();
                const remaining = Math.ceil((targetTime - now) / 1000);
                
                ui.status.innerText = `Waiting ${remaining}s...`;

                if (now >= targetTime) {
                    clearInterval(interval);
                    // RELOAD
                    const nextUrl = `?run=1&c=${currentCount+1}&l=${limit}&mi=${minDelay}&ma=${maxDelay}&t=${encodeURIComponent(topicVal)}&cat=${catVal}&ind=${isIndian?'1':'0'}`;
                    window.location.href = nextUrl;
                }
            }, 100); // Check every 100ms for responsiveness
        }

        // --- STARTUP ---
        if (isRunning) {
            runAutomation();
        }

        // --- HANDLERS ---
        ui.start.addEventListener('click', () => {
            const l = ui.limit.value;
            const mi = ui.min.value;
            const ma = ui.max.value;
            const t = encodeURIComponent(ui.topic.value);
            const c = ui.cat.value;
            const i = ui.indianBtn.classList.contains('active') ? '1' : '0';
            window.location.href = `?run=1&c=0&l=${l}&mi=${mi}&ma=${ma}&t=${t}&cat=${c}&ind=${i}`;
        });

        ui.stop.addEventListener('click', () => {
            window.location.href = window.location.pathname;
        });

        ui.indianBtn.addEventListener('click', () => {
            if(ui.indianBtn.classList.contains('active')) {
                ui.indianBtn.classList.remove('active');
                ui.indianBtn.innerHTML = `<img src="https://flagcdn.com/w40/in.png" style="height:12px"> INDIAN SEARCH MODE: OFF`;
            } else {
                ui.indianBtn.classList.add('active');
                ui.indianBtn.innerHTML = `<img src="https://flagcdn.com/w40/in.png" style="height:12px"> INDIAN SEARCH MODE: ON`;
            }
        });

    </script>
</body>
</html>
