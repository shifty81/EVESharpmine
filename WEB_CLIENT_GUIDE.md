# Building a Web-Based Client for EVESharpmine

## Overview

This guide provides practical steps to create a standalone web-based client that integrates with the existing EVESharp server infrastructure.

## Existing Foundation

EVESharpmine already includes:
- ✅ **REST API** (`Server/EVESharp.API/`) with XML endpoints
- ✅ **Account management** (character listing)
- ✅ **Server status** endpoint
- ✅ **Character management** capabilities
- ✅ **Image serving** (character portraits)

**Current API Endpoints:**
- `/account/Characters.xml.aspx` - List characters
- `/server/ServerStatus.xml.aspx` - Server status
- `/character/*` - Character operations
- `/images/*` - Image assets

## Recommended Approach: Progressive Web App

### Technology Stack

**Backend:**
- Keep existing C# server and API
- Add SignalR for real-time updates
- Extend REST API with JSON endpoints

**Frontend:**
- HTML5 + CSS3 + JavaScript
- Optional: React, Vue, or vanilla JS
- WebSocket/SignalR for live data
- Canvas/WebGL for simple space visualization

### Architecture

```
┌────────────────────────────────────────────┐
│  Web Browser (Client)                      │
│  ┌──────────────────────────────────────┐ │
│  │  Single Page Application (SPA)       │ │
│  │  - Login screen                      │ │
│  │  - Character selection               │ │
│  │  - Station UI (inventory, market)    │ │
│  │  - Space view (2D/simple 3D)         │ │
│  │  - Chat windows                      │ │
│  └──────────────────────────────────────┘ │
└────────────────────────────────────────────┘
           ▲              ▲
           │              │
    REST API (HTTP)   SignalR (WebSocket)
           │              │
           ▼              ▼
┌────────────────────────────────────────────┐
│  EVESharp Server                           │
│  ┌────────────┐  ┌───────────────────┐   │
│  │  Web API   │  │  Game Services    │   │
│  │  (existing)│  │  (existing)       │   │
│  └────────────┘  └───────────────────┘   │
│         ▲              ▲                   │
│         └──────┬───────┘                   │
│            Database                        │
└────────────────────────────────────────────┘
```

## Implementation Phases

### Phase 1: Basic Web UI (4-6 weeks)

**Goal:** Login and character selection

**Tasks:**
1. Create new project: `Server/EVESharp.WebClient/`
2. Setup ASP.NET Core with static file serving
3. Build login page (HTML/CSS/JS)
4. Implement authentication via API
5. Create character selection screen
6. Display character list from `/account/Characters.xml.aspx`

**File Structure:**
```
Server/EVESharp.WebClient/
├── wwwroot/
│   ├── index.html
│   ├── css/
│   │   └── styles.css
│   ├── js/
│   │   ├── app.js
│   │   ├── auth.js
│   │   └── characters.js
│   └── assets/
│       └── images/
├── Program.cs
└── EVESharp.WebClient.csproj
```

**Example Login Page (wwwroot/index.html):**
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>EVESharp Web Client</title>
    <link rel="stylesheet" href="css/styles.css">
</head>
<body>
    <div id="app">
        <div id="login-screen" class="screen active">
            <h1>EVESharp Login</h1>
            <form id="login-form">
                <input type="text" id="username" placeholder="Username" required>
                <input type="password" id="password" placeholder="Password" required>
                <button type="submit">Login</button>
            </form>
            <div id="error-message"></div>
        </div>
        
        <div id="character-select" class="screen">
            <h2>Select Character</h2>
            <div id="character-list"></div>
        </div>
        
        <div id="game-ui" class="screen">
            <!-- Game interface goes here -->
        </div>
    </div>
    
    <script src="js/app.js"></script>
</body>
</html>
```

**Example JavaScript (wwwroot/js/auth.js):**
```javascript
class AuthService {
    constructor(apiUrl) {
        this.apiUrl = apiUrl;
        this.token = null;
    }
    
