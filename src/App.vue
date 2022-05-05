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

let scores = {
  X: 10,
  O: -10,
  tie: 0
};

let coordinates = {
  0: "0,0",
  1: "0,1",
  2: "0,2",
  3: "1,0",
  4: "1,1",
  5: "1,2",
  6: "2,0",
  7: "2,1",
  8: "2,2"
};

//All posible Winning scenarios on the current board
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

  let winner = null;

  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
      winner = squares[a];
      return winner;
    }
  }

  //Check if the whole board is full and no winner is declared
  let openSpots = 0;
  for (let j = 0; j < squares.length; j++) {
    if(squares[j] == '') openSpots++;
  }

  if (winner == null && openSpots <= 1) {
    return 'tie';
  } else {
    return winner;
  }
};

const winner = computed(() => CalculateWinner(board.value.flat())); //Calculate and declare the winner

const CheckIfHumanWinsNextRound = (boardVal, activePlayer) => {
  let coord1 = -1;
  let coord2 = -1;
  let winningMove;

  //Check if human could win with the next move
  console.log("Current Player is - " + activePlayer);
  console.log(boardVal);

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

  let possibleWinner = null;
  let tempPlayer = activePlayer === ai ? human : ai;

  for (let i = 0; i < lines.length; i++) {
    const [a, b, c] = lines[i];
    // if (squares[a] && squares[a] === squares[b] && squares[a] === squares[c]) {
    //   possibleWinner = squares[a];
    // }

    //Check if any 2 of the 3-set winning cells are filled with the current player
    if((boardVal[a] === human && boardVal[b] === human) || (boardVal[b] === human && boardVal[c] === human) || (boardVal[a] === human && boardVal[c] === human)){
      possibleWinner = human;

      if(possibleWinner != null) {
        if(boardVal[a] === '') {
          coord1 = coordinates[a].split(',')[0];
          coord2 = coordinates[a].split(',')[1];
        }
        else if(boardVal[b] === '') {
          coord1 = coordinates[b].split(',')[0];
          coord2 = coordinates[b].split(',')[1];
        }
        else if(boardVal[c] === '') {
          coord1 = coordinates[c].split(',')[0];
          coord2 = coordinates[c].split(',')[1];
        }

        break;
      }      
    }
    else continue;
  }

  winningMove = { coord1, coord2 };
  return winningMove;
};

const sleep = (ms) => {
  return new Promise(resolve => setTimeout(resolve, ms));
};

//Gameplay - Multi player
const MakeMove = async (x, y) => {
  if (winner.value) return; //Game Over - Winner is declared
  
  if (board.value[x][y] != "") return; //The position is already filled. Can't make a move here.
  
  try {
    if(showSinglePlayer.value) {
      
      //#region AI Gameplay     
      if (currentPlayer == human) {
        // If valid turn
        if (board.value[x][y] == '') {
          board.value[x][y] = human;
          currentPlayer = ai;

          //Track open spots
          let openSpotsCount = 0;

          for (let i = 0; i < 3; i++) {
            for (let j = 0; j < 3; j++) {
              if (board.value[i][j] == '') {
                openSpotsCount++;
              }
            }
          }

          //Is the middle cell empty? - Best Move
          if(currentPlayer === ai && openSpotsCount > 7 && board.value[1][1] === '') {
            board.value[1][1] = ai; 
            currentPlayer = human;
          }
          else 
            BestMove();
        }
      }

      //#endregion
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
const BestMove = async () => {
  /* Single Player gameplay (Player = AI) - MiniMax logic */
  let bestScore = -Infinity;
  let move;

  //Check if human(X) wins with the next move
  move = await CheckIfHumanWinsNextRound(board.value.flat(), currentPlayer);

  if(move != null && move.coord1 !== -1 && move.coord2 !== -1) {
    board.value[move.coord1][move.coord2] = ai;   //Make the best move found
    player.value = human; //Switch back to human
    currentPlayer = human;
  }
  else {
    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == '') {
          board.value[i][j] = ai;    //Set the player to the position - TEMPORARY!!
          let score = MiniMax(board, 0, false);
          board.value[i][j] = '';   //Undo the move immediately (Don't want to alter the gameboard yet)
          
          if(score > bestScore) {
            bestScore = score;
            move = {i, j};
          }
        }
      }
    }

    board.value[move.i][move.j] = ai;   //Make the best move found
    player.value = human; //Switch back to human
    currentPlayer = human;
  }
};

const MiniMax = (board, depth, isMaximizing) => {
  let result = winner.value;
  if(result !== null) {
    return scores[result];
  }
   
  if(isMaximizing) {
    //Maximizer play (X)
    let bestScore = -Infinity;

    for (let i = 0; i < 3; i++) {
      for (let j = 0; j < 3; j++) {
        //Is the spot available(empty)?
        if (board.value[i][j] == '') {
          board.value[i][j] = ai;
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
          board.value[i][j] = human;
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
  console.clear();
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
          {{ cell === 'X' ? "close" : cell === 'O' ? "circle" : '' }}
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

