# 🎯 Mission Control - Product Spec

## 📋 Features Overview

### Level 1: No API (LocalStorage)
| Feature | Status | Description |
|---------|--------|-------------|
| Task Manager | ✅ Ready | CRUD with localStorage |
| Notes/Memo | 🔲 Todo | Quick notes |
| Price Alerts | 🔲 Todo | Set alert targets |
| Bookmarks | 🔲 Todo | Quick links |

### Level 2: Public API
| Feature | Status | Description |
|---------|--------|-------------|
| Trading Prices | 🔲 Todo | FMP API |
| Weather | 🔲 Todo | wttr.in |
| News | 🔲 Todo | Brave Search |

### Level 3: Backend Call
| Feature | Status | Description |
|---------|--------|-------------|
| Jarvis Command | 🔲 Todo | Sessions API |
| System Status | 🔲 Todo | Gateway API |
| Cron Jobs | 🔲 Todo | jobs.json |
| Calendar | 🔲 Todo | Google Calendar |

### Level 4: Advanced
| Feature | Status | Description |
|---------|--------|-------------|
| Voice Command | 🔲 Todo | Web Speech API |
| Push Notifications | 🔲 Todo | Notification API |
| Offline Mode | 🔲 Todo | PWA + ServiceWorker |

---

## 🎯 Priority Order

### Phase 1: Quick Wins (30 min)
1. Task Manager (localStorage persistence)
2. Notes/Memo
3. Bookmarks/Links
4. System Status (Gateway API)

### Phase 2: Core Features (1 hr)
5. Trading Prices (FMP)
6. Price Alerts (localStorage)
7. Cron Jobs Monitor

### Phase 3: Integration (1 hr)
8. Jarvis Command (Sessions API)
9. Calendar Events
10. Weather

### Phase 4: Advanced (Optional)
11. Voice Command
12. Push Notifications
13. Offline Mode (PWA)

---

## 💻 Code Examples

### 1. Task Manager (localStorage)

```javascript
// Task Manager Module
const TaskManager = {
    STORAGE_KEY: 'mission_tasks',
    
    getAll() {
        const data = localStorage.getItem(this.STORAGE_KEY);
        if (!data) return this.getDefaults();
        return JSON.parse(data);
    },
    
    getDefaults() {
        return [
            { id: 1, title: 'ทำรายงาน Trading Weekly', priority: 'high', due: 'Today', done: false, createdAt: new Date().toISOString() },
            { id: 2, title: 'ตรวจสอบ Cron Jobs', priority: 'med', due: 'Tomorrow', done: false, createdAt: new Date().toISOString() },
            { id: 3, title: 'อัปเดต Knowledge App', priority: 'low', due: '7 มี.ค.', done: false, createdAt: new Date().toISOString() }
        ];
    },
    
    save(tasks) {
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(tasks));
    },
    
    add(title, priority = 'med') {
        const tasks = this.getAll();
        tasks.push({
            id: Date.now(),
            title,
            priority,
            due: this.getDefaultDue(),
            done: false,
            createdAt: new Date().toISOString()
        });
        this.save(tasks);
        return tasks;
    },
    
    toggle(id) {
        const tasks = this.getAll();
        const task = tasks.find(t => t.id === id);
        if (task) task.done = !task.done;
        this.save(tasks);
        return tasks;
    },
    
    delete(id) {
        const tasks = this.getAll().filter(t => t.id !== id);
        this.save(tasks);
        return tasks;
    },
    
    getDefaultDue() {
        const tomorrow = new Date();
        tomorrow.setDate(tomorrow.getDate() + 1);
        return tomorrow.toLocaleDateString('th-TH', { day: 'numeric', month: 'short' });
    }
};
```

### 2. Notes/Memo

```javascript
// Notes Module
const Notes = {
    STORAGE_KEY: 'mission_notes',
    
    getAll() {
        return JSON.parse(localStorage.getItem(this.STORAGE_KEY) || '[]');
    },
    
    save(notes) {
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(notes));
    },
    
    add(content) {
        const notes = this.getAll();
        notes.unshift({
            id: Date.now(),
            content,
            createdAt: new Date().toISOString()
        });
        this.save(notes);
        return notes;
    },
    
    delete(id) {
        const notes = this.getAll().filter(n => n.id !== id);
        this.save(notes);
        return notes;
    },
    
    render(containerId) {
        const notes = this.getAll();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        container.innerHTML = notes.map(n => `
            <div class="note-item" onclick="Notes.delete(${n.id})">
                <div class="note-content">${n.content}</div>
                <div class="note-date">${new Date(n.createdAt).toLocaleString('th-TH')}</div>
            </div>
        `).join('') || '<div class="empty">ยังไม่มีโน้ต</div>';
    }
};
```

