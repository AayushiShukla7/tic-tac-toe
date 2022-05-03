<script setup>
import { ref, computed } from "vue";

const player = ref("X"); //The player
const board = ref([
  ["", "", ""],
  ["", "", ""],
  ["", "", ""],
]); //The game board

//Single/Multi player selection
const showSinglePlayer = ref(false);

//All posible Winning scenarios on the current board
//Logic from - https://reactjs.org/tutorial/tutorial.html#declaring-a-winner
const CalculateWinner = (squares) => {
  const lines = [
    [0, 1, 2],
    [3, 4, 5],
    [6, 7, 8],
    [0, 3, 6],
    [1, 4, 7],
    [2, 5, 8],
    [0, 4, 8],
    [2, 4, 6],
  ];

  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      return squares[a];
    }
  }

  //Check if the whole board is full and no winner is declared
  let openSpots = 0;
  for (let j = 0; j < squares.length; j++) {
    if(squares[j] == '') openSpots++;
  }
  if(openSpots == 0)  return 'tie';

  return null;
};

const winner = computed(() => CalculateWinner(board.value.flat())); //Calculate and declare the winner

const sleep = (ms) => {
  return new Promise(resolve => setTimeout(resolve, ms));
}

//Gameplay - Multi player
const MakeMove = async (x, y) => {
  if (winner.value) return; //Game Over - Winner is declared
  
  if (board.value[x][y] != "") return; //The position is already filled. Can't make a move here.
  
  try {
    if(showSinglePlayer.value) {
      if(player.value === "X") {
        //Play as the user wants (irrespective of the gameplay mode)
        board.value[x][y] = player.value; //Player makes a move

        if (winner.value) return; //Game Over - Winner is declared

        player.value = player.value === "X" ? "O" : "X"; //Switch between players
        await sleep(1000);  //Wait for 1 second before makig the AI move

        console.info("Single player mode gameplay start.");
        await MakeAIMove(board);
        console.info("Single player mode gameplay end.");       
        
        player.value = player.value === "X" ? "O" : "X"; //Switch between players        
      }
    }
    else {
      //Play as the user wants (Multi player mode)
      board.value[x][y] = player.value; //Player makes a move

      player.value = player.value === "X" ? "O" : "X"; //Switch between players
    }
  }
  catch (err) {
    throw err;
  }
};

//Gameplay - Single player (Opponent --> AI)
const MakeAIMove = (board) => {
  /* Single Player gameplay - MiniMax logic */
  let bestScore = -Infinity;
  let bestMove;

  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      //Is the spot available(empty)?
      if (board.value[i][j] == "") {
        board.value[i][j] = player.value;    //Set the player to the position - TEMPORARY!!
        let score = MiniMax(board, 0, true);
        board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
        
        if(score > bestScore) {
          bestScore = score;
          bestMove = {i, j};
        }
      }
    }
  }

  //if(bestMove !== null && bestMove !== undefined) {
    board.value[bestMove.i][bestMove.j] = player.value;   //Make the best move found
    player.value = player.value === "X" ? "O" : "X"; //Switch between players
  //}
};

const Scores = ref({
  X: 1,
  O: -1,
  tie: 0
});

const MiniMax = (board, depth, isMinimizing) => {
  let result = winner.value;
  if(result !== null) {
    return Scores[result];
  }

  if(isMinimizing) {
    let bestScore = -Infinity;

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == "") {
          board.value[i][j] = player.value;
          let score = MiniMax(board, depth + 1, false);
          board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
          bestScore = Math.min(score, bestScore);
        }
      }
    }

    return bestScore;
  }
  else {
    //Maximizer play (X)
    let bestScore = Infinity;

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == "") {
          board.value[i][j] = player.value;
          let score = MiniMax(board, depth + 1, true);
          board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
          bestScore = Math.max(score, bestScore);
        }
      }
    }

    return bestScore;
  }
}

const ResetGame = () => {
  board.value = [
    ["", "", ""],
    ["", "", ""],
    ["", "", ""],
  ];

  player.value = "X"; //Set the player value back to 'X'
  showSinglePlayer.value = false;   //Play multi-player by default
};

const toggle = () => {
  showSinglePlayer.value = !showSinglePlayer.value;
  
  //Reset game board
  board.value = [
    ["", "", ""],
    ["", "", ""],
    ["", "", ""],
  ];

  player.value = "X"; //Set the player value back to 'X'
};

</script>

<template>
  <main class="pt-8 text-center dark:bg-gray-800 min-h-screen dark:text-white">
    <h1 class="mb-8 text-3xl font-bold uppercase">Tic Tac Toe</h1>

    <h3 class="text-xl mb-4">Player {{ player }}'s turn</h3>

    <div class="big mb-8">
      <button
        @click="toggle"
        :class="`px-4 py-2 bg-gray-400 rounded uppercase font-bold hover:bg-green-600 ${
          showSinglePlayer ? 'bg-green-500' : ''
        }`"
        title="AI is player 'O'"
      >
        Single Player
      </button>
      <button
        @click="toggle"
        :class="`ml-1 px-4 py-2 bg-gray-400 rounded uppercase font-bold hover:bg-blue-600 ${
          !showSinglePlayer ? 'bg-blue-500' : ''
        }`"
      >
        Multi Player
      </button>
    </div>

    <div class="flex flex-col items-center mb-8">
      <div v-for="(row, x) in board" :key="x" class="flex">
        <div
          v-for="(cell, y) in row"
          :key="y"
          @click="MakeMove(x, y)"
          :class="`border border-white w-20 h-20 hover:bg-gray-700 flex items-center justify-center material-icons-outlined text-4xl cursor-pointer ${
            cell === 'X' ? 'text-red-500' : 'text-green-400'
          }`"
        >
          {{ cell === "X" ? "close" : cell === "O" ? "circle" : "" }}
        </div>
      </div>
    </div>

    <h2 v-if="winner" class="text-6xl font-bold mb-8">
      Player '{{ winner }}' wins!
    </h2>
    <h2 v-else-if="winner === 'tie'" class="text-6xl font-bold mb-8">
      It's a '{{ winner }}'
    </h2>

    <button
      @click="ResetGame"
      class="
        px-4
        py-2
        bg-pink-500
        rounded
        uppercase
        font-bold
        hover:bg-pink-600
        duration-300
      "
    >
      Reset Game
    </button>
  </main>
</template>

<style>
</style>
