<script lang="ts">
  export let showPieces = true;
  export let flipped = false;
  export let targetSquare = '';
  export let gameActive = false;
  export let correctSquares: string[] = [];
  export let incorrectSquares: string[] = [];
  export let darkMode = false;
  
  let selectedSquare = '';

  const files = ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'];
  const ranks = ['8', '7', '6', '5', '4', '3', '2', '1'];
  
  $: displayFiles = flipped ? [...files].reverse() : files;
  $: displayRanks = flipped ? [...ranks].reverse() : ranks;

  const pieces: { [key: string]: string } = {
    '♜': '♜', '♞': '♞', '♝': '♝', '♛': '♛', '♚': '♚', '♟': '♟',
    '♖': '♖', '♘': '♘', '♗': '♗', '♕': '♕', '♔': '♔', '♙': '♙'
  };

  const initialBoard: { [key: string]: string } = {
    'a8': '♜', 'b8': '♞', 'c8': '♝', 'd8': '♛', 'e8': '♚', 'f8': '♝', 'g8': '♞', 'h8': '♜',
    'a7': '♟', 'b7': '♟', 'c7': '♟', 'd7': '♟', 'e7': '♟', 'f7': '♟', 'g7': '♟', 'h7': '♟',
    'a2': '♙', 'b2': '♙', 'c2': '♙', 'd2': '♙', 'e2': '♙', 'f2': '♙', 'g2': '♙', 'h2': '♙',
    'a1': '♖', 'b1': '♘', 'c1': '♗', 'd1': '♕', 'e1': '♔', 'f1': '♗', 'g1': '♘', 'h1': '♖'
  };

  import { createEventDispatcher } from 'svelte';
  const dispatch = createEventDispatcher();

  function handleSquareClick(file: string, rank: string) {
    const clickedSquare = file + rank;
    selectedSquare = clickedSquare;
    
    if (gameActive && targetSquare) {
      dispatch('squareClicked', { square: clickedSquare, isCorrect: clickedSquare === targetSquare });
    }
  }

  function isLightSquare(fileIndex: number, rankIndex: number): boolean {
    return (fileIndex + rankIndex) % 2 === 0;
  }

  function getPieceOnSquare(file: string, rank: string): string {
    return initialBoard[file + rank] || '';
  }

  function getSquareClass(file: string, rank: string, fileIndex: number, rankIndex: number): string {
    const square = file + rank;
    let classes = `square ${isLightSquare(fileIndex, rankIndex) ? 'light' : 'dark'}`;
    
    if (gameActive && square === targetSquare) {
      classes += ' target';
    }
    if (correctSquares.includes(square)) {
      classes += ' correct';
    }
    if (incorrectSquares.includes(square)) {
      classes += ' incorrect';
    }
    
    return classes;
  }
</script>

<div class="chess-board-container {darkMode ? 'dark-mode' : ''}">
  <div class="chess-board">
    {#each displayRanks as rank, rankIndex}
      {#each displayFiles as file, fileIndex}
        <button
          class={getSquareClass(file, rank, fileIndex, rankIndex)}
          on:click={() => handleSquareClick(file, rank)}
          aria-label="Square {file}{rank}"
        >
          {#if showPieces && getPieceOnSquare(file, rank)}
            <span class="piece">{getPieceOnSquare(file, rank)}</span>
          {/if}
        </button>
      {/each}
    {/each}
  </div>
  
  {#if gameActive && targetSquare}
    <p class="coordinates game-info">Find square: <strong class="target">{targetSquare}</strong></p>
  {:else if selectedSquare}
    <p class="coordinates">Selected square: <strong>{selectedSquare}</strong></p>
  {:else}
    <p class="coordinates">Click a square to see its coordinates</p>
  {/if}
</div>

<style>
  .chess-board-container {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
    padding: 20px;
  }

  .chess-board {
    display: grid;
    grid-template-columns: repeat(8, 60px);
    grid-template-rows: repeat(8, 60px);
    border: 3px solid #312e2b;
    background-color: #312e2b;
  }

  .square {
    width: 60px;
    height: 60px;
    border: none;
    cursor: pointer;
    transition: opacity 0.2s;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 0;
  }

  @media (max-width: 768px) {
    .chess-board {
      grid-template-columns: repeat(8, 40px);
      grid-template-rows: repeat(8, 40px);
      border: 2px solid #312e2b;
    }

    .square {
      width: 40px;
      height: 40px;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0;
    }

    .piece {
      font-size: 28px;
      font-family: 'Arial Unicode MS', 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
    }

    .chess-board-container {
      padding: 10px;
      gap: 15px;
    }

    .coordinates {
      font-size: 16px;
    }

    .game-info {
      font-size: 18px;
    }

    .target {
      font-size: 20px;
    }
  }

  @media (max-width: 480px) {
    .chess-board {
      grid-template-columns: repeat(8, 35px);
      grid-template-rows: repeat(8, 35px);
    }

    .square {
      width: 35px;
      height: 35px;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 0;
    }

    .piece {
      font-size: 24px;
      font-family: 'Arial Unicode MS', 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol', serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
    }

    .coordinates {
      font-size: 14px;
    }

    .game-info {
      font-size: 16px;
    }

    .target {
      font-size: 18px;
    }
  }

  .square:hover {
    opacity: 0.8;
  }

  .light {
    background-color: #f0d9b5;
  }

  .dark {
    background-color: #b58863;
  }

  .piece {
    font-size: 40px;
    line-height: 1;
    user-select: none;
  }

  .coordinates {
    font-family: Arial, sans-serif;
    font-size: 18px;
    color: #312e2b;
    margin: 0;
    transition: color 0.3s ease;
  }

  .chess-board-container.dark-mode .coordinates {
    color: #e0e0e0;
  }

  .game-info {
    font-size: 20px;
    font-weight: bold;
  }

  .target {
    color: #d63031;
    font-size: 24px;
  }

  .square.target {
    box-shadow: 0 0 0 4px #d63031;
    animation: pulse 1s infinite;
  }

  .square.correct {
    box-shadow: 0 0 0 4px #00b894;
  }

  .square.incorrect {
    box-shadow: 0 0 0 4px #e17055;
  }

  @keyframes pulse {
    0% { box-shadow: 0 0 0 4px #d63031; }
    50% { box-shadow: 0 0 0 8px rgba(214, 48, 49, 0.5); }
    100% { box-shadow: 0 0 0 4px #d63031; }
  }
</style>