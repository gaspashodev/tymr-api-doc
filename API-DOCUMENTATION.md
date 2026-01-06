# 📡 Timer Multi-Joueurs - Documentation API

Cette documentation explique comment récupérer les données d'une partie en temps réel pour les afficher sur un site web, un overlay de streaming, ou toute autre application.

---

## 🌐 URL de base

```
https://tymr-server.up.railway.app
```

---

## 📋 Endpoints disponibles

| Endpoint | Description |
|----------|-------------|
| `GET /api/sessions/join/{CODE}` | Trouver une session par son code |
| `GET /api/stream/{SESSION_ID}` | Données optimisées pour affichage |
| `GET /api/party/{SESSION_ID}/stats` | Statistiques complètes |
| `GET /api/party/{SESSION_ID}/player/{PLAYER_ID}` | Données d'un joueur spécifique |
| `GET /health` | Vérifier que le serveur est en ligne |

---

## 🔍 Détails des endpoints

### 1. Trouver une session par code

```
GET /api/sessions/join/{CODE}
```

**Paramètres :**
- `CODE` : Code à 6 caractères de la partie (ex: `A1B2C3`)

**Exemple :**
```bash
curl https://tymr-server.up.railway.app/api/sessions/join/A1B2C3
```

**Réponse :**
```json
{
  "sessionId": "550e8400-e29b-41d4-a716-446655440000",
  "session": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "mode": "sequential",
    "displayMode": "distributed",
    "players": [...],
    "currentPlayerIndex": 1,
    "status": "started"
  }
}
```

---

### 2. Données pour affichage (recommandé)

```
GET /api/stream/{SESSION_ID}
```

**Idéal pour :** Overlays, widgets, affichages temps réel

**Exemple :**
```bash
curl https://tymr-server.up.railway.app/api/stream/550e8400-e29b-41d4-a716-446655440000
```

**Réponse :**
```json
{
  "mode": "sequential",
  "globalTime": 245,
  "globalTimeFormatted": "4:05",
  "totalPlayers": 4,
  "players": [
    {
      "id": 0,
      "playerNumber": 1,
      "name": "Alice",
      "time": 120,
      "timeFormatted": "2:00",
      "isActive": false,
      "isEliminated": false,
      "isConnected": true,
      "percentageRemaining": 67
    },
    {
      "id": 1,
      "playerNumber": 2,
      "name": "Bob",
      "time": 95,
      "timeFormatted": "1:35",
      "isActive": true,
      "isEliminated": false,
      "isConnected": true,
      "percentageRemaining": 53
    }
  ],
  "currentPlayer": "Bob",
  "currentPlayerId": 1
}
```

**Champs importants :**

| Champ | Description |
|-------|-------------|
| `id` | Index du joueur (0, 1, 2...) |
| `playerNumber` | Numéro du joueur (1, 2, 3...) - pour affichage |
| `time` | Temps en secondes |
| `timeFormatted` | Temps formaté (ex: "2:30") |
| `isActive` | Le timer de ce joueur tourne-t-il ? |
| `isEliminated` | Le joueur est-il éliminé ? |
| `isConnected` | Le joueur est-il connecté ? |
| `percentageRemaining` | % du temps restant (mode compte à rebours) |

---

### 3. Statistiques complètes

```
GET /api/party/{SESSION_ID}/stats
```

**Idéal pour :** Pages de résultats, tableaux de bord

**Réponse :**
```json
{
  "sessionId": "550e8400-e29b-41d4-a716-446655440000",
  "joinCode": "550E84",
  "mode": "sequential",
  "displayMode": "distributed",
  "status": "started",
  "timeLimit": 180,
  "timeLimitFormatted": "3:00",
  
  "globalTime": 245,
  "globalTimeFormatted": "4:05",
  "averageTime": 61,
  "averageTimeFormatted": "1:01",
  
  "totalPlayers": 4,
  "connectedPlayers": 3,
  "activePlayers": 1,
  "eliminatedPlayers": 0,
  
  "players": [
    {
      "id": 0,
      "playerNumber": 1,
      "name": "Alice",
      "time": 120,
      "timeFormatted": "2:00",
      "isRunning": false,
      "isEliminated": false,
      "isConnected": true,
      "percentageRemaining": 67,
      "rank": 2
    }
  ],
  
  "ranking": [
    {
      "rank": 1,
      "id": 1,
      "playerNumber": 2,
      "name": "Bob",
      "time": 125,
      "timeFormatted": "2:05",
      "isEliminated": false,
      "percentageRemaining": 69
    }
  ],
  
  "currentPlayerIndex": 1,
  "currentPlayerName": "Bob"
}
```