### 3. Price Alerts (localStorage)

```javascript
// Price Alert Module
const PriceAlert = {
    STORAGE_KEY: 'mission_price_alerts',
    
    getAll() {
        return JSON.parse(localStorage.getItem(this.STORAGE_KEY) || '[]');
    },
    
    add(symbol, targetPrice, condition) {
        const alerts = this.getAll();
        alerts.push({
            id: Date.now(),
            symbol: symbol.toUpperCase(),
            targetPrice: parseFloat(targetPrice),
            condition, // 'above' or 'below'
            triggered: false,
            createdAt: new Date().toISOString()
        });
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(alerts));
        return alerts;
    },
    
    check(prices) {
        const alerts = this.getAll();
        const triggered = [];
        
        alerts.forEach(alert => {
            const currentPrice = prices[alert.symbol];
            if (!currentPrice) return;
            
            const isTriggered = alert.condition === 'above' 
                ? currentPrice > alert.targetPrice 
                : currentPrice < alert.targetPrice;
            
            if (isTriggered && !alert.triggered) {
                alert.triggered = true;
                triggered.push(alert);
            }
        });
        
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(alerts));
        return triggered;
    },
    
    delete(id) {
        const alerts = this.getAll().filter(a => a.id !== id);
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(alerts));
        return alerts;
    },
    
    render(containerId) {
        const alerts = this.getAll();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        container.innerHTML = alerts.map(a => `
            <div class="alert-item">
                <div class="alert-info">
                    <strong>${a.symbol}</strong> 
                    ${a.condition === 'above' ? '▲' : '▼'} 
                    $${a.targetPrice}
                </div>
                <div class="alert-status ${a.triggered ? 'triggered' : 'waiting'}">
                    ${a.triggered ? '🔔 Triggered' : '⏳ Waiting'}
                </div>
                <button onclick="PriceAlert.delete(${a.id})">×</button>
            </div>
        `).join('') || '<div class="empty">ไม่มี Alert</div>';
    }
};
```

### 4. Bookmarks/Links

```javascript
// Bookmarks Module
const Bookmarks = {
    STORAGE_KEY: 'mission_bookmarks',
    
    getDefaults() {
        return [
            { id: 1, name: 'Knowledge App', url: 'https://basssg.github.io/daily-knowledge/', icon: '📚' },
            { id: 2, name: 'Content Plan', url: 'https://docs.google.com/spreadsheets/d/199EGCix13XC1urDD9d8PSPgFSsMYNHQ5MNFbzwA6lDI', icon: '📋' },
            { id: 3, name: 'Trading Journal', url: 'https://example.com/journal', icon: '📈' }
        ];
    },
    
    getAll() {
        const data = localStorage.getItem(this.STORAGE_KEY);
        return data ? JSON.parse(data) : this.getDefaults();
    },
    
    add(name, url, icon = '🔗') {
        const bookmarks = this.getAll();
        bookmarks.push({ id: Date.now(), name, url, icon });
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(bookmarks));
        return bookmarks;
    },
    
    delete(id) {
        const bookmarks = this.getAll().filter(b => b.id !== id);
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(bookmarks));
        return bookmarks;
    },
    
    render(containerId) {
        const bookmarks = this.getAll();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        container.innerHTML = bookmarks.map(b => `
            <div class="bookmark-item" onclick="window.open('${b.url}', '_blank')">
                <span class="bookmark-icon">${b.icon}</span>
                <span class="bookmark-name">${b.name}</span>
            </div>
        `).join('');
    }
};
```

### 5. Trading Prices (FMP API)

