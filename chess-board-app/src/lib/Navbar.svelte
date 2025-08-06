<script lang="ts">
  import { createEventDispatcher } from 'svelte';
  
  export let showPieces = true;
  export let flipped = false;
  export let roundLength = 8;
  export let gameActive = false;
  export let showTimer = true;
  export let darkMode = false;
  
  const dispatch = createEventDispatcher();
  const roundOptions = [8, 16, 32, 64];
  let customRoundLength = 16;
  let showCustomInput = false;

  function togglePieces() {
    showPieces = !showPieces;
    dispatch('togglePieces', showPieces);
  }

  function toggleFlip() {
    dispatch('toggleFlip', flipped);
  }

  function handleRoundLengthChange(event: Event) {
    if (!gameActive) {
      const value = (event.target as HTMLSelectElement).value;
      if (value === 'custom') {
        showCustomInput = true;
        roundLength = customRoundLength;
        dispatch('changeRoundLength', customRoundLength);
      } else {
        showCustomInput = false;
        roundLength = parseInt(value);
        dispatch('changeRoundLength', roundLength);
      }
    }
  }

  function handleCustomLengthBlur(event: Event) {
    if (!gameActive) {
      let value = parseInt((event.target as HTMLInputElement).value);
      
      // Ensure it's a valid integer between 1 and 64
      if (isNaN(value) || value < 1) {
        value = 1;
      } else if (value > 64) {
        value = 64;
      }
      
      customRoundLength = value;
      roundLength = value;
      (event.target as HTMLInputElement).value = value.toString(); // Update the input field
      dispatch('changeRoundLength', value);
    }
  }

  function toggleTimer() {
    dispatch('toggleTimer', showTimer);
  }

  function toggleDarkMode() {
    dispatch('toggleDarkMode', darkMode);
  }

  function startGame() {
    dispatch('startGame');
  }

  function endGame() {
    dispatch('endGame');
  }
</script>

