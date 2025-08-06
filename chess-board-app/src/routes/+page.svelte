<script>
  import ChessBoard from '$lib/ChessBoard.svelte';
  import Navbar from '$lib/Navbar.svelte';
  
  let showPieces = true;
  let flipped = false;
  let roundLength = 8;
  let gameActive = false;
  let showTimer = true;
  let currentTime = 0;
  let timerInterval;
  let darkMode = false;
  
  // Game state
  let targetSquare = '';
  let correctSquares = [];
  let incorrectSquares = [];
  let currentRound = [];
  let currentIndex = 0;
  let score = 0;
  let misclicks = 0;

  // Generate all possible squares
  const files = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
  const ranks = ['8', '7', '6', '5', '4', '3', '2', '1'];
  const allSquares = files.flatMap(file => ranks.map(rank => file + rank));

  function handleTogglePieces(event) {
    showPieces = event.detail;
  }

  function handleToggleFlip(event) {
    flipped = event.detail;
  }

  function handleChangeRoundLength(event) {
    roundLength = event.detail;
  }

  function handleToggleTimer(event) {
    showTimer = event.detail;
  }

  function handleToggleDarkMode(event) {
    darkMode = event.detail;
  }

  function startTimer() {
    currentTime = 0;
    timerInterval = setInterval(() => {
      currentTime += 100;
    }, 100);
  }

  function stopTimer() {
    if (timerInterval) {
      clearInterval(timerInterval);
      timerInterval = null;
    }
  }

  function generateRandomRound(length) {
    const shuffled = [...allSquares].sort(() => Math.random() - 0.5);
    return shuffled.slice(0, length);
  }

  function startGame() {
    gameActive = true;
    currentRound = generateRandomRound(roundLength);
    currentIndex = 0;
    score = 0;
    misclicks = 0;
    correctSquares = [];
    incorrectSquares = [];
    targetSquare = currentRound[0];
    gameStartTime = Date.now();
    startTimer();
  }

  function endGame() {
    gameActive = false;
    targetSquare = '';
    correctSquares = [];
    incorrectSquares = [];
    currentRound = [];
    currentIndex = 0;
    score = 0;
    misclicks = 0;
    stopTimer();
  }

  // Game end modal state
  let showEndModal = false;
  let gameEndMessage = '';
  let gameTime = 0;
  let gameStartTime = 0;

  function handleSquareClicked(event) {
    const { square, isCorrect } = event.detail;
    
    if (isCorrect) {
      correctSquares = [...correctSquares, square];
      currentIndex++;
      
      if (currentIndex < currentRound.length) {
        targetSquare = currentRound[currentIndex];
        // Clear incorrect squares for next target
        incorrectSquares = [];
      } else {
        // Round completed
        stopTimer();
        gameTime = currentTime;
        gameEndMessage = `Congratulations! You completed the round with ${currentIndex}/${roundLength} correct answers!`;
        showEndModal = true;
        // Don't call endGame() here since we need the misclicks count for the modal
        gameActive = false;
        targetSquare = '';
        correctSquares = [];
        incorrectSquares = [];
      }
    } else {
      misclicks++;
      if (!incorrectSquares.includes(square)) {
        incorrectSquares = [...incorrectSquares, square];
      }
    }
  }

  function closeEndModal() {
    showEndModal = false;
    // Reset remaining game state
    currentRound = [];
    currentIndex = 0;
    score = 0;
    misclicks = 0;
  }
</script>

