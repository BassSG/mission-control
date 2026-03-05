# 🎯 Mission Control v3 - Implementation Plan

## Problem
App ปัจจุบันแค่ "แสดง" แต่ไม่ได้ "ควบคุม"

---

## 1. Jarvis Command Center 🤖

### Current State
- แสดง UI แชท + Quick Commands
- กดส่งแล้วแค่แสดงข้อความ Static ไม่ได้เรียก AI จริง

### Implementation

**Frontend:**
```javascript
// เรียก AI ผ่าน OpenClaw Sessions API
async function sendCommand() {
    const input = document.getElementById('jarvis-input');
    const cmd = input.value.trim();
    if (!cmd) return;
    
    // แสดง User message
    addChatMessage('👤 คุณเบส', cmd);
    
    // เรียก Jarvis Session จริง
    const response = await fetch('/api/sessions/agent:main:jarvis:direct/turn', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({message: cmd, model: 'minimax-portal/MiniMax-M2.5'})
    });
    
    const data = await response.json();
    addChatMessage('🤖 Jarvis', data.reply);
}
```

**Backend (OpenClaw):**
- ใช้ `sessions/agent:main:jarvis:direct/turn` endpoint ที่มีอยู่แล้ว
- Model: Minimax-M2.5 (low thinking เพื่อประหยัด)

**Quick Commands ใหม่:**
- 📊 ราคาทอง → `fetchPrices()` → แสดงผล
- 📈 ราคา BTC → `fetchPrices()` → แสดงผล  
- 📋 Tasks → `goPage('tasks')`
- 📅 Calendar → `fetchCalendar()`

---

## 2. Trading Control 💰

### Current State
- แสดงราคา Hardcoded: $5,171.50, $84,230
- กด Refresh แค่ Fake loading → แสดงค่าเดิม

### Implementation

**FMP API Integration:**
```javascript
// ดึงราคาจริงจาก FMP
const FMP_API_KEY = 'AIzaSyD...'; // from TOOLS.md

async function fetchPrices() {
    const symbols = ['XAUUSD', 'BTCUSD', 'USDTHB'];
    const results = {};
    
    for (const sym of symbols) {
        const res = await fetch(
            `https://financialmodelingprep.com/api/v3/quote/${sym}?apikey=${FMP_API_KEY}`
        );
        const data = await res.json();
        if (data[0]) {
            results[sym] = {
                price: data[0].price,
                change: data[0].change,
                changePercent: data[0].changesPercentage
            };
        }
    }
    
    updatePriceUI(results);
}
```

**Price Alert (localStorage):**
```javascript
// ตั้ง Alert
function setPriceAlert(symbol, targetPrice, condition) {
    const alerts = JSON.parse(localStorage.getItem('priceAlerts') || '[]');
    alerts.push({
        id: Date.now(),
        symbol, targetPrice, condition, // 'above' or 'below'
        createdAt: new Date().toISOString()
    });
    localStorage.setItem('priceAlerts', JSON.stringify(alerts));
}

// ตรวจสอบทุกครั้งที่ดึงราคา
function checkAlerts(prices) {
    const alerts = JSON.parse(localStorage.getItem('priceAlerts') || '[]');
    alerts.forEach(alert => {
        const price = prices[alert.symbol];
        if (!price) return;
        
        const triggered = alert.condition === 'above' 
            ? price > alert.targetPrice 
            : price < alert.targetPrice;
            
        if (triggered) {
            // แสดง Notification
            notifyUser(`🔔 ${alert.symbol} ${alert.condition} $${alert.targetPrice}! Current: $${price}`);
        }
    });
}
```

**UI Updates:**
- เพิ่มปุ่ม "🔔 Set Alert" ในแต่ละ Card
- แสดง List ของ Active Alerts

---

## 3. Task Manager 📋

### Current State
- Tasks อยู่ใน Array ตรงๆ → หายทุกครั้งที่ Refresh

### Implementation (localStorage)

```javascript
// Load Tasks
function loadTasks() {
    return JSON.parse(localStorage.getItem('tasks') || '[]');
}

// Save Tasks  
function saveTasks(tasks) {
    localStorage.setItem('tasks', JSON.stringify(tasks));
}

// Add Task
function addTask(title, priority = 'med') {
    const tasks = loadTasks();
    tasks.push({
        id: Date.now(),
        title,
        priority,
        due: new Date().toISOString(),
        done: false,
        createdAt: new Date().toISOString()
    });
    saveTasks(tasks);
    renderTasks();
}

// Toggle Done
function toggleTask(id) {
    const tasks = loadTasks();
    const task = tasks.find(t => t.id === id);
    if (task) {
        task.done = !task.done;
        saveTasks(tasks);
        renderTasks();
    }
}

// Delete Task
function deleteTask(id) {
    const tasks = loadTasks().filter(t => t.id !== id);
    saveTasks(tasks);
    renderTasks();
}