<nav class="navbar">
  <div class="nav-brand">
    <h2>Chess Square Memorizer</h2>
  </div>
  
  <div class="nav-controls">
    <button 
      class="nav-button {showPieces ? 'active' : ''}"
      on:click={togglePieces}
    >
      {showPieces ? 'Hide Pieces' : 'Show Pieces'}
    </button>
    
    <div class="toggle-container">
      <span class="toggle-label">Board View:</span>
      <div class="toggle-switch">
        <input 
          type="checkbox" 
          bind:checked={flipped} 
          on:change={toggleFlip}
          id="board-flip"
          disabled={gameActive}
        />
        <label for="board-flip" class="toggle-slider">
          <span class="toggle-text">{flipped ? 'Black' : 'White'}</span>
        </label>
      </div>
    </div>

    <div class="round-selector">
      <label for="round-select">Round Length:</label>
      <select 
        id="round-select"
        value={showCustomInput ? 'custom' : roundLength}
        on:change={handleRoundLengthChange}
        disabled={gameActive}
      >
        {#each roundOptions as option}
          <option value={option}>{option}</option>
        {/each}
        <option value="custom">Custom</option>
      </select>
      {#if showCustomInput}
        <input 
          id="custom-length"
          type="number" 
          bind:value={customRoundLength}
          on:blur={handleCustomLengthBlur}
          min="1"
          max="64"
          step="1"
          disabled={gameActive}
          class="custom-input"
          placeholder="1-64"
          aria-label="Custom round length (1-64)"
        />
      {/if}
    </div>

    <div class="toggle-container">
      <span class="toggle-label">Timer:</span>
      <div class="toggle-switch">
        <input 
          type="checkbox" 
          bind:checked={showTimer} 
          on:change={toggleTimer}
          id="timer-toggle"
        />
        <label for="timer-toggle" class="toggle-slider">
          <span class="toggle-text">{showTimer ? 'On' : 'Off'}</span>
        </label>
      </div>
    </div>

    <div class="toggle-container">
      <span class="toggle-label">Dark Mode:</span>
      <div class="toggle-switch">
        <input 
          type="checkbox" 
          bind:checked={darkMode} 
          on:change={toggleDarkMode}
          id="dark-mode-toggle"
        />
        <label for="dark-mode-toggle" class="toggle-slider">
          <span class="toggle-text">{darkMode ? 'On' : 'Off'}</span>
        </label>
      </div>
    </div>


    <button 
      class="nav-button game-button {gameActive ? 'end' : 'start'}"
      on:click={gameActive ? endGame : startGame}
    >
      {gameActive ? 'End Game' : 'Start Game'}
    </button>
  </div>
</nav>

<style>
  .navbar {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #312e2b;
    color: white;
    padding: 1rem 2rem;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .nav-brand h2 {
    margin: 0;
    font-size: 1.5rem;
    font-weight: 600;
  }

  .nav-controls {
    display: flex;
    gap: 1rem;
  }

  .nav-button {
    background-color: #b58863;
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    border-radius: 4px;
    cursor: pointer;
    font-size: 0.9rem;
    font-weight: 500;
    transition: all 0.2s;
  }

  .nav-button:hover {
    background-color: #f0d9b5;
    color: #312e2b;
  }

  .nav-button.active {
    background-color: #f0d9b5;
    color: #312e2b;
  }

  .nav-button.active:hover {
    background-color: #e8d0a5;
    color: #312e2b;
  }

  .nav-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .round-selector {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .round-selector label {
    font-size: 0.9rem;
    color: white;
    margin-right: 0.5rem;
  }

  .sr-only {
    position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0, 0, 0, 0);
    white-space: nowrap;
    border: 0;
  }

  .nav-button.game-button {
    font-weight: bold;
    font-size: 1rem;
  }

  .nav-button.start {
    background-color: #00b894;
  }

  .nav-button.start:hover {
    background-color: #00a085;
  }

  .nav-button.end {
    background-color: #d63031;
  }

  .nav-button.end:hover {
    background-color: #c92a2a;
  }

  .toggle-container {
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .toggle-label {
    font-size: 0.9rem;
    color: white;
    margin-right: 0.5rem;
  }

  .toggle-switch {
    position: relative;
    display: inline-block;
    width: 60px;
    height: 30px;
  }

  .toggle-switch input {
    opacity: 0;
    width: 0;
    height: 0;
  }

  .toggle-slider {
    position: absolute;
    cursor: pointer;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background-color: #ccc;
    transition: 0.3s;
    border-radius: 30px;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .toggle-slider:before {
    position: absolute;
    content: "";
    height: 22px;
    width: 22px;
    left: 4px;
    bottom: 4px;
    background-color: white;
    transition: 0.3s;
    border-radius: 50%;
  }

  input:checked + .toggle-slider {
    background-color: #b58863;
  }

  input:checked + .toggle-slider:before {
    transform: translateX(30px);
  }

  .toggle-text {
    font-size: 0.7rem;
    color: white;
    font-weight: bold;
    z-index: 1;
  }

  select {
    background-color: #b58863;
    color: white;
    border: none;
    padding: 0.5rem;
    border-radius: 4px;
    font-size: 0.9rem;
    cursor: pointer;
    margin-right: 0.5rem;
  }

  select:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .custom-input {
    width: 70px;
    padding: 0.3rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 0.8rem;
  }

  .custom-input:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  @media (max-width: 768px) {
    .navbar {
      flex-direction: column;
      gap: 0.75rem;
      padding: 0.75rem;
      width: 100%;
      box-sizing: border-box;
    }
    
    .nav-brand h2 {
      font-size: 1.2rem;
      text-align: center;
    }
    
    .nav-controls {
      flex-wrap: wrap;
      justify-content: center;
      gap: 0.75rem;
      width: 100%;
    }
    
    .nav-button {
      min-width: 90px;
      white-space: nowrap;
      font-size: 0.8rem;
      padding: 0.4rem 0.8rem;
    }
    
    .toggle-container {
      min-width: 120px;
    }
    
    .toggle-label {
      font-size: 0.8rem;
    }
    
    .round-selector {
      flex-wrap: wrap;
      justify-content: center;
      gap: 0.25rem;
    }
    
    .round-selector label {
      font-size: 0.8rem;
    }
    
    select {
      font-size: 0.8rem;
      padding: 0.4rem;
    }
  }

  @media (max-width: 480px) {
    .navbar {
      padding: 0.5rem;
      gap: 0.5rem;
    }
    
    .nav-brand h2 {
      font-size: 1rem;
    }
    
    .nav-controls {
      gap: 0.5rem;
    }
    
    .nav-button {
      min-width: 80px;
      font-size: 0.75rem;
      padding: 0.3rem 0.6rem;
    }
    
    .toggle-container {
      min-width: 100px;
    }
    
    .toggle-switch {
      width: 50px;
      height: 25px;
    }
    
    .toggle-slider:before {
      height: 19px;
      width: 19px;
      left: 3px;
      bottom: 3px;
    }
    
    input:checked + .toggle-slider:before {
      transform: translateX(25px);
    }
    
    .toggle-text {
      font-size: 0.6rem;
    }
  }
</style>