    async login(username, password) {
        try {
            const response = await fetch(`${this.apiUrl}/auth/login`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ username, password })
            });
            
            if (!response.ok) throw new Error('Login failed');
            
            const data = await response.json();
            this.token = data.token;
            localStorage.setItem('auth_token', this.token);
            return true;
        } catch (error) {
            console.error('Login error:', error);
            return false;
        }
    }
    
    async getCharacters() {
        const response = await fetch(`${this.apiUrl}/account/Characters.xml.aspx`, {
            method: 'POST',
            headers: {
                'Authorization': `Bearer ${this.token}`
            }
        });
        
        // Parse XML response
        const xmlText = await response.text();
        const parser = new DOMParser();
        const xmlDoc = parser.parseFromString(xmlText, 'text/xml');
        
        return this.parseCharacterXML(xmlDoc);
    }
    
    parseCharacterXML(xmlDoc) {
        const rows = xmlDoc.querySelectorAll('row');
        return Array.from(rows).map(row => ({
            characterID: row.getAttribute('characterID'),
            name: row.getAttribute('name'),
            corporationID: row.getAttribute('corporationID'),
            corporationName: row.getAttribute('corporationName')
        }));
    }
}
```

### Phase 2: Station UI (8-12 weeks)

**Goal:** Implement docked gameplay

**Features:**
- Inventory management (drag-and-drop)
- Market browser (buy/sell items)
- Fitting window (ship modules)
- Character sheet (skills, attributes)
- Station services (repair, insurance)
- Chat interface

**New API Endpoints Needed:**
```
GET  /inventory/{characterID}
POST /inventory/move
GET  /market/orders
POST /market/buy
POST /market/sell
GET  /character/{characterID}/skills
POST /fitting/save
GET  /fitting/load
```

**UI Components:**
```javascript
class InventoryManager {
    constructor(apiService) {
        this.api = apiService;
        this.items = [];
    }
    
    async loadInventory(characterID) {
        this.items = await this.api.get(`/inventory/${characterID}`);
        this.render();
    }
    
    render() {
        const container = document.getElementById('inventory');
        container.innerHTML = '';
        
        this.items.forEach(item => {
            const itemElement = this.createItemElement(item);
            container.appendChild(itemElement);
        });
    }
    
    createItemElement(item) {
        const div = document.createElement('div');
        div.className = 'inventory-item';
        div.draggable = true;
        div.innerHTML = `
            <img src="/images/type/${item.typeID}_64.png" alt="${item.typeName}">
            <span>${item.typeName}</span>
            <span class="quantity">x${item.quantity}</span>
        `;
        div.addEventListener('dragstart', (e) => this.onDragStart(e, item));
        return div;
    }
    
    onDragStart(event, item) {
        event.dataTransfer.setData('item', JSON.stringify(item));
    }
}
```

### Phase 3: Space View (12-16 weeks)

**Goal:** Undocking and basic space interaction

**Features:**
- 2D top-down space view
- Ship movement visualization
- Object selection (ships, stations, asteroids)
- Navigation controls
- Overview panel
- Basic combat visualization

**Technology Options:**

**Option A: Canvas 2D (Simpler)**
```javascript
class SpaceView {
    constructor(canvasId) {
        this.canvas = document.getElementById(canvasId);
        this.ctx = this.canvas.getContext('2d');
        this.objects = [];
        this.camera = { x: 0, y: 0, zoom: 1.0 };
    }
    
    addObject(obj) {
        this.objects.push(obj);
    }
    
    render() {
        this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
        
        // Draw space background
        this.ctx.fillStyle = '#000011';
        this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
        
        // Draw stars
        this.drawStars();
        
        // Draw game objects
        this.objects.forEach(obj => this.drawObject(obj));
        
        // Request next frame
        requestAnimationFrame(() => this.render());
    }
    
    drawObject(obj) {
        const screenX = (obj.x - this.camera.x) * this.camera.zoom;
        const screenY = (obj.y - this.camera.y) * this.camera.zoom;
        
        // Draw ship/station icon
        this.ctx.fillStyle = obj.color || '#ffffff';
        this.ctx.beginPath();
        this.ctx.arc(screenX, screenY, obj.radius * this.camera.zoom, 0, Math.PI * 2);
        this.ctx.fill();
        
        // Draw label
        this.ctx.fillStyle = '#ffffff';
        this.ctx.font = '12px Arial';
        this.ctx.fillText(obj.name, screenX + 10, screenY - 10);
    }
    
