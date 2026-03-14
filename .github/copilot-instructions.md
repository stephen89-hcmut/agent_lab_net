---
description: Copilot instructions for the SocOps Bingo game—a Blazor WebAssembly app built for in-person social mixers.
---

# SocOps Bingo: Copilot Instructions

## Project Overview

**SocOps** is an interactive Bingo game designed for in-person events and mixers. Players find people matching descriptions (e.g., "bikes to work", "speaks >2 languages") and mark squares to win. Built with **Blazor WebAssembly 10.0** and deployed to GitHub Pages.

- **Tech Stack**: C# + Blazor WASM, .NET 10, localStorage persistence, custom CSS utilities
- **Key Goal**: Fast, mobile-friendly event ice-breaker with instant win detection

## Architecture & Data Flow

### Service Layer (`Services/`)

**BingoGameService** (scoped, registered in Program.cs)
- Owns game state: `CurrentGameState`, `Board`, `WinningLine`, `ShowBingoModal`
- **Key methods**:
  - `StartGame()` → generates board, resets state
  - `HandleSquareClick(squareId)` → toggles square, checks for bingo
  - `ResetGame()` → returns to start screen
  - `LoadGameStateAsync()` / `SaveGameStateAsync()` → localStorage sync (fire-and-forget pattern)
- Exposes `OnStateChanged` event for component subscriptions

**BingoLogicService** (static utilities)
- `GenerateBoard()` → shuffles 24 questions (Fisher-Yates), places FREE_SPACE at center (index 12)
- `ToggleSquare()` → immutable toggle (returns new list)
- `CheckBingo()` → detects winning lines (rows, cols, diagonals)
- `GetWinningSquareIds()` → returns HashSet<int> for highlighting

### Component Hierarchy

```
App.razor (router)
└── MainLayout.razor
    ├── NavMenu.razor (header)
    └── [Page] (Home.razor with state management)
        ├── StartScreen.razor (conditional display)
        └── GameScreen.razor (when playing)
            ├── BingoBoard.razor (5x5 grid container)
            │   └── BingoSquare.razor (individual cells, repeating)
            └── BingoModal.razor (win celebration)
```

**Data binding pattern**: Home.razor subscribes to `BingoGameService.OnStateChanged`, calls `NotifyComponentRefresh()` on cascade parameters.

### Models (`Models/`)

- `GameState` enum: `Start`, `Playing`, `Bingo`
- `BingoSquareData`: `Id`, `Text`, `IsMarked` (mutable only via ToggleSquare), `IsFreeSpace`
- `BingoLine`: `LineType` (row/col/diag), `StartIndex`, `Squares`

### Data (`Data/Questions.cs`)

Static list of 24 icebreaker prompts. Shuffled fresh for each game. Extend this list by adding strings to `QuestionsList`.

## Key Patterns

### 1. **Immutability in Game Logic**
Question board mutations happen through `BingoLogicService.ToggleSquare()`, not direct property assignment. Always return a new `List<BingoSquareData>` to ensure change detection.

**Pattern**:
```csharp
board.Select(square =>
    square.Id == targetId
        ? new BingoSquareData { ... updated values ... }
        : square
).ToList()
```

### 2. **Fire-and-Forget localStorage**
Async save/load via `_ = SaveGameStateAsync()` prevents blocking. JSON serialization with versioning (`STORAGE_VERSION`).

### 3. **Event-Driven State Updates**
Components don't poll; they subscribe to `OnStateChanged` and call `StateHasChanged()` or refresh via cascade parameters.

### 4. **CSS Utilities (Custom, No Tailwind)**
Defined in `wwwroot/css/app.css`. Use composable classes:
- Layout: `.flex`, `.grid`, `.grid-cols-5`, `.items-center`
- Spacing: `.p-3`, `.mb-2`, `.gap-2`
- Colors: `.bg-white`, `.bg-accent` (primary), `.text-gray-500`
- See `.github/instructions/css-utilities.instructions.md` for full reference

### 5. **Mobile-First Responsive Design**
No breakpoints defined; layouts should work on small screens by default. Flexbox/grid with appropriate gaps and padding.

## Developer Workflows

### Build
```bash
cd SocOps
dotnet build
```

### Run Dev Server
```bash
cd SocOps
dotnet run
# Launches https://localhost:7xxx with hot reload
```

### Deploy
Push to `main` branch → GitHub Actions auto-deploys to GitHub Pages (`gh-pages`).

### Add New Questions
Edit `SocOps/Data/Questions.cs` → add to `QuestionsList`. Only 24 are used per game (shuffled).

### Styling Changes
- **New utilities**: Add to `SocOps/wwwroot/css/app.css` following existing patterns
- **Component styles**: Use `.razor.css` files for scoped styles
- **Theme colors**: Define in `:root` CSS variables for consistency

## Frontend Design Principles

**See `.github/instructions/frontend-design.instructions.md`** for detailed creative guidance. Key principles:
- **Avoid AI slop**: Choose distinctive fonts, cohesive color palettes, intentional animations
- **Motion matters**: Use CSS animations for page loads and win celebrations (staggered reveals via `animation-delay`)
- **Atmosphere**: Gradients and layered backgrounds over flat colors
- **Mobile experience**: Touch-friendly tap targets (at least 44×44px)

## Common Tasks

| Task | How |
|------|-----|
| **Add a question** | Edit `Questions.cs`, add to `QuestionsList` |
| **Change board size** | Update `BOARD_SIZE` and `CENTER_INDEX` in `BingoLogicService.cs` |
| **Customize win condition** | Modify `CheckBingo()` logic in `BingoLogicService.cs` |
| **Persist new game state** | Add property to `StoredGameData` class in `BingoGameService.cs` |
| **Add animations** | Use `.razor.css` files or `app.css`; favor CSS-only (no JS) |
| **Change colors/theme** | Update `:root` CSS vars in `app.css` or add new utility classes |

## File Organization

```
SocOps/
├── Components/          # Razor components (.razor files)
├── Pages/               # Routable pages (home, 404, etc.)
├── Services/            # Business logic (game service, win detection)
├── Models/              # C# data classes (GameState, BingoSquareData, etc.)
├── Data/                # Static data (Questions.cs)
├── Layout/              # MainLayout.razor + shared UI
├── wwwroot/css/         # Custom CSS utilities (app.css)
└── Program.cs           # Service registration, startup config
```

## Testing Locally

1. **Start the game**: Click "Start Game"
2. **Mark squares**: Tap any non-FREE square
3. **Win detection**: Mark 5 in a row (row, column, or diagonal) → modal appears
4. **Persistence**: Refresh page → board state restored
5. **Reset**: Click "Back" → return to start screen

## CI/CD & Deployment

- **Branch**: `main`
- **Action**: `.github/workflows/` (auto-build, deploy to `gh-pages`)
- **Live site**: [GitHub Pages link in README.md](README.md)

## Notes for AI Agents

- **Service lifecycle**: `BingoGameService` is scoped per session; state persists via localStorage
- **Change detection**: Blazor detects changes via property assignments and events; ensure `StateHasChanged()` is called after manual updates
- **JSInterop**: Used only for localStorage (low overhead, no external JS libs)
- **Error handling**: Wrapped in try-catch; logged to console; defaults to fresh state on load failure
