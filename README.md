<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>我的貼心月經記錄 App</title>
    <style>
        :root {
            --primary-color: #ff6b81;
            --secondary-color: #ffe4e6;
            --bg-color: #fff5f7;
            --text-color: #333;
        }
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            margin: 0;
            padding: 20px;
        }
        .container {
            max-width: 500px;
            margin: 0 auto;
            background: white;
            padding: 20px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(255, 107, 129, 0.15);
        }
        h1 {
            color: var(--primary-color);
            text-align: center;
            font-size: 1.5rem;
            margin-bottom: 20px;
        }
        .card {
            background: var(--secondary-color);
            padding: 15px;
            border-radius: 10px;
            margin-bottom: 20px;
            text-align: center;
        }
        .form-group {
            margin-bottom: 15px;
        }
        label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
            font-size: 0.9rem;
        }
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-sizing: border-box;
            font-size: 1rem;
        }
        .checkbox-group {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
        }
        .checkbox-group label {
            font-weight: normal;
            background: #fff;
            padding: 5px 10px;
            border-radius: 20px;
            border: 1px solid #ddd;
            font-size: 0.85rem;
            cursor: pointer;
        }
        .checkbox-group input {
            width: auto;
            margin-right: 5px;
        }
        button {
            background-color: var(--primary-color);
            color: white;
            border: none;
            padding: 12px;
            width: 100%;
            border-radius: 8px;
            font-size: 1rem;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.3s;
        }
        button:hover {
            opacity: 0.9;
        }
        .history-list {
            margin-top: 20px;
        }
        .history-item {
            background: #f9f9f9;
            padding: 12px;
            border-radius: 8px;
            margin-bottom: 10px;
            border-left: 4px solid var(--primary-color);
            font-size: 0.9rem;
            position: relative;
        }
        .history-item button.delete-btn {
            position: absolute;
            right: 10px;
            top: 10px;
            background: #ff4757;
            color: white;
            border: none;
            padding: 4px 8px;
            border-radius: 4px;
            font-size: 0.75rem;
            width: auto;
            cursor: pointer;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>🌸 我的經期小幫手</h1>
    
    <div class="card" id="status-card">
        <h3>下次經期預測</h3>
        <p id="prediction-text">記錄幾次生理期後，這裡將為您精準預測</p>
    </div>

    <form id="period-form">
        <input type="hidden" id="record-id">
        
        <div class="form-group">
            <label for="record-date">記錄日期</label>
            <input type="date" id="record-date" required>
        </div>

        <div class="form-group">
            <label for="flow">經血量</label>
            <select id="flow">
                <option value="無">無 / 結束</option>
                <option value="點狀出血">點狀出血 (Spotting)</option>
                <option value="量少">量少</option>
                <option value="量中">量中</option>
                <option value="量多">量多 🌊</option>
            </select>
        </div>

        <div class="form-group">
            <label>身體症狀 (可複選)</label>
            <div class="checkbox-group">
                <label><input type="checkbox" name="symptom" value="腹痛"> 腹痛</label>
                <label><input type="checkbox" name="symptom" value="頭痛"> 頭痛</label>
                <label><input type="checkbox" name="symptom" value="腰痠"> 腰痠</label>
                <label><input type="checkbox" name="symptom" value="胸脹"> 胸脹</label>
                <label><input type="checkbox" name="symptom" value="疲勞"> 疲勞</label>
                <label><input type="checkbox" name="symptom" value="青春痘"> 青春痘</label>
            </div>
        </div>

        <div class="form-group">
            <label for="mood">心情狀態</label>
            <select id="mood">
                <option value="平靜 😊">平靜 😊</option>
                <option value="煩躁 😤">煩躁 😤</option>
                <option value="憂鬱 💧">憂鬱 💧</option>
                <option value="疲憊 😴">疲憊 😴</option>
                <option value="開心 🥳">開心 🥳</option>
            </select>
        </div>

        <div class="form-group">
            <label for="note">備註</label>
            <textarea id="note" rows="2" placeholder="寫點筆記..."></textarea>
        </div>

        <button type="submit" id="submit-btn">儲存紀錄</button>
    </form>

    <div class="history-list">
        <h3>歷史紀錄</h3>
        <div id="history-container"></div>
    </div>
</div>

<script>
    document.getElementById('record-date').valueAsDate = new Date();
    
    let records = JSON.parse(localStorage.getItem('period_records')) || [];

    const form = document.getElementById('period-form');
    const historyContainer = document.getElementById('history-container');
    const predictionText = document.getElementById('prediction-text');

    form.addEventListener('submit', function(e) {
        e.preventDefault();
        
        const id = document.getElementById('record-id').value;
        const date = document.getElementById('record-date').value;
        const flow = document.getElementById('flow').value;
        const mood = document.getElementById('mood').value;
        const note = document.getElementById('note').value;
        
        const symptoms = Array.from(document.querySelectorAll('input[name="symptom"]:checked'))
                            .map(cb => cb.value);

        if (id) {
            // 修改紀錄
            records = records.map(r => r.id == id ? { id: Number(id), date, flow, symptoms, mood, note } : r);
            document.getElementById('record-id').value = '';
            document.getElementById('submit-btn').textContent = '儲存紀錄';
        } else {
            // 新增紀錄
            const newRecord = {
                id: Date.now(),
                date,
                flow,
                symptoms,
                mood,
                note
            };
            records.push(newRecord);
        }

        records.sort((a, b) => new Date(b.date) - new Date(a.date));
        localStorage.setItem('period_records', JSON.stringify(records));
        
        form.reset();
        document.getElementById('record-date').valueAsDate = new Date();
        render();
    });

    function deleteRecord(id) {
        if(confirm('確定要刪除這筆紀錄嗎？')) {
            records = records.filter(r => r.id !== id);
            localStorage.setItem('period_records', JSON.stringify(records));
            render();
        }
    }

    function editRecord(id) {
        const record = records.find(r => r.id === id);
        if (record) {
            document.getElementById('record-id').value = record.id;
            document.getElementById('record-date').value = record.date;
            document.getElementById('flow').value = record.flow;
            document.getElementById('mood').value = record.mood;
            document.getElementById('note').value = record.note || '';
            
            document.querySelectorAll('input[name="symptom"]').forEach(cb => {
                cb.checked = record.symptoms.includes(cb.value);
            });

            document.getElementById('submit-btn').textContent = '修改紀錄';
            window.scrollTo({ top: 0, behavior: 'smooth' });
        }
    }

    function render() {
        historyContainer.innerHTML = '';
        if (records.length === 0) {
            historyContainer.innerHTML = '<p style="text-align:center; color:#888;">尚無記錄</p>';
            predictionText.textContent = '記錄幾次生理期後，這裡將為您精準預測';
            return;
        }

        records.forEach(r => {
            const div = document.createElement('div');
            div.className = 'history-item';
            div.innerHTML = `
                <strong>📅 ${r.date}</strong> | 經量: ${r.flow} | 心情: ${r.mood}<br>
                <small>症狀: ${r.symptoms.length > 0 ? r.symptoms.join(', ') : '無'}</small><br>
                ${r.note ? `<small>備註: ${r.note}</small>` : ''}
                <button class="delete-btn" onclick="deleteRecord(${r.id})">刪除</button>
                <button class="delete-btn" style="right: 60px; background: #0984e3;" onclick="editRecord(${r.id})">修改</button>
            `;
            historyContainer.appendChild(div);
        });

        // 簡單預測邏輯 (抓取最近一次「量多」或有經期開始的日期間隔約 28 天作示範)
        const heavyRecords = records.filter(r => r.flow === '量多');
        if (heavyRecords.length >= 1) {
            const lastDate = new Date(heavyRecords[0].date);
            lastDate.setDate(lastDate.getDate() + 28);
            predictionText.textContent = `預計下次經期大約在: ${lastDate.toISOString().split('T')[0]}`;
        }
    }

    render();
</script>

</body>
</html>