    drawStars() {
        // Simple starfield effect
        for (let i = 0; i < 100; i++) {
            this.ctx.fillStyle = `rgba(255, 255, 255, ${Math.random()})`;
            this.ctx.fillRect(
                Math.random() * this.canvas.width,
                Math.random() * this.canvas.height,
                2, 2
            );
        }
    }
}
```

**Option B: Three.js (More Advanced)**
```javascript
class SpaceView3D {
    constructor(containerId) {
        this.container = document.getElementById(containerId);
        this.scene = new THREE.Scene();
        this.camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
        this.renderer = new THREE.WebGLRenderer({ antialias: true });
        
        this.renderer.setSize(this.container.clientWidth, this.container.clientHeight);
        this.container.appendChild(this.renderer.domElement);
        
        this.setupLights();
        this.setupStarfield();
        this.animate();
    }
    
    setupStarfield() {
        const geometry = new THREE.BufferGeometry();
        const vertices = [];
        
        for (let i = 0; i < 10000; i++) {
            vertices.push(
                Math.random() * 2000 - 1000,
                Math.random() * 2000 - 1000,
                Math.random() * 2000 - 1000
            );
        }
        
        geometry.setAttribute('position', new THREE.Float32BufferAttribute(vertices, 3));
        const material = new THREE.PointsMaterial({ color: 0xffffff, size: 2 });
        const stars = new THREE.Points(geometry, material);
        this.scene.add(stars);
    }
    
    addShip(position, type) {
        // Simple ship representation (cube for now)
        const geometry = new THREE.BoxGeometry(5, 2, 10);
        const material = new THREE.MeshPhongMaterial({ color: 0x00ff00 });
        const ship = new THREE.Mesh(geometry, material);
        
        ship.position.set(position.x, position.y, position.z);
        this.scene.add(ship);
        
        return ship;
    }
    
    animate() {
        requestAnimationFrame(() => this.animate());
        this.renderer.render(this.scene, this.camera);
    }
}
```

### Phase 4: Real-Time Features (16-20 weeks)

**Goal:** Live server communication

**Implementation:**
1. Add SignalR to server
2. Implement real-time notifications
3. Chat system (local, corp, alliance)
4. Fleet system
5. Position updates
6. Market updates

**SignalR Hub (Server-side):**
```csharp
using Microsoft.AspNetCore.SignalR;

namespace EVESharp.WebClient.Hubs
{
    public class GameHub : Hub
    {
        public async Task JoinCharacter(int characterID)
        {
            await Groups.AddToGroupAsync(Context.ConnectionId, $"character_{characterID}");
            await Clients.Caller.SendAsync("Connected", characterID);
        }
        
        public async Task SendChatMessage(string channel, string message)
        {
            await Clients.Group($"chat_{channel}").SendAsync("ReceiveChatMessage", 
                Context.User.Identity.Name, message);
        }
        
        public async Task UpdatePosition(double x, double y, double z)
        {
            // Broadcast position to nearby players
            await Clients.Others.SendAsync("PlayerMoved", Context.ConnectionId, x, y, z);
        }
    }
}
```

**Client-side SignalR:**
```javascript
class GameConnection {
    constructor(hubUrl) {
        this.connection = new signalR.HubConnectionBuilder()
            .withUrl(hubUrl)
            .withAutomaticReconnect()
            .build();
        
        this.setupHandlers();
    }
    
    setupHandlers() {
        this.connection.on('ReceiveChatMessage', (sender, message) => {
            this.displayChatMessage(sender, message);
        });
        
        this.connection.on('PlayerMoved', (playerId, x, y, z) => {
            this.updatePlayerPosition(playerId, x, y, z);
        });
        
        this.connection.on('MarketOrderUpdated', (order) => {
            this.updateMarketOrder(order);
        });
    }
    
    async start() {
        await this.connection.start();
        console.log('Connected to game server');
    }
    
