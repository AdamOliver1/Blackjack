# Blackjack Project — Logic Summary

## Project Overview

This is a **multiplayer Blackjack web application** built as a monorepo with two Git submodules:

- **BlackJackClient** — React 18 + TypeScript frontend (Vite, Zustand, SignalR)
- **BlackjackServer** — ASP.NET Core 6 (C#) backend with SignalR for real-time communication

---

## Game Flow

### 1. Lobby (BEFORE_GAME)

- The server starts with **3 pre-created game rooms** managed by a singleton `GamesManager` using a `ConcurrentDictionary` for thread safety.
- The **Home page** fetches available rooms via `GET /api/game/games` and checks capacity via `GET /api/game/available?gameId={id}`.
- Players pick a room and join via SignalR's `JoinGame(gameId, playerId)`. Each room supports **up to 3 players**, each assigned a GUID.

### 2. Dealing (PLAYING)

- Any player clicks **"Start Game"** → server calls `GameHandler.StartGame()`:
  - A fresh 52-card deck is created and shuffled (Fisher-Yates algorithm) via `DeckHandler`.
  - Each player and the dealer receive **2 cards** (`DrawHand()`).
  - Players are arranged in a **linked list** (`NextPlayer`) for turn order.
  - The dealer's second card is displayed **face-down** on the client.

### 3. Player Turns (PLAYING)

- The **current player** gets "Hit me!" and "Stand" buttons enabled; other players' buttons are disabled.
- **Hit**: `GameHandler.PlayTurn()` draws a card from the deck for the current player. All clients are notified via `NotifyCardDrawn`.
- **Stand**: Advances to the next player via `NextTurn()`, or triggers `FinishRound()` if it's the last player.
- **Auto-finish**: If a player's hand exceeds 21 (bust), the client auto-triggers the next turn or round finish.

### 4. Hand Value Calculation

Implemented in `PlayerHandler.getSumOfHandValue()`:

- Number cards (2–10) = face value
- Face cards (J, Q, K) = **10**
- Aces = **11** if the total stays ≤ 21, otherwise **1**
- `IsHandValid` returns `true` if sum ≤ 21

### 5. Round Resolution (AFTER_GAME)

`GameHandler.FinishRound()`:

1. **Dealer draws** until hand value ≥ 17 (`drawDealerCards()`).
2. **Comparison** per player (`calcWinningPlayersByDealerScore()`):
   - Player bust (>21) → **loss**
   - Dealer bust (>21) and player not bust → **win**
   - Player sum ≥ Dealer sum → **win**
   - Player sum < Dealer sum → **loss**
3. Returns `winnersId[]` to all clients. Winners are highlighted in the UI.

### 6. Reset

- **"Another Round!"** → clears cards, creates a new deck, resets game state.
- **"Exit Room"** → disconnects the player from the SignalR group.

---

## Architecture & Communication

| Layer | Technology | Role |
|-------|-----------|------|
| **REST API** | ASP.NET Controllers | Room listing & availability checks |
| **Real-time** | SignalR Hub (`/gamehub`) | All game actions (join, hit, stand, results) |
| **State (server)** | `GamesManager` singleton | Holds all active `GameHandler` instances |
| **State (client)** | Zustand stores | `RoomStore` (game state, persisted to sessionStorage) + `GamesStore` (lobby data) |
| **UI** | React components | `GameTable` → `PlayerSeat` → `Card`, plus `ButtonsPanel` for controls |

### SignalR Events

| Client → Server | Server → All Clients |
|----------------|---------------------|
| `JoinGame` | `NotifynewPlayer` |
| `ReciveStartGame` | `NotifyStartGame` |
| `ReciveCardDrawn` | `NotifyCardDrawn` |
| `ReciveNextTurn` | `NotifyNextTurn` |
| `ReciveFinishRound` | `NotifyFinishRound` |
| `RecivePlayerExitGame` | `NotifyPlayerExitGame` |
| `RecivePlayAnotherRound` | `NotifyNewRound` |

---

## Key Backend Files

| File | Purpose |
|------|---------|
| `Handlers/GameHandler.cs` | Core blackjack logic (deal, hit, stand, winner calculation) |
| `Handlers/PlayerHandler.cs` | Player entity with hand value calculation and linked-list turn order |
| `Handlers/DeckHandler.cs` | 52-card deck creation, Fisher-Yates shuffle, draw |
| `Handlers/CardHandler.cs` | Card value mapping (face cards → 10, ace logic) |
| `Handlers/GamesManager.cs` | Singleton managing all active game rooms |
| `Hubs/GameHub.cs` | SignalR hub routing client actions to game logic |
| `Controllers/GameController.cs` | REST endpoints for lobby |

## Key Frontend Files

| File | Purpose |
|------|---------|
| `pages/Home/Home.tsx` | Lobby — lists rooms, checks availability |
| `pages/GameRoom/GameRoom.tsx` | Main game page — joins room, manages game lifecycle |
| `components/GameTable/GameTable.tsx` | Renders table with 3 player seats + dealer |
| `components/PlayerSeat/PlayerSeat.tsx` | Renders a player's cards at a table position |
| `components/Card/Card.tsx` | Individual card with suit icon, face-down state, glow effect |
| `components/ButtonsPanel/ButtonsPanel.tsx` | Hit/Stand/Start/Exit buttons with state-aware enable/disable |
| `stores/RoomStore.ts` | Zustand store for game state (persisted to sessionStorage) |
| `Hubs/useGameHubConnection.ts` | SignalR hook — connects, sends actions, listens for server events |

---

## Summary

The application is a standard-rules multiplayer Blackjack game where up to 3 players share a table against a dealer. All game logic (deck, dealing, hand evaluation, winner determination) runs on the .NET server. The React client is a thin presentation layer that sends player actions and reacts to server broadcasts via SignalR. State is kept in sync on both sides — the server is the source of truth, and the client mirrors it through Zustand stores.
