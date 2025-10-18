# Sear_Fight

# HTNL
```
<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Морской Бой: Радар + Самолёт</title>
  <link rel="stylesheet" href="sea_fight.css">
</head>
<body>

  <header>
    <button id="burgerBtn">✈️</button> 
    <div id="gameStatus">Расставьте корабли</div>
  </header>

  <!-- Подсказки -->
  <div class="hints">
    <div>Попаданий: <span id="user-hint">0</span>/20</div>
    <div>Осталось: <span id="comp-hint">20</span></div>
  </div>

  <!-- Палитра кораблей -->
  <div id="shipPalette"></div>

  <!-- Ориентация -->
  <div id="orientationControls">
    <button id="orientH" class="orient-btn active">Горизонтально</button>
    <button id="orientV" class="orient-btn">Вертикально</button>
  </div>

  <!-- Поля -->
  <div class="boards">
    <div class="field-container">
      <div class="field-title">Ваше поле</div>
      <div id="field-user" class="field-grid"></div>
    </div>
    <div class="field-container">
      <div class="field-title">Поле противника</div>
      <div id="field-comp" class="field-grid"></div>
    </div>
  </div>

  <!-- Бургер-меню -->
  <div id="burgerMenu">
    <h3>Способности</h3>
    <button id="btnRadar" class="menu-btn" disabled>📡 Радар (1 раз)</button>
    <button id="btnPlane" class="menu-btn" disabled>✈️ Удалить строку (1 раз)</button>
    <hr>
    <button id="btnNewGame" class="menu-btn">🔄 Новая игра</button>
  </div>

  <script src="sea_fight.js"></script>
</body>
</html>
```


# CSS
```
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: Arial, sans-serif;
}

body {
  background: #0a2e5c;
  color: white;
  padding: 12px;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
}

#burgerBtn {
  background: #3498db;
  border: none;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  font-size: 20px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

#gameStatus {
  font-size: 1.2em;
  font-weight: bold;
  text-align: center;
  padding: 0 10px;
}

.hints {
  display: flex;
  justify-content: space-around;
  margin: 12px 0;
  font-size: 1em;
}

/* Палитра кораблей */
#shipPalette {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  justify-content: center;
  margin: 12px 0;
}

.ship-item {
  padding: 6px 10px;
  background: #3498db;
  border-radius: 5px;
  cursor: pointer;
  font-size: 0.9em;
}

.ship-item.active {
  background: #2980b9;
  font-weight: bold;
}

/* Ориентация */
#orientationControls {
  text-align: center;
  margin: 10px 0;
  display: none; 
}

.orient-btn {
  padding: 6px 12px;
  margin: 0 5px;
  background: #7f8c8d;
  border: none;
  border-radius: 4px;
  color: white;
  cursor: pointer;
}

.orient-btn.active {
  background: #2ecc71;
}

/* Поля */
.boards {
  display: flex;
  justify-content: space-around;
  flex-wrap: wrap;
  gap: 15px;
  margin: 15px 0;
}

.field-container {
  text-align: center;
}

.field-title {
  margin-bottom: 8px;
  font-size: 1em;
}

.field-grid {
  display: grid;
  grid-template-columns: repeat(10, 28px);
  gap: 1px;
  background: #1a4a8c;
  padding: 4px;
  border-radius: 6px;
  width: fit-content;
  margin: 0 auto;
}

.cell {
  width: 28px;
  height: 28px;
  background: #4a90e2;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  color: transparent;
  user-select: none;
  cursor: pointer;
}

.cell.sheep { background: #555; }          /* Корабль игрока */
.cell.broken { background: #e74c3c; color: white; }
.cell.missed { background: #ecf0f1; }
.cell.radar-visible { background: #ff9800; } /* Обнаружен радаром */

/* Бургер-меню */
#burgerMenu {
  position: fixed;
  top: 0;
  right: -220px;
  width: 220px;
  height: 100vh;
  background: #2c3e50;
  transition: right 0.3s ease;
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 15px;
  z-index: 1000;
}

#burgerMenu.active {
  right: 0;
}

.menu-btn {
  width: 100%;
  padding: 10px;
  background: #3498db;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  text-align: left;
}

.menu-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
  background: #7f8c8d;
}

.menu-btn:hover:not(:disabled) {
  background: #2980b9;
}

/* Адаптив */
@media (max-width: 700px) {
  .boards {
    flex-direction: column;
    align-items: center;
  }
  .field-grid {
    grid-template-columns: repeat(10, 24px);
  }
  .cell {
    width: 24px;
    height: 24px;
    font-size: 12px;
  }
}
```