<div class="app {darkMode ? 'dark-mode' : ''}">
  <Navbar 
    {showPieces} 
    {flipped}
    {roundLength}
    {gameActive}
    {showTimer}
    {darkMode}
    on:togglePieces={handleTogglePieces} 
    on:toggleFlip={handleToggleFlip}
    on:changeRoundLength={handleChangeRoundLength}
    on:toggleTimer={handleToggleTimer}
    on:toggleDarkMode={handleToggleDarkMode}
    on:startGame={startGame}
    on:endGame={endGame}
  />
  
  <main>
    <div class="game-status-placeholder">
      {#if gameActive}
        <div class="game-status">
          <p class="progress">Progress: {currentIndex}/{roundLength} ({Math.round((currentIndex/roundLength) * 100)}%)</p>
          <p class="current">Current Target: <strong>{currentIndex + 1}</strong> of <strong>{roundLength}</strong></p>
          {#if showTimer}
            <div class="timer-display">
              Time: {(currentTime / 1000).toFixed(1)}s
            </div>
          {/if}
        </div>
      {/if}
    </div>
    
    <ChessBoard 
      {showPieces} 
      {flipped}
      {targetSquare}
      {gameActive}
      {correctSquares}
      {incorrectSquares}
      {darkMode}
      on:squareClicked={handleSquareClicked}
    />
  </main>
</div>

{#if showEndModal}
  <div 
    class="modal-overlay" 
    on:click={closeEndModal}
    on:keydown={(e) => e.key === 'Escape' && closeEndModal()}
    role="dialog"
    aria-modal="true"
    tabindex="-1"
  >
    <div 
      class="modal" 
      role="document"
      on:click|stopPropagation
    >
      <h2>Game Complete!</h2>
      <p class="end-message">{gameEndMessage}</p>
      <p class="misclicks-result">Misclicks: {misclicks}</p>
      {#if showTimer}
        <p class="time-result">Time: {(gameTime / 1000).toFixed(2)} seconds</p>
      {/if}
      <button class="close-button" on:click={closeEndModal}>Close</button>
    </div>
  </div>
{/if}

<style>
  .app {
    min-height: 100vh;
    font-family: Arial, sans-serif;
    background-color: #f5f5f5;
    transition: background-color 0.3s ease;
  }

  .app.dark-mode {
    background-color: #2a2a2a;
  }

  main {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    padding: 2rem;
    min-height: calc(100vh - 80px);
    gap: 1rem;
  }

  .game-status-placeholder {
    height: 130px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  :global(body) {
    margin: 0;
    padding: 0;
  }

  .game-status {
    text-align: center;
    padding: 1rem;
    background-color: #ffffff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    transition: background-color 0.3s ease, color 0.3s ease;
    min-width: 300px;
    max-width: 400px;
  }

  .app.dark-mode .game-status {
    background-color: #3a3a3a;
    box-shadow: 0 2px 4px rgba(0,0,0,0.3);
  }

  .game-status p {
    margin: 0.5rem 0;
    font-size: 1.1rem;
  }

  .progress {
    color: #00b894;
    font-weight: bold;
  }

  .current {
    color: #312e2b;
  }

  .app.dark-mode .current {
    color: #e0e0e0;
  }

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
  }

  .modal {
    background: white;
    padding: 2rem;
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3);
    text-align: center;
    max-width: 400px;
    width: 90%;
    transition: background-color 0.3s ease, color 0.3s ease;
  }

  .app.dark-mode .modal {
    background: #3a3a3a;
    color: #e0e0e0;
  }

  .modal h2 {
    color: #312e2b;
    margin-bottom: 1rem;
    font-size: 1.8rem;
  }

  .app.dark-mode .modal h2 {
    color: #e0e0e0;
  }

  .end-message {
    font-size: 1.2rem;
    color: #00b894;
    font-weight: bold;
    margin: 1rem 0;
  }

  .time-result {
    font-size: 1.1rem;
    color: #312e2b;
    margin: 1rem 0;
  }

  .app.dark-mode .time-result {
    color: #e0e0e0;
  }

  .misclicks-result {
    font-size: 1.1rem;
    color: #e17055;
    font-weight: bold;
    margin: 1rem 0;
  }

  .close-button {
    background-color: #312e2b;
    color: white;
    border: none;
    padding: 0.75rem 1.5rem;
    border-radius: 6px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.2s;
  }

  .close-button:hover {
    background-color: #2c2a27;
  }

  .timer-display {
    background-color: #f0d9b5;
    color: #312e2b;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    font-weight: bold;
    font-size: 1.1rem;
    margin-top: 0.5rem;
    display: inline-block;
    transition: background-color 0.3s ease, color 0.3s ease;
  }

  .app.dark-mode .timer-display {
    background-color: #4a4a4a;
    color: #e0e0e0;
  }

  @media (max-width: 768px) {
    main {
      padding: 1rem;
    }

    .game-status {
      min-width: auto;
      max-width: 90%;
    }
  }
</style>