```javascript
// Trading Module - FMP API
const Trading = {
    // ประหยัด API: ใช้ Cache 5 นาที
    CACHE_DURATION: 5 * 60 * 1000,
    CACHE_KEY: 'mission_price_cache',
    
    async fetchPrices() {
        const cached = this.getCache();
        if (cached) return cached;
        
        // FMP Demo API (ฟรี 5 requests/day)
        const symbols = ['XAUUSD', 'BTCUSD']; // ประหยัด: ไม่ต้องทุกตัว
        const results = {};
        
        try {
            // ใช้ Gateway proxy แทนเรียกตรง
            const response = await fetch('/api/fmp/quote', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ symbols })
            });
            
            const data = await response.json();
            data.forEach(item => {
                results[item.symbol] = {
                    price: item.price,
                    change: item.change,
                    changePercent: item.changesPercentage
                };
            });
            
            this.setCache(results);
            return results;
        } catch (e) {
            console.error('Price fetch error:', e);
            return this.getDemoPrices();
        }
    },
    
    getCache() {
        const cached = localStorage.getItem(this.CACHE_KEY);
        if (!cached) return null;
        
        const { data, timestamp } = JSON.parse(cached);
        if (Date.now() - timestamp > this.CACHE_DURATION) return null;
        
        return data;
    },
    
    setCache(data) {
        localStorage.setItem(this.CACHE_KEY, JSON.stringify({
            data,
            timestamp: Date.now()
        }));
    },
    
    getDemoPrices() {
        return {
            XAUUSD: { price: 5171.50, change: 23.50, changePercent: 0.46 },
            BTCUSD: { price: 84230, change: 1230, changePercent: 1.48 }
        };
    }
};
```

### 6. Weather (wttr.in)