#JavaScript
```
// Глобальные переменные
let userField = [];
let compField = [];
let gameState = 'placing';
let selectedShipSize = null;
let shipOrientation = 'horizontal';
let placedShips = [];

let playerRadarUsed = false;
let playerPlaneUsed = false;
let compRadarUsed = false;
let compPlaneUsed = false;

let radarMode = false;
let planeMode = false;

let radarRevealedCells = new Set();

let compHitChain = []; 

const userHintEl = document.getElementById('user-hint');
const compHintEl = document.getElementById('comp-hint');
const gameStatusEl = document.getElementById('gameStatus');

const SHIP_CONFIG = [
  { size: 4, count: 1 },
  { size: 3, count: 2 },
  { size: 2, count: 3 },
  { size: 1, count: 4 }
];

function initGame() {
  userField = Array(10).fill().map(() => Array(10).fill('.'));
  compField = Array(10).fill().map(() => Array(10).fill('.'));
  placedShips = [];
  gameState = 'placing';
  selectedShipSize = null;
  shipOrientation = 'horizontal';
  playerRadarUsed = false;
  playerPlaneUsed = false;
  compRadarUsed = false;
  compPlaneUsed = false;
  radarMode = false;
  planeMode = false;
  radarRevealedCells = new Set();

  compHitChain = [];

  userHintEl.textContent = '0';
  compHintEl.textContent = '20';
  gameStatusEl.textContent = 'Расставьте корабли';

  document.getElementById('btnRadar').disabled = true;
  document.getElementById('btnPlane').disabled = true;
  document.getElementById('shipPalette').style.display = 'flex';
  document.getElementById('orientationControls').style.display = 'block';

  renderShipPalette();
  renderField('user');
  renderField('comp', true);
}

function renderShipPalette() {
  const palette = document.getElementById('shipPalette');
  palette.innerHTML = '';
  const placedCount = {};
  placedShips.forEach(ship => {
    placedCount[ship.size] = (placedCount[ship.size] || 0) + 1;
  });

  for (const config of SHIP_CONFIG) {
    const placed = placedCount[config.size] || 0;
    const remaining = config.count - placed;
    if (remaining > 0) {
      const el = document.createElement('div');
      el.className = 'ship-item';
      el.textContent = `Корабль ${config.size} кл. (${remaining})`;
      el.dataset.size = config.size;
      el.onclick = () => selectShip(config.size);
      palette.appendChild(el);
    }
  }
}

function selectShip(size) {
  if (gameState !== 'placing') return;
  selectedShipSize = size;
  document.querySelectorAll('.ship-item').forEach(el => el.classList.remove('active'));
  document.querySelector(`.ship-item[data-size="${size}"]`)?.classList.add('active');
  gameStatusEl.textContent = `Выбран корабль ${size} кл. (${shipOrientation})`;
}

function updateOrientationButtons() {
  document.getElementById('orientH').classList.toggle('active', shipOrientation === 'horizontal');
  document.getElementById('orientV').classList.toggle('active', shipOrientation === 'vertical');
}

document.getElementById('orientH').onclick = () => {
  if (gameState === 'placing') {
    shipOrientation = 'horizontal';
    updateOrientationButtons();
  }
};
document.getElementById('orientV').onclick = () => {
  if (gameState === 'placing') {
    shipOrientation = 'vertical';
    updateOrientationButtons();
  }
};

function canPlaceShip(field, row, col, size, horizontal) {
  if (horizontal && col + size > 10) return false;
  if (!horizontal && row + size > 10) return false;
  for (let i = 0; i < size; i++) {
    const r = horizontal ? row : row + i;
    const c = horizontal ? col + i : col;
    if (field[r][c] !== '.') return false;
    for (let dr = -1; dr <= 1; dr++) {
      for (let dc = -1; dc <= 1; dc++) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
          if (field[nr][nc] === '+') return false;
        }
      }
    }
  }
  return true;
}

function placeShip(field, row, col, size, horizontal) {
  for (let i = 0; i < size; i++) {
    const r = horizontal ? row : row + i;
    const c = horizontal ? col + i : col;
    field[r][c] = '+';
  }
  if (field === userField) {
    placedShips.push({ size, row, col, orientation: horizontal ? 'horizontal' : 'vertical' });
  }
}

function findShipAt(row, col) {
  return placedShips.find(ship => {
    if (ship.orientation === 'horizontal') {
      return ship.row === row && col >= ship.col && col < ship.col + ship.size;
    } else {
      return ship.col === col && row >= ship.row && row < ship.row + ship.size;
    }
  });
}

function removeShip(ship) {
  const { row, col, size, orientation } = ship;
  if (orientation === 'horizontal') {
    for (let c = col; c < col + size; c++) userField[row][c] = '.';
  } else {
    for (let r = row; r < row + size; r++) userField[r][col] = '.';
  }
  placedShips = placedShips.filter(s => s !== ship);
}

function renderField(role, isEnemy = false) {
  const field = role === 'user' ? userField : compField;
  const container = document.getElementById('field-' + role);
  container.innerHTML = '';

  for (let r = 0; r < 10; r++) {
    for (let c = 0; c < 10; c++) {
      const cell = document.createElement('div');
      cell.className = 'cell';

      if (field[r][c] === '+' && !isEnemy) cell.classList.add('sheep');
      if (field[r][c] === 'X') cell.classList.add('broken');
      if (field[r][c] === 'O') cell.classList.add('missed');

      const key = `${r},${c}`;
      if (radarRevealedCells.has(key) && compField[r][c] === '+') {
        cell.classList.add('radar-visible');
      }

      if (role === 'user' && gameState === 'placing') {
        cell.dataset.row = r;
        cell.dataset.col = c;
        cell.onclick = () => handleUserClick(r, c);
      } else if (role === 'comp' && gameState === 'playing') {
        cell.dataset.row = r;
        cell.dataset.col = c;
        cell.onclick = () => handleCompClick(r, c);
      }

      container.appendChild(cell);
    }
  }
}

function handleUserClick(row, col) {
  if (gameState !== 'placing') return;
  const shipHere = findShipAt(row, col);
  if (shipHere) {
    removeShip(shipHere);
    renderField('user');
    renderShipPalette();
    compHintEl.textContent = placedShips.reduce((sum, s) => sum + s.size, 0);
    return;
  }
  if (selectedShipSize === null) {
    gameStatusEl.textContent = 'Выберите корабль!';
    return;
  }
  if (canPlaceShip(userField, row, col, selectedShipSize, shipOrientation === 'horizontal')) {
    placeShip(userField, row, col, selectedShipSize, shipOrientation === 'horizontal');
    selectedShipSize = null;
    document.querySelectorAll('.ship-item').forEach(el => el.classList.remove('active'));
    renderField('user');
    renderShipPalette();
    const total = placedShips.reduce((sum, s) => sum + s.size, 0);
    compHintEl.textContent = total;
    if (total === 20) startGame();
  } else {
    gameStatusEl.textContent = '❌ Нельзя поставить сюда!';
  }
}

function startGame() {
  gameState = 'playing';
  gameStatusEl.textContent = 'Ваш ход';
  document.getElementById('shipPalette').style.display = 'none';
  document.getElementById('orientationControls').style.display = 'none';
  document.getElementById('btnRadar').disabled = false;
  document.getElementById('btnPlane').disabled = false;
  renderField('user');
  renderField('comp', true);

  compField = Array(10).fill().map(() => Array(10).fill('.'));
  const allShips = [];
  SHIP_CONFIG.forEach(cfg => {
    for (let i = 0; i < cfg.count; i++) allShips.push(cfg.size);
  });
  for (const size of allShips) {
    let placed = false;
    while (!placed) {
      const hor = Math.random() > 0.5;
      const r = Math.floor(Math.random() * (hor ? 10 : 10 - size + 1));
      const c = Math.floor(Math.random() * (hor ? 10 - size + 1 : 10));
      if (canPlaceShip(compField, r, c, size, hor)) {
        placeShip(compField, r, c, size, hor);
        placed = true;
      }
    }
  }
}

function checkWin(field) {
  return field.flat().every(cell => cell !== '+');
}

function updateHitCount() {
  const hits = compField.flat().filter(x => x === 'X').length;
  userHintEl.textContent = hits;
  return hits;
}

function markNeighborsAsMiss(field, shipCells) {
  const toMark = [];
  for (const [r, c] of shipCells) {
    for (let dr = -1; dr <= 1; dr++) {
      for (let dc = -1; dc <= 1; dc++) {
        const nr = r + dr;
        const nc = c + dc;
        if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
          if (!shipCells.some(([sr, sc]) => sr === nr && sc === nc)) {
            if (field[nr][nc] === '.') {
              toMark.push([nr, nc]);
            }
          }
        }
      }
    }
  }
  for (const [r, c] of toMark) {
    field[r][c] = 'O';
  }
}

function checkAndMarkSunk(field, lastRow, lastCol) {
  const visited = new Set();
  const stack = [[lastRow, lastCol]];
  const shipCells = [];

  while (stack.length > 0) {
    const [r, c] = stack.pop();
    const key = `${r},${c}`;
    if (visited.has(key)) continue;
    visited.add(key);
    if (field[r][c] === 'X') {
      shipCells.push([r, c]);
      for (const [dr, dc] of [[-1,0],[1,0],[0,-1],[0,1]]) {
        const nr = r + dr, nc = c + dc;
        if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10 && field[nr][nc] === 'X') {
          if (!visited.has(`${nr},${nc}`)) {
            stack.push([nr, nc]);
          }
        }
      }
    }
  }

  if (shipCells.length === 0) return false;

  for (const [r, c] of shipCells) {
    for (const [dr, dc] of [[-1,0],[1,0],[0,-1],[0,1]]) {
      const nr = r + dr, nc = c + dc;
      if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
        if (field[nr][nc] === '+') {
          return false;
        }
      }
    }
  }

  markNeighborsAsMiss(field, shipCells);
  return true;
}

function handleCompClick(row, col) {
  if (radarMode) {
    useRadar(row, col);
    radarMode = false;
    document.getElementById('btnRadar').disabled = true;
    playerRadarUsed = true;
    setTimeout(() => {
      compMove();
    }, 600);
    return;
  }

  if (planeMode) {
    usePlane(row);
    planeMode = false;
    document.getElementById('btnPlane').disabled = true;
    playerPlaneUsed = true;
    setTimeout(() => {
      compMove();
    }, 600);
    return;
  }

  if (compField[row][col] === '.') {
    compField[row][col] = 'O';
    renderField('comp', true);
    setTimeout(() => {
      compMove();
    }, 600);
  } else if (compField[row][col] === '+') {
    compField[row][col] = 'X';
    const sunk = checkAndMarkSunk(compField, row, col);
    renderField('comp', true);
    updateHitCount();
    if (checkWin(compField)) {
      alert('🏆 Вы победили!');
      return;
    }
    if (sunk) {
      gameStatusEl.textContent = 'Убит! Ходите снова.';
    } else {
      gameStatusEl.textContent = 'Попадание! Ходите снова.';
    }
    return;
  }
}

document.getElementById('btnRadar').onclick = () => {
  if (playerRadarUsed || gameState !== 'playing') return;
  radarMode = true;
  document.getElementById('burgerMenu').classList.remove('active');
  gameStatusEl.textContent = '📡 Кликните на клетку противника (3×3)';
};

document.getElementById('btnPlane').onclick = () => {
  if (playerPlaneUsed || gameState !== 'playing') return;
  planeMode = true;
  document.getElementById('burgerMenu').classList.remove('active');
  gameStatusEl.textContent = '✈️ Кликните на клетку — уничтожит строку!';
};

function useRadar(centerRow, centerCol) {
  for (let dr = -1; dr <= 1; dr++) {
    for (let dc = -1; dc <= 1; dc++) {
      const r = centerRow + dr;
      const c = centerCol + dc;
      if (r >= 0 && r < 10 && c >= 0 && c < 10) {
        if (compField[r][c] === '+') {
          radarRevealedCells.add(`${r},${c}`);
        }
      }
    }
  }
  renderField('comp', true);
  updateHitCount();
  if (checkWin(compField)) {
    alert('🏆 Вы победили!');
  }
}

function usePlane(row) {
  for (let c = 0; c < 10; c++) {
    if (compField[row][c] === '+') compField[row][c] = 'X';
    else if (compField[row][c] === '.') compField[row][c] = 'O';
  }
  for (let c = 0; c < 10; c++) {
    if (compField[row][c] === 'X') {
      checkAndMarkSunk(compField, row, c);
    }
  }
  renderField('comp', true);
  updateHitCount();
  if (checkWin(compField)) {
    alert('🏆 Вы победили!');
  }
}

// === Ход компьютера ===
function compMove() {
  if (gameState !== 'playing') return;

  gameStatusEl.textContent = 'Ход компьютера...';

  setTimeout(() => {
    // === 1. Если есть активная цепочка попаданий — продолжаем добивать ===
if (compHitChain.length > 0) {
  let nextTarget = null;
  let directionKnown = false;
  let horizontal = false;
  let vertical = false;

  // Определяем, есть ли уже направление
  if (compHitChain.length >= 2) {
    const rows = compHitChain.map(p => p[0]);
    const cols = compHitChain.map(p => p[1]);
    horizontal = new Set(rows).size === 1;
    vertical = new Set(cols).size === 1;
    directionKnown = horizontal || vertical;
  }

  if (directionKnown) {
    // Продолжаем в известном направлении
    if (horizontal) {
      const r = compHitChain[0][0];
      const minC = Math.min(...compHitChain.map(p => p[1]));
      const maxC = Math.max(...compHitChain.map(p => p[1]));
      if (maxC + 1 < 10 && userField[r][maxC + 1] !== 'X' && userField[r][maxC + 1] !== 'O') {
        nextTarget = [r, maxC + 1];
      } else if (minC - 1 >= 0 && userField[r][minC - 1] !== 'X' && userField[r][minC - 1] !== 'O') {
        nextTarget = [r, minC - 1];
      }
    } else if (vertical) {
      const c = compHitChain[0][1];
      const minR = Math.min(...compHitChain.map(p => p[0]));
      const maxR = Math.max(...compHitChain.map(p => p[0]));
      if (maxR + 1 < 10 && userField[maxR + 1][c] !== 'X' && userField[maxR + 1][c] !== 'O') {
        nextTarget = [maxR + 1, c];
      } else if (minR - 1 >= 0 && userField[minR - 1][c] !== 'X' && userField[minR - 1][c] !== 'O') {
        nextTarget = [minR - 1, c];
      }
    }
  } else {
    // Только одно попадание — пробуем непроверенные соседние клетки
    const [r0, c0] = compHitChain[0];
    const directions = [
      [r0 - 1, c0], // вверх
      [r0 + 1, c0], // вниз
      [r0, c0 - 1], // влево
      [r0, c0 + 1]  // вправо
    ];

    // Ищем первую непроверенную клетку
    for (const [nr, nc] of directions) {
      if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
        if (userField[nr][nc] !== 'X' && userField[nr][nc] !== 'O') {
          nextTarget = [nr, nc];
          break;
        }
      }
    }
  }

  if (nextTarget) {
    const [r, c] = nextTarget;
    if (userField[r][c] === '+') {
      userField[r][c] = 'X';
      compHitChain.push([r, c]);
      const sunk = checkAndMarkSunk(userField, r, c);
      renderField('user');

      if (checkWin(userField)) {
        alert('💀 Компьютер победил!');
        return;
      }

      if (sunk) {
        compHitChain = []; // корабль убит — сброс
        gameStatusEl.textContent = 'Ваш ход';
      } else {
        // Продолжаем ходить
        setTimeout(compMove, 600);
        return;
      }
    } else {
      // Промах — помечаем
      userField[r][c] = 'O';
      renderField('user');

      // Проверим, остались ли непроверенные направления от первой точки
      if (compHitChain.length === 1) {
        const [r0, c0] = compHitChain[0];
        const directions = [
          [r0 - 1, c0],
          [r0 + 1, c0],
          [r0, c0 - 1],
          [r0, c0 + 1]
        ];
        const hasUnshot = directions.some(([nr, nc]) =>
          nr >= 0 && nr < 10 && nc >= 0 && nc < 10 &&
          userField[nr][nc] !== 'X' && userField[nr][nc] !== 'O'
        );

        if (!hasUnshot) {
          // Все направления проверены — можно сбросить
          compHitChain = [];
        }
        // Иначе — оставляем цепочку для следующего хода
      }

      gameStatusEl.textContent = 'Ваш ход';
    }
    return;
  } else {
    // Нет куда стрелять — сбрасываем
    compHitChain = [];
  }
}

    // === 2. РАДАР ===
    if (!compRadarUsed) {
      for (let r = 0; r < 10; r++) {
        for (let c = 0; c < 10; c++) {
          if (userField[r][c] === 'X') {
            const neighbors = [[r-1,c], [r+1,c], [r,c-1], [r,c+1]];
            const hasUnshot = neighbors.some(
              ([nr, nc]) => nr >= 0 && nr < 10 && nc >= 0 && nc < 10 &&
                           userField[nr][nc] !== 'X' && userField[nr][nc] !== 'O'
            );
            if (hasUnshot) {
              compRadarUsed = true;
              gameStatusEl.textContent = '🤖 Компьютер использует РАДАР!';

              setTimeout(() => {
                let hitOccurred = false;
                for (let dr = -1; dr <= 1; dr++) {
                  for (let dc = -1; dc <= 1; dc++) {
                    const nr = r + dr;
                    const nc = c + dc;
                    if (nr >= 0 && nr < 10 && nc >= 0 && nc < 10) {
                      if (userField[nr][nc] === '+') {
                        userField[nr][nc] = 'X';
                        compHitChain.push([nr, nc]);
                        checkAndMarkSunk(userField, nr, nc);
                        hitOccurred = true;
                      } else if (userField[nr][nc] === '.') {
                        userField[nr][nc] = 'O';
                      }
                    }
                  }
                }
                renderField('user');
                if (checkWin(userField)) {
                  alert('💀 Компьютер победил!');
                  return;
                }
                if (hitOccurred && compHitChain.length > 0) {
                  setTimeout(compMove, 600);
                } else {
                  compHitChain = [];
                  gameStatusEl.textContent = 'Ваш ход';
                }
              }, 600);
              return;
            }
          }
        }
      }
    }

    // === 3. Самолёт ===
    const playerShipsLeft = userField.flat().filter(cell => cell === '+').length;
    if (!compPlaneUsed && playerShipsLeft >= 8) {
      compPlaneUsed = true;
      const row = Math.floor(Math.random() * 10);
      gameStatusEl.textContent = `🤖 Компьютер запускает САМОЛЁТ по строке ${row + 1}!`;

      setTimeout(() => {
        let hitOccurred = false;
        for (let c = 0; c < 10; c++) {
          if (userField[row][c] === '+') {
            userField[row][c] = 'X';
            compHitChain.push([row, c]);
            checkAndMarkSunk(userField, row, c);
            hitOccurred = true;
          } else if (userField[row][c] === '.') {
            userField[row][c] = 'O';
          }
        }
        renderField('user');
        if (checkWin(userField)) {
          alert('💀 Компьютер победил!');
          return;
        }
        if (hitOccurred) {
          setTimeout(compMove, 600);
        } else {
          gameStatusEl.textContent = 'Ваш ход';
        }
      }, 600);
      return;
    }

    // === 4. Случайный выстрел по непроверенным клеткам ===
    const candidates = [];
    for (let r = 0; r < 10; r++) {
      for (let c = 0; c < 10; c++) {
        if (userField[r][c] !== 'X' && userField[r][c] !== 'O') {
          candidates.push([r, c]);
        }
      }
    }

    if (candidates.length === 0) {
      gameStatusEl.textContent = 'Ваш ход';
      return;
    }

    const [row, col] = candidates[Math.floor(Math.random() * candidates.length)];

    if (userField[row][col] === '+') {
      userField[row][col] = 'X';
      compHitChain = [[row, col]];
      const sunk = checkAndMarkSunk(userField, row, col);
      renderField('user');

      if (checkWin(userField)) {
        alert('💀 Компьютер победил!');
        return;
      }

      if (!sunk) {
        setTimeout(compMove, 600);
        return;
      } else {
        compHitChain = [];
        gameStatusEl.textContent = 'Ваш ход';
      }
    } else {
      userField[row][col] = 'O';
      renderField('user');
      gameStatusEl.textContent = 'Ваш ход';
    }
  }, 400);
}

document.getElementById('burgerBtn').onclick = () => {
  document.getElementById('burgerMenu').classList.toggle('active');
};

document.getElementById('btnNewGame').onclick = () => {
  document.getElementById('burgerMenu').classList.remove('active');
  initGame();
};

initGame();
```
