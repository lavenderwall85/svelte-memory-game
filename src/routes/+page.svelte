<script>
    import { writable } from 'svelte/store';
  
    // Lista de símbolos para las cartas
    const cardValues = ["🍎", "🍌", "🍇", "🍒", "🍋", "🍉", "🥝", "🍍"];
  
    // Crear variables para manejar el juego
    let cards = writable([]);          // Lista de cartas
    let flippedCards = [];             // Cartas volteadas
    let matchedCards = writable([]);   // Cartas emparejadas
    let attempts = writable(0);        // Intentos
  
    // Iniciar el juego
    function initializeGame() {
      const initialCards = [...cardValues, ...cardValues]
        .map(value => ({
          value,
          id: Math.random(),    
          flipped: false          
        }))
        .sort(() => Math.random() - 0.5);  
  
      // Establecer el estado inicial
      cards.set(initialCards);
      flippedCards = [];
      matchedCards.set([]);
      attempts.set(0);
    }
  
    // Voltear una carta
    function flipCard(card) {
      cards.update(currentCards => {
        // Evitar voltear si ya está volteada o si hay dos cartas volteadas
        if (card.flipped || flippedCards.length === 2) return currentCards;
  
        // Voltear la carta seleccionada
        const updatedCards = currentCards.map(c => 
          c.id === card.id ? { ...c, flipped: true } : c
        );
  
        // Agregar la carta volteada a la lista
        flippedCards = [...flippedCards, card];
  
        // Si hay dos cartas volteadas, verificar si son iguales
        if (flippedCards.length === 2) {
          attempts.update(n => n + 1); // 
  
          // Si las cartas coinciden, mantenerlas volteadas
          if (flippedCards[0].value === flippedCards[1].value) {
            matchedCards.update(matched => [...matched, ...flippedCards]);
            flippedCards = []; // Reiniciar las cartas volteadas
          } else {
            // Si no coinciden, volverlas a cubrir después de un segundo
            setTimeout(() => {
              cards.update(allCards => 
                allCards.map(c => flippedCards.find(fc => fc.id === c.id) 
                  ? { ...c, flipped: false } 
                  : c
                )
              );
              flippedCards = [];
            }, 1000);
          }
        }
        return updatedCards;
      });
    }
  
    // Iniciar el juego al cargar
    initializeGame();
  </script>
  
  <style>
    .game-board {
      display: grid;
      grid-template-columns: repeat(4, 1fr); /* 4 columnas */
      gap: 10px;
      max-width: 400px;
      margin: 0 auto;
    }
    
    .card {
      width: 80px;
      height: 80px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 2em;
      background-color: #f0f0f0;
      border: 2px solid #ccc;
      border-radius: 8px;
      cursor: pointer;
      transition: transform 0.3s;
    }
    
    .flipped {
      background-color: #fff;
    }
  
    .stats {
      text-align: center;
      margin-top: 20px;
      font-size: 1.2em;
    }
  </style>
  
  <div class="stats">
    <p>Intentos: {$attempts}</p>
    <button on:click={initializeGame}>Reiniciar Juego</button>
  </div>
  
  <div class="game-board">
    {#each $cards as card}
      <div
        class="card {card.flipped || $matchedCards.includes(card) ? 'flipped' : ''}"
        on:click={() => flipCard(card)}
      >
        {#if card.flipped || $matchedCards.includes(card)}
          {card.value}
        {/if}
      </div>
    {/each}
  </div>
