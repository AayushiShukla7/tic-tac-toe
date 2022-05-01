<script setup>
import { ref, computed } from 'vue'

const player = ref('X');  //The player
const board = ref([
  ['', '', ''],
  ['', '', ''],
  ['', '', '']
]); //The game board

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
  return null;
}

const winner = computed(() => CalculateWinner(board.value.flat()));  //Calculate and declare the winner

//Gameplay
const MakeMove = (x, y) => {
  if (winner.value) return;   //Game Over - Winner is declared

  if(board.value[x][y] != "") return;   //The position is already filled. Can't make a move here.

  board.value[x][y] = player.value;   //Player makes a move

  player.value = player.value === 'X' ? 'O' : 'X';    //Switch between players
};

const ResetGame = () => {
  board.value = [
    ['', '', ''],
    ['', '', ''],
    ['', '', '']
  ];

  player.value = 'X'; //Set the player value back to 'X'
  showSinglePlayer.value = false;
};

//Single/Multi player selection
const showSinglePlayer = ref(false);

const toggle = () => {
  showSinglePlayer.value = !showSinglePlayer.value;
};

</script>

<template>
  <main class="pt-8 text-center dark:bg-gray-800 min-h-screen dark:text-white">
    <h1 class="mb-8 text-3xl font-bold uppercase">Tic Tac Toe</h1>
    
    <h3 class="text-xl mb-4">Player {{ player }}'s turn</h3>

    <div class="big mb-8">
      <button @click="toggle" :class="`px-4 py-2 bg-gray-400 rounded uppercase font-bold hover:bg-green-600 ${ showSinglePlayer ? 'bg-green-500' : ''}`">Single Player</button>
      <button @click="toggle" :class="`ml-1 px-4 py-2 bg-gray-400 rounded uppercase font-bold hover:bg-blue-600 ${ !showSinglePlayer ? 'bg-blue-500' : ''}`">Multi Player</button>
    </div>

    <div class="flex flex-col items-center mb-8">
      <div 
        v-for="(row, x) in board"
        :key="x"
        class="flex">

        <div
          v-for="(cell, y) in row"
          :key="y"
          @click="MakeMove(x, y)"
          :class="`border border-white w-20 h-20 hover:bg-gray-700 flex items-center justify-center material-icons-outlined text-4xl cursor-pointer ${ cell === 'X' ? 'text-red-500' : 'text-green-400' }`">
          {{ cell === 'X' ? 'close' : cell === 'O' ? 'circle' : '' }}
        </div>

      </div>
    </div>

    <h2 v-if="winner" class="text-6xl font-bold mb-8">Player '{{ winner }}' wins!</h2>

    <button @click="ResetGame" class="px-4 py-2 bg-pink-500 rounded uppercase font-bold hover:bg-pink-600 duration-300">Reset Game</button>

  </main>

</template>

<style>

</style>
