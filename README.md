<!DOCTYPE html>
<html lang="zh-TW">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>猜拳九宮格圈圈叉叉</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
      font-family: "Microsoft JhengHei", "PingFang TC", sans-serif;
    }

    body {
      background-color: #f4f6f9;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 20px;
      color: #333;
    }

    h1 {
      margin-bottom: 12px;
      font-size: 1.9rem;
      color: #2c3e50;
    }

    /* 狀態與猜拳控制區 */
    .control-panel {
      background-color: #fff;
      padding: 15px 25px;
      border-radius: 16px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.08);
      margin-bottom: 20px;
      text-align: center;
      width: 100%;
      max-width: 440px;
    }

    .status {
      font-size: 1.2rem;
      font-weight: bold;
      margin-bottom: 12px;
      color: #2c3e50;
    }

    .rps-buttons {
      display: flex;
      gap: 12px;
      justify-content: center;
    }

    .rps-btn {
      flex: 1;
      padding: 12px 10px;
      font-size: 1.1rem;
      font-weight: bold;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      color: #fff;
      transition: transform 0.1s, opacity 0.2s;
    }

    .rps-btn:active {
      transform: scale(0.96);
    }

    .rps-btn.o-win {
      background-color: #2ecc71;
      box-shadow: 0 4px 0 #27ae60;
    }

    .rps-btn.x-win {
      background-color: #ff4757;
      box-shadow: 0 4px 0 #c0392b;
    }

    .rps-btn:disabled {
      opacity: 0.35;
      cursor: not-allowed;
      box-shadow: none;
    }

    /* 九宮格容器 */
    .board {
      display: grid;
      grid-template-columns: repeat(3, 140px);
      grid-template-rows: repeat(3, 140px);
      gap: 10px;
      background-color: #2c3e50;
      padding: 10px;
      border-radius: 16px;
      box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
      transition: opacity 0.25s ease;
    }

    /* 尚未猜拳決定贏家時，棋盤鎖定狀態 */
    .board.locked {
      opacity: 0.7;
    }

    .board.locked .cell:not(.taken) {
      cursor: not-allowed;
    }

    /* 單一格子 */
    .cell {
      position: relative;
      background-color: #ffffff;
      border-radius: 10px;
      overflow: hidden;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: transform 0.15s ease;
    }

    .board:not(.locked) .cell:hover:not(.taken) {
      transform: scale(1.03);
      box-shadow: 0 0 0 3px #f1c40f;
    }

    /* 背景圖片設定 */
    .cell img.bg-img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      display: block;
      transition: filter 0.3s ease;
    }

    /* 特殊圖示格（蛋糕、星星）背景 */
    .cell.special-cell {
      background: linear-gradient(135deg, #fff9e6 0%, #ffecb3 100%);
      flex-direction: column;
      padding: 10px;
      text-align: center;
    }

    .cell.special-cell span.label {
      font-size: 0.75rem;
      color: #8d6e63;
      margin-top: 4px;
      font-weight: bold;
    }

    /* 當格子被點擊後，底圖稍微變暗以凸顯 O 或 X */
    .cell.taken img.bg-img,
    .cell.taken .special-icon {
      filter: brightness(0.45);
    }

    /* O 與 X 覆蓋層 */
    .mark {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 5.5rem;
      font-weight: 900;
      pointer-events: none;
      text-shadow: 0 4px 12px rgba(0, 0, 0, 0.6);
      animation: popIn 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
    }

    .mark.o {
      color: #2ecc71;
    }

    .mark.x {
      color: #ff4757;
    }

    /* 連線獲勝高亮 */
    .cell.win {
      box-shadow: inset 0 0 0 5px #f1c40f;
    }

    @keyframes popIn {
      0% { transform: scale(0.3); opacity: 0; }
      100% { transform: scale(1); opacity: 1; }
    }

    /* 重新開始按鈕 */
    .restart-btn {
      margin-top: 22px;
      padding: 12px 30px;
      font-size: 1.05rem;
      font-weight: bold;
      color: #fff;
      background-color: #3498db;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      box-shadow: 0 4px 10px rgba(52, 152, 219, 0.3);
      transition: background-color 0.2s;
    }

    .restart-btn:hover {
      background-color: #2980b9;
    }

    /* 手機版自適應縮小 */
    @media (max-width: 480px) {
      .board {
        grid-template-columns: repeat(3, 100px);
        grid-template-rows: repeat(3, 100px);
        gap: 8px;
      }
      .mark {
        font-size: 4rem;
      }
    }
  </style>
</head>
<body>

  <h1>剪刀石頭布・九宮格</h1>

  <div class="control-panel">
    <div class="status" id="status">✊✌️🖐️ 請先猜拳，點選本回合贏家：</div>
    <div class="rps-buttons">
      <button class="rps-btn o-win" id="btnOWin">O 猜拳贏了</button>
      <button class="rps-btn x-win" id="btnXWin">X 猜拳贏了</button>
    </div>
  </div>

  <div class="board locked" id="board">
    <!-- 格子 0：全聯 -->
    <div class="cell" data-index="0">
      <img class="bg-img" src="全聯.jpg" alt="全聯" onerror="this.style.display='none'">
    </div>

    <!-- 格子 1：公園 -->
    <div class="cell" data-index="1">
      <img class="bg-img" src="公園.jpg" alt="公園" onerror="this.style.display='none'">
    </div>

    <!-- 格子 2：郵局 -->
    <div class="cell" data-index="2">
      <img class="bg-img" src="郵局.jpg" alt="郵局" onerror="this.style.display='none'">
    </div>

    <!-- 格子 3：圖書館 -->
    <div class="cell" data-index="3">
      <img class="bg-img" src="圖書館.jpg" alt="圖書館" onerror="this.style.display='none'">
    </div>

    <!-- 格子 4（正中央）：切半蛋糕 A piece of cake -->
    <div class="cell special-cell" data-index="4">
      <div class="special-icon">
        <svg width="70" height="70" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
          <ellipse cx="50" cy="82" rx="42" ry="10" fill="#e0e0e0" stroke="#bdbdbd" stroke-width="2"/>
          <polygon points="20,45 80,55 80,78 20,68" fill="#ffe0b2" stroke="#d84315" stroke-width="2"/>
          <polygon points="20,45 55,30 80,55" fill="#fff8e1" stroke="#d84315" stroke-width="2"/>
          <line x1="20" y1="56" x2="80" y2="66" stroke="#ff5252" stroke-width="4"/>
          <circle cx="52" cy="36" r="7" fill="#e53935"/>
          <path d="M48,30 Q52,25 56,30" stroke="#43a047" stroke-width="2.5" fill="none"/>
        </svg>
      </div>
      <span class="label">A piece of cake</span>
    </div>

    <!-- 格子 5：醫院 -->
    <div class="cell" data-index="5">
      <img class="bg-img" src="醫院.jpg" alt="醫院" onerror="this.style.display='none'">
    </div>

    <!-- 格子 6：銀行 -->
    <div class="cell" data-index="6">
      <img class="bg-img" src="銀行.jpg" alt="銀行" onerror="this.style.display='none'">
    </div>

    <!-- 格子 7：麵包店 -->
    <div class="cell" data-index="7">
      <img class="bg-img" src="麵包店.jpg" alt="麵包店" onerror="this.style.display='none'">
    </div>

    <!-- 格子 8（右下）：星星 -->
    <div class="cell special-cell" data-index="8">
      <div class="special-icon">
        <svg width="75" height="75" viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg">
          <polygon points="50,10 61,38 91,38 67,56 76,85 50,67 24,85 33,56 9,38 39,38" 
                   fill="#ffd54f" stroke="#f57f17" stroke-width="3" stroke-linejoin="round"/>
        </svg>
      </div>
      <span class="label">Star</span>
    </div>
  </div>

  <button class="restart-btn" id="restartBtn">重新開始整局遊戲</button>

  <script>
    const board = document.getElementById('board');
    const cells = document.querySelectorAll('.cell');
    const statusText = document.getElementById('status');
    const btnOWin = document.getElementById('btnOWin');
    const btnXWin = document.getElementById('btnXWin');
    const restartBtn = document.getElementById('restartBtn');

    let roundWinner = null; // null 代表還沒猜拳，'O' 或 'X' 代表這回合的贏家
    let gameState = ['', '', '', '', '', '', '', '', ''];
    let gameActive = true;

    const winningConditions = [
      [0, 1, 2],
      [3, 4, 5],
      [6, 7, 8],
      [0, 3, 6],
      [1, 4, 7],
      [2, 5, 8],
      [0, 4, 8],
      [2, 4, 6]
    ];

    // 選擇本回合猜拳贏家
    function selectRoundWinner(player) {
      if (!gameActive) return;
      roundWinner = player;
      board.classList.remove('locked');
      btnOWin.disabled = true;
      btnXWin.disabled = true;

      const color = player === 'O' ? '#2ecc71' : '#ff4757';
      statusText.innerHTML = `👉 請 <span style="color:${color}; font-size:1.4rem;">${player}</span> 點選一格放上標記！`;
    }

    // 點擊九宮格放上標記
    function handleCellClick(e) {
      const cell = e.currentTarget;
      const index = parseInt(cell.getAttribute('data-index'));

      // 如果還沒猜拳決定贏家、格子已被佔領、或遊戲已結束，則不動作
      if (!roundWinner || gameState[index] !== '' || !gameActive) {
        return;
      }

      gameState[index] = roundWinner;
      cell.classList.add('taken');

      const mark = document.createElement('div');
      mark.classList.add('mark', roundWinner.toLowerCase());
      mark.textContent = roundWinner;
      cell.appendChild(mark);

      checkResult();
    }

    function checkResult() {
      let roundWon = false;
      let winningCombo = [];

      for (let i = 0; i < winningConditions.length; i++) {
        const [a, b, c] = winningConditions[i];
        if (gameState[a] && gameState[a] === gameState[b] && gameState[a] === gameState[c]) {
          roundWon = true;
          winningCombo = [a, b, c];
          break;
        }
      }

      if (roundWon) {
        const color = roundWinner === 'O' ? '#2ecc71' : '#ff4757';
        statusText.innerHTML = `🎉 恭喜 <span style="color:${color}; font-size:1.4rem;">${roundWinner}</span> 連線獲勝！`;
        gameActive = false;
        roundWinner = null;
        board.classList.remove('locked');
        btnOWin.disabled = true;
        btnXWin.disabled = true;
        winningCombo.forEach(idx => cells[idx].classList.add('win'));
        return;
      }

      if (!gameState.includes('')) {
        statusText.textContent = '🤝 格子全滿，雙方平手！';
        gameActive = false;
        roundWinner = null;
        btnOWin.disabled = true;
        btnXWin.disabled = true;
        return;
      }

      // 放完一格後，重置為等待下一輪猜拳狀態
      roundWinner = null;
      board.classList.add('locked');
      btnOWin.disabled = false;
      btnXWin.disabled = false;
      statusText.textContent = '✊✌️🖐️ 下一回合！請先猜拳再點選贏家：';
    }

    function restartGame() {
      roundWinner = null;
      gameState = ['', '', '', '', '', '', '', '', ''];
      gameActive = true;
      board.classList.add('locked');
      btnOWin.disabled = false;
      btnXWin.disabled = false;
      statusText.textContent = '✊✌️🖐️ 請先猜拳，點選本回合贏家：';

      cells.forEach(cell => {
        cell.classList.remove('taken', 'win');
        const existingMark = cell.querySelector('.mark');
        if (existingMark) {
          existingMark.remove();
        }
      });
    }

    btnOWin.addEventListener('click', () => selectRoundWinner('O'));
    btnXWin.addEventListener('click', () => selectRoundWinner('X'));
    cells.forEach(cell => cell.addEventListener('click', handleCellClick));
    restartBtn.addEventListener('click', restartGame);
  </script>

</body>
</html>
