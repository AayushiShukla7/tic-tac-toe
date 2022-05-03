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
let ai = 'O';
let human = 'X';
let currentPlayer = human;

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

//Test code for single player - START

const bestMove = () => {
  // AI to make its turn
  let bestScore = -Infinity;
  let move;
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      // Is the spot available?
      if (board.value[i][j] == '') {
        board.value[i][j] = ai;
        let score = minimax(board, 0, true);
        board.value[i][j] = '';
        if (score > bestScore) {
          bestScore = score;
          move = { i, j };
        }
      }
    }
  }
  board.value[move.i][move.j] = ai;
  currentPlayer = human;
}

let scores = {
  X: 10,
  O: -10,
  tie: 0
};

const equals3 = (a, b, c) => {
  return a == b && b == c && a != '';
}

const checkWinner = () => {
  let winner = null;

  // horizontal
  for (let i = 0; i < 3; i++) {
    if (equals3(board.value[i][0], board.value[i][1], board.value[i][2])) {
      winner = board.value[i][0];
    }
  }

  // Vertical
  for (let i = 0; i < 3; i++) {
    if (equals3(board.value[0][i], board.value[1][i], board.value[2][i])) {
      winner = board.value[0][i];
    }
  }

  // Diagonal
  if (equals3(board.value[0][0], board.value[1][1], board.value[2][2])) {
    winner = board.value[0][0];
  }
  if (equals3(board.value[2][0], board.value[1][1], board.value[0][2])) {
    winner = board.value[2][0];
  }

  let openSpots = 0;
  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      if (board.value[i][j] == '') {
        openSpots++;
      }
    }
  }

  if (winner == null && openSpots == 0) {
    return 'tie';
  } else {
    return winner;
  }
}

const minimax = (board, depth, isMaximizing) => {
  let result = checkWinner();
  if (result !== null) {
    return scores[result];
  }

  if (isMaximizing) {
    let bestScore = -Infinity;
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        // Is the spot available?
        if (board.value[i][j] == '') {
          board.value[i][j] = ai;
          let score = minimax(board, depth + 1, false);
          board.value[i][j] = '';
          bestScore = Math.max(score, bestScore);
        }
      }
    }
    return bestScore;
  } else {
    let bestScore = Infinity;
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        // Is the spot available?
        if (board.value[i][j] == '') {
          board.value[i][j] = human;
          let score = minimax(board, depth + 1, true);
          board.value[i][j] = '';
          bestScore = Math.min(score, bestScore);
        }
      }
    }
    return bestScore;
  }
}

//Test code for single player - END

//Gameplay - Multi player
const MakeMove = async (x, y) => {
  if (winner.value) return; //Game Over - Winner is declared
  
  if (board.value[x][y] != "") return; //The position is already filled. Can't make a move here.
  
  try {
    if(showSinglePlayer.value) {
      //Approach #
      // if(player.value == "X") {
      //   //Play as the user wants (irrespective of the gameplay mode)
      //   board.value[x][y] = player.value; //Player makes a move

      //   if (winner.value) return; //Game Over - Winner is declared

      //   player.value = 'O'; //Switch to AI
      //   await sleep(1000);  //Wait for 1 second before making the AI move

      //   console.info("Single player mode gameplay start.");
      //   BestMove();
      //   console.info("Single player mode gameplay end."); 
      // }

      //Approach #2    
      if (currentPlayer == human) {
        // If valid turn
        if (board.value[x][y] == '') {
          board.value[x][y] = human;
          currentPlayer = ai;
          bestMove();
        }
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
// const BestMove = () => {
//   /* Single Player gameplay - MiniMax logic */
//   let bestScore = Infinity;
//   let move;

//   for (let i = 0; i < 3; i++) {
//     for (let j = 0; j < 3; j++) {
//       //Is the spot available(empty)?
//       if (board.value[i][j] == '') {
//         board.value[i][j] = 'O';    //Set the player to the position - TEMPORARY!!
//         let score = MiniMax(board, 0, true);
//         board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
        
//         if(score > bestScore) {
//           bestScore = score;
//           move = {i, j};
//         }
//       }
//     }
//   }

//   board.value[move.i][move.j] = 'O';   //Make the best move found
//   player.value = 'X'; //Switch back to human
// };
const BestMove = () => {
  /* Single Player gameplay - MiniMax logic */
  let bestScore = -Infinity;
  let move;

  for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
      //Is the spot available(empty)?
      if (board.value[i][j] == '') {
        board.value[i][j] = 'O';    //Set the player to the position - TEMPORARY!!
        let score = MiniMax(board, 0, false);
        board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
        
        if(score > bestScore) {
          bestScore = score;
          move = {i, j};
        }
      }
    }
  }

  board.value[move.i][move.j] = player.value;   //Make the best move found
  player.value = 'X'; //Switch back to human
};

const Scores = ref({
  X: 10,
  O: -10,
  tie: 0
});

const MiniMax = (board, depth, isMaximizing) => {
  let result = winner.value;
  if(result !== null) {
    return Scores[result];
  }
   
  if(isMaximizing) {
    //Maximizer play (X)
    let bestScore = -Infinity;

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == '') {
          board.value[i][j] = 'O';
          let score = MiniMax(board, depth + 1, false);
          board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
          bestScore = Math.max(score, bestScore);
        }
      }
    }

    return bestScore;
  }
  else {
    let bestScore = Infinity;

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == '') {
          board.value[i][j] = 'X';
          let score = MiniMax(board, depth + 1, true);
          board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
          bestScore = Math.min(score, bestScore);
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

    <h2 v-if="winner && winner !== 'tie'" class="text-6xl font-bold mb-8">
      Player '{{ winner }}' wins!
    </h2>
    <h2 v-if="winner === 'tie'" class="text-6xl font-bold mb-8">
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