    async sendChat(channel, message) {
        await this.connection.invoke('SendChatMessage', channel, message);
    }
}
```

## Project Setup Instructions

### 1. Create Web Client Project

```bash
cd Server
dotnet new web -n EVESharp.WebClient
cd EVESharp.WebClient
dotnet add package Microsoft.AspNetCore.SignalR
```

### 2. Update Program.cs

```csharp
using EVESharp.WebClient.Hubs;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddSignalR();
builder.Services.AddCors(options =>
{
    options.AddDefaultPolicy(policy =>
    {
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

var app = builder.Build();

// Serve static files
app.UseDefaultFiles();
app.UseStaticFiles();

// Enable CORS
app.UseCors();

// Map SignalR hub
app.MapHub<GameHub>("/gamehub");

app.Run();
```

### 3. Create wwwroot Directory Structure

```bash
mkdir -p wwwroot/{css,js,assets/images}
```

### 4. Add Dependencies (wwwroot/index.html)

```html
<!-- SignalR Client -->
<script src="https://cdn.jsdelivr.net/npm/@microsoft/signalr@latest/dist/browser/signalr.min.js"></script>

<!-- Optional: Three.js for 3D -->
<script src="https://cdn.jsdelivr.net/npm/three@latest/build/three.min.js"></script>

<!-- Your app -->
<script src="js/app.js"></script>
```

### 5. Run Combined Server

```bash
# Terminal 1: Run main game server
cd Server/EVESharp.Node
dotnet run -- --mode single

# Terminal 2: Run web client server
cd Server/EVESharp.WebClient
dotnet run

# Open browser: http://localhost:5000
```

## Integration with Existing Server

### Option A: Separate Processes (Recommended for Development)

- EVESharp.Node runs on port 26000 (game server)
- EVESharp.WebClient runs on port 5000 (web server)
- Web client makes API calls to Node server
- Deploy both together

### Option B: Integrated Process (Production)

Merge web client into EVESharp.Node:

```csharp
// In EVESharp.Node/Program.cs
var webAppBuilder = WebApplication.CreateBuilder();
webAppBuilder.Services.AddSignalR();
// ... configure web services

var webApp = webAppBuilder.Build();
webApp.UseStaticFiles();
webApp.MapHub<GameHub>("/gamehub");

// Run web server in background
Task.Run(() => webApp.RunAsync("http://localhost:5000"));

// Continue with existing game server startup
// ...
```

## Styling Guidelines

**Design Principles:**
- Dark theme (space aesthetic)
- Minimal, functional UI
- Clear information hierarchy
- Responsive layout

**Example CSS (wwwroot/css/styles.css):**
```css
:root {
    --bg-dark: #0a0e1a;
    --bg-panel: #141824;
    --accent: #00d4ff;
    --text: #e0e0e0;
    --text-dim: #8a8a8a;
}

body {
    margin: 0;
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: var(--bg-dark);
    color: var(--text);
}

#app {
    width: 100vw;
    height: 100vh;
    display: flex;
    flex-direction: column;
}

.screen {
    display: none;
}

.screen.active {
    display: flex;
}

#login-screen {
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.panel {
    background: var(--bg-panel);
    border: 1px solid var(--accent);
    border-radius: 4px;
    padding: 20px;
}

button {
    background: var(--accent);
    color: var(--bg-dark);
    border: none;
    padding: 10px 20px;
    cursor: pointer;
    font-weight: bold;
}

button:hover {
    opacity: 0.8;
}

.inventory-item {
    background: var(--bg-panel);
    border: 1px solid var(--text-dim);
    padding: 10px;
    margin: 5px;
    display: inline-block;
    cursor: grab;
}

.inventory-item:active {
    cursor: grabbing;
}
```

## Testing Strategy

1. **Unit Tests**: Test JavaScript modules independently
2. **Integration Tests**: Test API communication
3. **E2E Tests**: Use Playwright or Selenium
4. **Performance Tests**: Monitor WebSocket connections

## Deployment

**Development:**
```bash
dotnet run
```

**Production:**
```bash
dotnet publish -c Release
```

Serve from: `/bin/Release/net5.0/publish/`

## Next Steps

1. ✅ Start with Phase 1 (login/character select)
2. ✅ Test with existing API endpoints
3. ✅ Iterate on UI/UX
4. ✅ Expand to Phase 2 (station UI)
5. ✅ Add real-time features (Phase 4)
6. ⚠️ Consider Phase 3 (space view) last

## Resources

- **SignalR Docs**: https://docs.microsoft.com/aspnet/core/signalr/
- **Three.js**: https://threejs.org/docs/
- **Canvas Tutorial**: https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial

---

*Document Version: 1.0*  
*Last Updated: 2026-02-06*
