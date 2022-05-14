<script setup>
import { ref } from 'vue';

// 0 = empty, 1 = permanent, 2 = moving

const COLS = 10;
const ROWS = 20;

const grid = ref([]);
// Create grid.
for (let i = 0; i < ROWS; i++) {
  grid.value.push([]);
  for (let j = 0; j < COLS; j++) {
    grid.value[i][j] = 0;
  }
}

let tempGrid = [];

let move = 0;

function mapShape(shape, x = 3, y = 0) {
  shape.forEach((row, i) => {
    row.forEach((column, j) => {
      grid.value[i + y][j + x] = (column) ? 2 : grid.value[i + y][j + x];
    });
  });
}

// Need data to be mutable.
setInterval(() => {
  let currentMove = move;
  if (move) move = 0;
  let start = 0;
  tempGrid = [];
  let stop = false;

  let newPiece = true;

  for (let i = grid.value.length - 1; i >= 0; i--) {
    tempGrid[i] = [];
    for (let j = 0; j < grid.value[i].length; j++) {
      if (
        (typeof grid.value[i + 1] === 'undefined' || grid.value[i + 1][j + currentMove] === 1) &&
        grid.value[i][j] === 2
      ) {
        i = start;
        stop = true;
        break;
      } else if (grid.value[i][j] === 2) {
        newPiece = false;
        if (stop) {
          tempGrid[i][j] = 1;
          continue;
        }
        if (start === grid.value.length || !start) start = i;
        if (j + currentMove < 0 || j + currentMove > 9 || i + 1 >= ROWS) {
          currentMove = move;
          i = start + 2;
          start = grid.value.length;
          break;
        }

        tempGrid[i + 1][j + currentMove] = 2;
      }

      tempGrid[i][j] = (grid.value[i][j] < 2) ? grid.value[i][j] : 0;
    }
  }

  grid.value = tempGrid;

  if (newPiece) {
    let piece = shapes[Math.floor(Math.random() * shapes.length)];

    if (Math.random() >= 0.5) {
      piece = flipX(piece);
    }

    mapShape(piece);
  }
}, 1000);

function flipX(shape) {
  const newShape = [];
  shape.forEach((row) => {
    const part = [];
    for (let i = row.length; i; i--) {
      part.push(row[i-1]);
    }
    newShape.push(part);
  });

  return newShape;
}

function rotate(shape, direction = 'r') {
  const newShape = [];

  for (let i = 0; i < shape[0].length; i++) {
    newShape.push([]);
  }

  shape.forEach((row, i) => { // 2
    for (let j = row.length; j; j--) {
      if (direction === 'r') {
        newShape[j - 1][shape.length - i - 1] = row[j - 1];
      } else {
        newShape[row.length - j][i] = row[j - 1];
      }
    }
  });

  return newShape;
}

const shapes = [
  [[ 2, 2, 2, 2 ]],
  [[ 2, 0, 0 ],
    [ 2, 2, 2 ]], // flipX
  [[ 2, 2 ],
    [ 2, 2 ]],
  [[ 0, 2, 2 ],
    [ 2, 2, 0 ]], // flipX
  [[ 2, 2, 2 ],
    [ 0, 2, 0 ]]
];
</script>

<template>
  <button id="tetris" @keyup.left="move = -1" @keyup.right="move = 1">
    <div v-for="row in grid">
      <div v-for="value in row" style="display: inline-block">
        {{ value }}
      </div>
    </div>
  </button>
</template>
