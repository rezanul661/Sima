{
  "rewrites": [
    { "source": "/(.*)", "destination": "/index.html" }
  ]
}<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Coin Earn App</title>
    <script src="https://telegram.org/js/telegram-web-app.js"></script>
    
    <!-- Monetag Interstitial - তোমার Zone ID 10939956 বসানো -->
    <script src='//libtl.com/sdk.js' data-zone='10939956' data-sdk='show_10939956'></script>
    
    <style>
        body { 
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; 
            text-align: center; 
            background: var(--tg-theme-bg-color, #ffffff); 
            color: var(--tg-theme-text-color, #000000);
            margin: 0;
            padding: 10px;
        }
        .card { 
            background: var(--tg-theme-secondary-bg-color, #f1f1f1); 
            padding: 20px; 
            border-radius: 12px; 
            margin: 15px 0;
        }
        button { 
            background: #28a745; 
            color: white; 
            border: none; 
            padding: 15px 30px; 
            font-size: 18px; 
            border-radius: 8px; 
            cursor: pointer;
            width: 90%;
        }
        button:disabled { 
            background: #6c757d;
            cursor: not-allowed;
        }
        input {
            padding: 12px;
            width: 85%;
            margin: 10px 0;
            border-radius: 8px;
            border: 1px solid #ccc;
            font-size: 16px;
        }
        h1 { margin: 5px 0; }
        h2, h3 { margin: 10px 0; }
    </style>
</head>
<body>
    <div class="card">
        <h2>আপনার ব্যালেন্স</h2>
        <h1><span id="coins">0</span> কয়েন</h1>
    </div>

    <div class="card">
        <h3>টাস্ক কমপ্লিট করুন</h3>
        <p>প্রতি অ্যাডে 50 কয়েন</p>
        <button id="rewardBtn" onclick="claimReward()">🎁 অ্যাড দেখে কয়েন নিন</button>
        <p id="timer" style="font-size:14px; color: #888;"></p>
    </div>

    <div class="card">
        <h3>উইথড্র করুন</h3>
        <p>মিনিমাম 10,000 কয়েন = 10 টাকা</p>
        <input id="bkashNumber" type="number" placeholder="বিকাশ নাম্বার দিন">
        <button onclick="withdraw()">উইথড্র রিকোয়েস্ট</button>
    </div>

    <script>
        let tg = window.Telegram.WebApp;
        tg.expand();
        tg.ready();

        let coins = parseInt(localStorage.getItem('coins')) || 0;
        document.getElementById('coins').innerText = coins;
        let rewardBtn = document.getElementById('rewardBtn');
        
        // 60 সেকেন্ড Cooldown - ব্যান সেফটির জন্য
        let lastAdTime = localStorage.getItem('lastAdTime') || 0;
        const COOLDOWN = 60000;
        
        updateTimer();

        function updateTimer() {
            let now = Date.now();
            let remaining = COOLDOWN - (now - lastAdTime);
            if (remaining > 0) {
                rewardBtn.disabled = true;
                document.getElementById('timer').innerText = `আবার অ্যাড দেখতে ${Math.ceil(remaining / 1000)} সেকেন্ড`;
                setTimeout(updateTimer, 1000);
            } else {
                rewardBtn.disabled = false;
                document.getElementById('timer').innerText = '';
            }
        }

        // Rewarded অ্যাড ফাংশন - Monetag Zone 10939956
        function claimReward() {
            let now = Date.now();
            if (now - lastAdTime < COOLDOWN) {
                return;
            }

            rewardBtn.disabled = true;
            rewardBtn.innerText = "লোড হচ্ছে...";
            
            if (typeof show_10939956 === 'function') {
                show_10939956().then(() => {
                    // ইউজার অ্যাড দেখা শেষ করেছে
                    coins += 50;
                    lastAdTime = Date.now();
                    localStorage.setItem('coins', coins);
                    localStorage.setItem('lastAdTime', lastAdTime);
                    document.getElementById('coins').innerText = coins;
                    tg.showAlert('অভিনন্দন! 50 কয়েন যোগ হয়েছে।');
                    updateTimer();
                }).catch(e => {
                    // ইউজার অ্যাড স্কিপ করেছে
                    tg.showAlert('অ্যাড দেখা সম্পূর্ণ হয়নি। কয়েন পেতে পুরো অ্যাড দেখুন।');
                }).finally(() => {
                    rewardBtn.innerText = "🎁 অ্যাড দেখে কয়েন নিন";
                    updateTimer();
                });
            } else {
                 tg.showAlert('অ্যাড এখনো রেডি না। 5 সেকেন্ড পর আবার চেষ্টা করুন।');
                 rewardBtn.innerText = "🎁 অ্যাড দেখে কয়েন নিন";
                 updateTimer();
            }
        }

        // Withdraw ফাংশন - বটে ডেটা পাঠাবে
        function withdraw() {
            let bkash = document.getElementById('bkashNumber').value;
            if (coins < 10000) {
                tg.showAlert('মিনিমাম 10,000 কয়েন লাগবে।');
                return;
            }
            if (bkash.length !== 11) {
                tg.showAlert('সঠিক 11 ডিজিটের বিকাশ নাম্বার দিন।');
                return;
            }
            
            // Telegram বটে ডেটা পাঠানো হচ্ছে
            tg.sendData(JSON.stringify({
                action: 'withdraw',
                coins: coins,
                bkash: bkash,
                user_id: tg.initDataUnsafe.user.id
            }));
            
            tg.showAlert('রিকোয়েস্ট পাঠানো হয়েছে। 24 ঘন্টার মধ্যে পেমেন্ট পাবেন।');
            coins = 0;
            localStorage.setItem('coins', 0);
            document.getElementById('coins').innerText = 0;
        }
    </script>
</body>
</html>