---

### 4. Données d'un joueur spécifique

```
GET /api/party/{SESSION_ID}/player/{PLAYER_ID}
```

**Paramètres :**
- `SESSION_ID` : ID de la session
- `PLAYER_ID` : ID du joueur (0, 1, 2...)

**Exemple :**
```bash
curl https://tymr-server.up.railway.app/api/party/550e8400.../player/0
```

**Réponse :**
```json
{
  "playerId": 0,
  "playerNumber": 1,
  "name": "Alice",
  "time": 120,
  "timeFormatted": "2:00",
  "isRunning": false,
  "isEliminated": false,
  "isConnected": true,
  "percentageRemaining": 67,
  "rank": 2,
  "totalPlayers": 4,
  "isCurrent": false
}
```

---

## ⚡ Temps réel avec WebSocket

Pour des mises à jour instantanées (sans polling), utilisez WebSocket avec Socket.IO :

### Installation

```html
<script src="https://cdn.socket.io/4.6.0/socket.io.min.js"></script>
```

### Connexion

```javascript
const SERVER_URL = 'https://tymr-server.up.railway.app';

// 1. Récupérer l'ID de session via l'API REST
const response = await fetch(`${SERVER_URL}/api/sessions/join/${CODE}`);
const { sessionId } = await response.json();

// 2. Se connecter en WebSocket
const socket = io(SERVER_URL, {
  transports: ['websocket']
});

// 3. Rejoindre la session
socket.on('connect', () => {
  socket.emit('join-session', sessionId);
});

// 4. Recevoir les mises à jour en temps réel
socket.on('session-state', (session) => {
  console.log('Mise à jour reçue:', session);
  
  // Accéder aux joueurs
  session.players.forEach(player => {
    console.log(`${player.name}: ${player.time}s`);
  });
});
```

---

## 💡 Exemples d'utilisation

### Afficher un joueur spécifique par son numéro

```javascript
// Toujours afficher le joueur 1 au même endroit
const player1 = session.players.find(p => p.playerNumber === 1);
document.getElementById('player1-time').textContent = player1.timeFormatted;
```

### Afficher le joueur actif

```javascript
const activePlayer = session.players.find(p => p.isActive);
if (activePlayer) {
  document.getElementById('current').textContent = 
    `${activePlayer.name}: ${activePlayer.timeFormatted}`;
}
```

### Grille de tous les joueurs

```javascript
session.players.forEach(player => {
  const html = `
    <div class="player ${player.isActive ? 'active' : ''}">
      <span class="number">#${player.playerNumber}</span>
      <span class="name">${player.name}</span>
      <span class="time">${player.timeFormatted}</span>
    </div>
  `;
  container.innerHTML += html;
});
```

### Polling simple (sans WebSocket)

```javascript
async function updateDisplay() {
  const response = await fetch(`${SERVER_URL}/api/stream/${sessionId}`);
  const data = await response.json();
  
  // Mettre à jour l'affichage
  document.getElementById('total').textContent = data.globalTimeFormatted;
  // ...
}

// Rafraîchir toutes les secondes
setInterval(updateDisplay, 1000);
```

---

## 🎨 Exemple HTML complet

Un fichier d'exemple `timer-overlay.html` est fourni avec le projet. Il montre comment :
- Se connecter à une partie par code
- Afficher les temps en temps réel via WebSocket
- Styliser l'affichage avec du CSS moderne

---

## ❓ FAQ

### Comment trouver le SESSION_ID ?

1. Utilisez d'abord `/api/sessions/join/{CODE}` avec le code à 6 caractères
2. La réponse contient le `sessionId` complet

### Quelle est la différence entre `id` et `playerNumber` ?

- `id` : Index technique (0, 1, 2...) - utilisé pour les appels API
- `playerNumber` : Numéro affiché (1, 2, 3...) - pour l'interface utilisateur

### Comment savoir qui joue actuellement ?

- En mode séquentiel : `currentPlayer` (nom) et `currentPlayerId` (id)
- Ou cherchez le joueur avec `isActive: true`

### WebSocket ou REST ?

| Méthode | Avantage | Inconvénient |
|---------|----------|--------------|
| WebSocket | Instantané | Plus complexe |
| REST (polling) | Simple | Délai de 1+ seconde |

Pour un overlay de stream, **WebSocket est recommandé**.

---

## 📞 Support

Pour toute question ou problème, consultez le repository du projet ou ouvrez une issue.