```javascript
// Weather Module - wttr.in (Free, No API Key)
const Weather = {
    async fetch(city = 'Bangkok') {
        try {
            const response = await fetch(`https://wttr.in/${city}?format=j1`);
            const data = await response.json();
            
            const current = data.current_condition[0];
            return {
                temp: current.temp_C,
                condition: current.weatherDesc[0].value,
                humidity: current.humidity,
                wind: current.windspeedKmph,
                location: data.nearest_area[0].areaName[0].value
            };
        } catch (e) {
            return null;
        }
    },
    
    async render(containerId) {
        const weather = await this.fetch();
        const container = document.getElementById(containerId);
        if (!container || !weather) return;
        
        container.innerHTML = `
            <div class="weather-card">
                <div class="weather-temp">${weather.temp}°C</div>
                <div class="weather-condition">${weather.condition}</div>
                <div class="weather-meta">💧 ${weather.humidity}% | 💨 ${weather.wind} km/h</div>
            </div>
        `;
    }
};
```

### 7. News (Brave Search)

```javascript
// News Module - Brave Search API
const News = {
    async search(query, limit = 5) {
        try {
            const response = await fetch('/api/search/news', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ 
                    q: query,
                    count: limit
                })
            });
            
            const data = await response.json();
            return data.results || [];
        } catch (e) {
            return [];
        }
    },
    
    async render(containerId, query = 'cryptocurrency') {
        const news = await this.search(query);
        const container = document.getElementById(containerId);
        if (!container) return;
        
        container.innerHTML = news.map(n => `
            <div class="news-item" onclick="window.open('${n.url}', '_blank')">
                <div class="news-title">${n.title}</div>
                <div class="news-meta">${n.age || ''}</div>
            </div>
        `).join('') || '<div class="empty">ไม่มีข่าว</div>';
    }
};
```

### 8. Jarvis Command (OpenClaw Sessions)

```javascript
// Jarvis Command Module
const Jarvis = {
    async sendCommand(message) {
        try {
            // เรียก Sessions API
            const response = await fetch('/api/sessions/agent:main:jarvis:direct/turn', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    message,
                    model: 'minimax-portal/MiniMax-M2.5'
                })
            });
            
            const data = await response.json();
            return data.reply || data.message || 'ไม่สามารถตอบกลับได้';
        } catch (e) {
            console.error('Jarvis error:', e);
            return '❌ เกิดข้อผิดพลาด';
        }
    },
    
    addMessage(role, content) {
        const chat = document.getElementById('chat-history');
        const icon = role === 'user' ? '👤' : '🤖';
        const name = role === 'user' ? 'คุณเบส' : 'Jarvis';
        
        chat.innerHTML += `
            <div class="chat-message ${role}">
                <div class="chat-header">${icon} ${name}</div>
                <div class="chat-content">${content}</div>
            </div>
        `;
        
        chat.scrollTop = chat.scrollHeight;
    },
    
    async process(input) {
        this.addMessage('user', input);
        
        // Loading
        const loadingMsg = this.addMessage('jarvis', '💖 กำลังคิด...');
        
        const reply = await this.sendCommand(input);
        
        // Replace loading with actual reply
        this.addMessage('jarvis', reply);
    }
};
```

### 9. Cron Jobs (jobs.json)

```javascript
// Cron Monitor Module
const CronMonitor = {
    async load() {
        try {
            // อ่านจาก jobs.json
            const response = await fetch('/api/files/memory/jobs_config_latest.json');
            const config = await response.json();
            
            return config.jobs.map(job => ({
                name: job.name,
                schedule: job.schedule?.expr || 'Unknown',
                enabled: job.enabled !== false,
                lastRun: job.state?.lastRunAtMs 
                    ? new Date(job.state.lastRunAtMs).toLocaleString('th-TH')
                    : 'Never',
                nextRun: job.state?.nextRunAtMs
                    ? new Date(job.state.nextRunAtMs).toLocaleString('th-TH')
                    : '-'
            }));
        } catch (e) {
            // Fallback: อ่านจาก Gateway
            return this.getFromGateway();
        }
    },
    
    async getFromGateway() {
        try {
            const response = await fetch('/api/cron/jobs');
            const data = await response.json();
            return data.jobs || [];
        } catch (e) {
            return [];
        }
    },
    
    async render(containerId) {
        const jobs = await this.load();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        container.innerHTML = jobs.map(j => `
            <div class="cron-item">
                <div class="cron-header">
                    <strong>${j.name}</strong>
                    <span class="status ${j.enabled ? 'active' : 'inactive'}">
                        ${j.enabled ? '✅ Active' : '⛔ Off'}
                    </span>
                </div>
                <div class="cron-meta">
                    ⏰ ${j.schedule} | Next: ${j.nextRun}
                </div>
            </div>
        `).join('') || '<div class="empty">ไม่มี Cron Jobs</div>';
    }
};
```

### 10. System Status (Gateway API)

```javascript
// System Monitor Module
const SystemMonitor = {
    async checkAll() {
        const checks = {
            gateway: this.checkGateway(),
            openclaw: this.checkOpenClaw(),
            minimax: this.checkMinimax(),
            brave: this.checkBrave(),
            fmp: this.checkFMP(),
            calendar: this.checkCalendar()
        };
        
        const results = await Promise.allSettled(Object.values(checks));
        
        return {
            gateway: results[0].value || 'Error',
            openclaw: results[1].value || 'Error',
            minimax: results[2].value || 'Error',
            brave: results[3].value || 'Error',
            fmp: results[4].value || 'Error',
            calendar: results[5].value || 'Error'
        };
    },
    
    async checkGateway() {
        try {
            const res = await fetch('/api/gateway/status');
            return res.ok ? 'OK' : 'Error';
        } catch { return 'Offline'; }
    },
    
    async checkOpenClaw() {
        try {
            const res = await fetch('/api/openclaw/version');
            const data = await res.json();
            return `v${data.version}`;
        } catch { return 'Offline'; }
    },
    
    async checkMinimax() {
        try {
            const res = await fetch('/api/llm/models');
            return res.ok ? 'Ready' : 'Error';
        } catch { return 'Offline'; }
    },
    
    async checkBrave() {
        try {
            const res = await fetch('/api/search/health');
            return res.ok ? 'OK' : 'Error';
        } catch { return 'Offline'; }
    },
    
    async checkFMP() {
        // ทดสอบด้วยการดึงราคา 1 ครั้ง
        try {
            const res = await fetch('/api/fmp/quote/XAUUSD');
            return res.ok ? 'OK' : 'Error';
        } catch { return 'Offline'; }
    },
    
    async checkCalendar() {
        try {
            const res = await fetch('/api/gog/calendar/status');
            return res.ok ? 'OK' : 'Error';
        } catch { return 'Offline'; }
    },
    
    async render(containerId) {
        const status = await this.checkAll();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        // Update individual elements
        Object.keys(status).forEach(key => {
            const el = document.getElementById(`sys-${key}`);
            if (el) {
                el.textContent = status[key];
                el.className = `system-status ${status[key] === 'OK' || status[key] === 'Ready' ? 'ok' : 'warn'}`;
            }
        });
    }
};
```

### 11. Calendar (Google Calendar API)

```javascript
// Calendar Module
const Calendar = {
    async fetchEvents(days = 7) {
        try {
            const response = await fetch('/api/gog/calendar/events', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    timeMin: new Date().toISOString(),
                    timeMax: new Date(Date.now() + days * 24 * 60 * 60 * 1000).toISOString(),
                    maxResults: 10
                })
            });
            
            const data = await response.json();
            return data.items || [];
        } catch (e) {
            return [];
        }
    },
    
    formatEventTime(start) {
        if (!start) return '';
        const date = new Date(start.dateTime || start.date);
        return date.toLocaleString('th-TH', { 
            hour: '2-digit', 
            minute: '2-digit',
            day: 'numeric',
            month: 'short'
        });
    },
    
    isToday(dateStr) {
        const date = new Date(dateStr);
        const today = new Date();
        return date.toDateString() === today.toDateString();
    },
    
    async render(containerId) {
        const events = await this.fetchEvents();
        const container = document.getElementById(containerId);
        if (!container) return;
        
        if (events.length === 0) {
            container.innerHTML = '<div class="empty">ไม่มีนัดหมาย</div>';
            return;
        }
        
        container.innerHTML = events.map(e => `
            <div class="task-item">
                <div class="task-content">
                    <div class="task-title">${e.summary}</div>
                    <div class="task-meta">${this.formatEventTime(e.start)}</div>
                </div>
            </div>
        `).join('');
    }
};
```

### 12. Voice Command (Web Speech API)

```javascript
// Voice Command Module
const VoiceCommand = {
    recognition: null,
    
    init() {
        if (!('webkitSpeechRecognition' in window) && !('SpeechRecognition' in window)) {
            console.warn('Speech recognition not supported');
            return false;
        }
        
        const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
        this.recognition = new SpeechRecognition();
        
        this.recognition.continuous = false;
        this.recognition.interimResults = false;
        this.recognition.lang = 'th-TH';
        
        this.recognition.onresult = (event) => {
            const transcript = event.results[0][0].transcript;
            console.log('Voice input:', transcript);
            
            // ส่งไป Jarvis
            Jarvis.process(transcript);
        };
        
        this.recognition.onerror = (event) => {
            console.error('Speech error:', event.error);
        };
        
        return true;
    },
    
    start() {
        if (!this.recognition) {
            if (!this.init()) {
                alert('Browser ไม่รองรับ Voice Command');
                return;
            }
        }
        
        this.recognition.start();
    },
    
    toggle() {
        if (this.recognition && this.recognition.state === 'listening') {
            this.recognition.stop();
        } else {
            this.start();
        }
    }
};
```

### 13. Push Notifications

```javascript
// Push Notification Module
const PushNotification = {
    async requestPermission() {
        if (!('Notification' in window)) {
            return false;
        }
        
        const permission = await Notification.requestPermission();
        return permission === 'granted';
    },
    
    send(title, options = {}) {
        if (Notification.permission === 'granted') {
            new Notification(title, {
                icon: '🎯',
                badge: '🎯',
                ...options
            });
        }
    },
    
    // Price Alert Notification
    priceAlert(symbol, price, target, condition) {
        this.send(`🔔 ${symbol} Alert!`, {
            body: `${symbol} is now $${price} (${condition} $${target})`,
            tag: `price-${symbol}`
        });
    },
    
    // Task Reminder
    taskReminder(task) {
        this.send(`📋 Task Reminder`, {
            body: task.title,
            tag: 'task-reminder'
        });
    }
};
```

### 14. Offline Mode (PWA)

```javascript
// Service Worker Registration
if ('serviceWorker' in navigator) {
    navigator.serviceWorker.register('/sw.js').then(reg => {
        console.log('Service Worker registered');
    });
}

// sw.js (Service Worker)
const CACHE_NAME = 'mission-control-v1';
const ASSETS = [
    '/mission-control/index.html',
    '/mission-control/styles.css',
    '/mission-control/app.js'
];

self.addEventListener('install', event => {
    event.waitUntil(
        caches.open(CACHE_NAME).then(cache => {
            return cache.addAll(ASSETS);
        })
    );
});

self.addEventListener('fetch', event => {
    event.respondWith(
        caches.match(event.request).then(response => {
            return response || fetch(event.request);
        })
    );
});
```

---

## 📁 File Structure

```
mission-control/
├── index.html          # Main app
├── SPEC.md             # This file
├── api/
│   └── proxy.js        # Backend API proxies
└── sw.js              # Service Worker (PWA)
```

---

## ✅ Implementation Checklist

- [x] Task Manager (localStorage)
- [ ] Notes/Memo
- [ ] Price Alerts
- [ ] Bookmarks
- [ ] Trading Prices (FMP)
- [ ] Weather (wttr.in)
- [ ] News (Brave)
- [ ] Jarvis Command
- [ ] System Status
- [ ] Cron Jobs
- [ ] Calendar
- [ ] Voice Command
- [ ] Push Notifications
- [ ] Offline Mode (PWA)