// UI: เพิ่ม Swipe to Delete หรือ ปุ่ม Delete
```

**Initial Data (ถ้า localStorage ว่าง):**
```javascript
const defaultTasks = [
    {id: 1, title: 'ทำรายงาน Trading Weekly', priority: 'high', due: 'Today', done: false},
    {id: 2, title: 'ตรวจสอบ Cron Jobs', priority: 'med', due: 'Tomorrow', done: false}
];
```

---

## 4. Calendar Control 📅

### Current State
- แสดง Calendar Grid Static + Event List Hardcoded

### Implementation

**Google Calendar API (via Gog skill):**
```javascript
// ดึง Events จริง
async function fetchCalendar() {
    // ใช้ Gog skill - Calendar API
    const response = await fetch('/api/gog/calendar/events', {
        method: 'POST',
        headers: {'Content-Type': 'application/json'},
        body: JSON.stringify({
            account: 'bass1135@gmail.com',
            timeMin: new Date().toISOString(),
            timeMax: new Date(Date.now() + 7*24*60*60*1000).toISOString(),
            maxResults: 10
        })
    });
    
    const events = await response.json();
    renderCalendarEvents(events);
}

function renderCalendarEvents(events) {
    const container = document.getElementById('calendar-events');
    container.innerHTML = events.map(e => `
        <div class="task-item">
            <div class="task-content">
                <div class="task-title">${e.summary}</div>
                <div class="task-meta">${formatTime(e.start.dateTime || e.start.date)}</div>
            </div>
        </div>
    `).join('');
}
```

**Fallback (ถ้า Google ไม่ได้เชื่อม):**
- แสดงข้อความ "กรุณาเชื่อมม Google Calendar ก่อน" + ลิงก์ไป Gog setup

---

## 5. Cron Jobs Monitor ⏰

### Current State
- Cron Jobs อยู่ใน Array ตรงๆ → ไม่ตรงกับความจริง

### Implementation

**อ่านจาก jobs.json จริง:**
```javascript
async function loadCronJobs() {
    // อ่านจาก memory/jobs_config_*.json ล่าสุด
    const files = await fetch('/api/files/memory/jobs_config_*.json');
    
    // หรือเรียก Gateway API
    const response = await fetch('/api/cron/jobs');
    const data = await response.json();
    
    return data.jobs.map(job => ({
        name: job.name,
        schedule: job.schedule.expr,
        enabled: job.enabled,
        nextRun: new Date(job.state.nextRunAtMs).toLocaleString('th-TH'),
        status: job.enabled ? 'ok' : 'off'
    }));
}

function renderCronJobs() {
    loadCronJobs().then(jobs => {
        document.getElementById('cron-list').innerHTML = jobs.map(j => `
            <div class="big-card">
                <h3>${j.name}</h3>
                <div class="change">${j.status === 'ok' ? '✅ Active' : '⛔ Disabled'}</div>
                <div class="task-meta">
                    Schedule: ${j.schedule} | Next: ${j.nextRun}
                </div>
            </div>
        `).join('');
    });
}
```

---

## 6. System Status 💻

### Current State
- แสดง Status Static: "OK", "Ready"

### Implementation

**ดึงจาก Gateway API:**
```javascript
async function fetchSystemStatus() {
    const endpoints = {
        gateway: '/api/gateway/status',
        openclaw: '/api/openclaw/version',
        minimax: '/api/llm/models', 
        brave: '/api/search/health',
        fmp: '/api/fmp/quote/XAUUSD', // Test API key
        calendar: '/api/gog/calendar/status'
    };
    
    const results = {};
    
    for (const [key, url] of Object.entries(endpoints)) {
        try {
            const res = await fetch(url, {method: 'GET', timeout: 5000});
            results[key] = res.ok ? 'OK' : 'Error';
        } catch (e) {
            results[key] = 'Offline';
        }
    }
    
    // Update UI
    document.getElementById('sys-gateway').textContent = results.gateway;
    document.getElementById('sys-openclaw').textContent = `v${results.openclaw}`;
    document.getElementById('sys-minimax').textContent = results.minimax === 'OK' ? 'Ready' : results.minimax;
    // ...
}
```

---

## Summary Table

| Feature | Current | After | Data Source |
|---------|---------|-------|-------------|
| Jarvis | Static msg | Real AI response | OpenClaw Sessions API |
| Trading | Hardcoded | Live prices | FMP API |
| Price Alert | ❌ | ✅ (localStorage) | localStorage |
| Tasks | In-memory | Persistent | localStorage |
| Calendar | Hardcoded | Real events | Google Calendar API |
| Cron Jobs | Static list | Real jobs | jobs.json / Gateway |
| System Status | Static | Live status | Gateway APIs |

---

## Priority & Effort

1. **Task Manager** - Easy, High Impact (Persistence)
2. **Trading Control** - Medium, High Impact (Real prices)
3. **System Status** - Easy, Medium Impact (Gateway)
4. **Cron Monitor** - Medium, Medium Impact (jobs.json)
5. **Jarvis Command** - Medium, High Impact (AI integration)
6. **Calendar Control** - Hard, Medium Impact (OAuth setup)

---

## Constraints

- **Model:** Minimax เท่านั้น
- **Storage:** localStorage แทน Server
- **API:** ประหยัด ใช้เมื่อจำเป็น
