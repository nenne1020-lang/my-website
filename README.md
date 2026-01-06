<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>등급 시뮬레이터</title>
    <style>
        body { font-family: 'Malgun Gothic', sans-serif; background-color: #f1f3f5; display: flex; flex-direction: column; align-items: center; padding: 20px; color: #333; margin: 0; }
        .container { background: white; padding: 30px; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,0.1); width: 100%; max-width: 400px; text-align: center; }
        
        .info-panel { background: #212529; color: white; border-radius: 15px; padding: 20px; margin-bottom: 20px; text-align: left; }
        .info-label { font-size: 0.85rem; color: #adb5bd; }
        .info-value { font-size: 1.4rem; font-weight: bold; display: block; margin-bottom: 10px; }
        .powder { color: #ffd43b; }

        .result-card { min-height: 140px; display: flex; flex-direction: column; align-items: center; justify-content: center; border: 2px solid #e9ecef; border-radius: 20px; margin-bottom: 20px; background: #fff; }
        .grade-name { font-size: 3rem; font-weight: 900; margin: 0; }
        .score-badge { background: #495057; color: white; padding: 4px 12px; border-radius: 20px; font-size: 0.9rem; margin-top: 10px; }

        button { width: 100%; padding: 18px; font-size: 1.1rem; cursor: pointer; border: none; border-radius: 12px; font-weight: bold; transition: 0.15s; margin-bottom: 10px; }
        .btn-draw { background: #4c6ef5; color: white; }
        /* 1만회 버튼 색상 차별화 */
        .btn-draw-10000 { background: #5c7cfa; color: white; }
        .btn-reset { background: #ff6b6b; color: white; font-size: 0.8rem; padding: 10px; }
        button:active { transform: scale(0.98); }

        .SSS { color: #fab005; }
        .high { color: #228be6; }
        .mid { color: #40c057; }
    </style>
</head>
<body>

<div class="container">
    <div class="info-panel">
        <span class="info-label">누적 가루 소모량</span>
        <span class="info-value powder" id="powder">0</span>
        <span class="info-label">총 추출 횟수</span>
        <span class="info-value" id="count">0</span>
    </div>

    <div class="result-card" id="card">
        <div id="grade" class="grade-name">READY</div>
        <div id="score" class="score-badge">0 점</div>
    </div>

    <button class="btn-draw" onclick="draw(1000)">1,000회 추출</button>
    <button class="btn-draw btn-draw-10000" onclick="draw(10000)">10,000회 추출</button>
    <button class="btn-reset" onclick="reset()">초기화</button>
</div>

<script>
    const data = [
        { n: "SSS", s: 24, p: 0.00005 }, { n: "ASS", s: 23, p: 0.00025 },
        { n: "SAS", s: 22, p: 0.00015 }, { n: "BSS", s: 22, p: 0.00245 },
        { n: "SSA", s: 21, p: 0.00025 }, { n: "AAS", s: 21, p: 0.00075 }, { n: "CSS", s: 21, p: 0.00225 },
        { n: "ASA", s: 20, p: 0.00125 }, { n: "SBS", s: 20, p: 0.00245 }, { n: "BAS", s: 20, p: 0.00735 },
        { n: "SAA", s: 19, p: 0.00075 }, { n: "CAS", s: 19, p: 0.00675 }, { n: "BSA", s: 19, p: 0.01225 }, { n: "ABS", s: 19, p: 0.01225 },
        { n: "SCS", s: 18, p: 0.00235 }, { n: "AAA", s: 18, p: 0.00375 }, { n: "SSB", s: 18, p: 0.00490 }, { n: "CSA", s: 18, p: 0.01125 }, { n: "BBS", s: 18, p: 0.12005 },
        { n: "ACS", s: 17, p: 0.01175 }, { n: "SBA", s: 17, p: 0.01225 }, { n: "ASB", s: 17, p: 0.02450 }, { n: "BAA", s: 17, p: 0.03675 }, { n: "CBS", s: 17, p: 0.11025 },
        { n: "SAB", s: 16, p: 0.01470 }, { n: "CAA", s: 16, p: 0.03375 }, { n: "ABA", s: 16, p: 0.06125 }, { n: "BCS", s: 16, p: 0.11515 }, { n: "BSB", s: 16, p: 0.24010 },
        { n: "SSC", s: 15, p: 0.00480 }, { n: "SCA", s: 15, p: 0.01175 }, { n: "AAB", s: 15, p: 0.07350 }, { n: "CCS", s: 15, p: 0.10575 }, { n: "CSB", s: 15, p: 0.22050 }, { n: "BBA", s: 15, p: 0.60025 },
        { n: "ASC", s: 14, p: 0.02400 }, { n: "ACA", s: 14, p: 0.05875 }, { n: "SBB", s: 14, p: 0.24010 }, { n: "CBA", s: 14, p: 0.55125 }, { n: "BAB", s: 14, p: 0.72030 },
        { n: "SAC", s: 13, p: 0.01440 }, { n: "BSC", s: 13, p: 0.23520 }, { n: "BCA", s: 13, p: 0.57575 }, { n: "CAB", s: 13, p: 0.66150 }, { n: "ABB", s: 13, p: 1.20050 },
        { n: "AAC", s: 12, p: 0.07200 }, { n: "CSC", s: 12, p: 0.21600 }, { n: "SCB", s: 12, p: 0.23030 }, { n: "CCA", s: 12, p: 0.52875 }, { n: "BBB", s: 12, p: 11.76490 },
        { n: "SBC", s: 11, p: 0.23520 }, { n: "BAC", s: 11, p: 0.70560 }, { n: "ACB", s: 11, p: 1.15150 }, { n: "CBB", s: 11, p: 10.80450 },
        { n: "CAC", s: 10, p: 0.64800 }, { n: "ABC", s: 10, p: 1.17600 }, { n: "BCB", s: 10, p: 11.28470 },
        { n: "SCC", s: 9, p: 0.22560 }, { n: "CCB", s: 9, p: 10.36350 }, { n: "BBC", s: 9, p: 11.52480 },
        { n: "ACC", s: 8, p: 1.12800 }, { n: "CBC", s: 8, p: 10.58400 },
        { n: "BCC", s: 7, p: 11.05440 },
        { n: "CCC", s: 6, p: 10.15200 }
    ];

    let totalCount = 0;
    let totalPowder = 0;
    let bestResult = { n: "READY", s: 0 };

    // 횟수를 인자로 받도록 수정 (times)
    function draw(times) {
        let currentBest = { n: "", s: -1 };
        for (let i = 0; i < times; i++) {
            totalCount++;
            totalPowder += 30;
            const picked = getGrade();
            if (picked.s > currentBest.s) currentBest = picked;
        }

        // 이번 추출 결과 중 최고점이 기존 전체 최고점보다 높으면 갱신
        if (currentBest.s > bestResult.s) {
            bestResult = currentBest;
            updateDisplay();
        }
        updateStatus();
    }

    function getGrade() {
        const rand = Math.random() * 100;
        let cumulative = 0;
        for (const item of data) {
            cumulative += item.p;
            if (rand <= cumulative) return item;
        }
        return data[data.length - 1];
    }

    function updateDisplay() {
        const gradeEl = document.getElementById('grade');
        gradeEl.innerText = bestResult.n;
        gradeEl.className = "grade-name " + (bestResult.n === "SSS" ? "SSS" : (bestResult.s >= 20 ? "high" : (bestResult.s >= 15 ? "mid" : "")));
        document.getElementById('score').innerText = bestResult.s + " 점";
    }

    function updateStatus() {
        document.getElementById('powder').innerText = totalPowder.toLocaleString();
        document.getElementById('count').innerText = totalCount.toLocaleString();
    }

    function reset() {
        totalCount = 0;
        totalPowder = 0;
        bestResult = { n: "READY", s: 0 };
        document.getElementById('grade').innerText = "READY";
        document.getElementById('grade').className = "grade-name";
        document.getElementById('score').innerText = "0 점";
        updateStatus();
    }
</script>

</body>
</html>
