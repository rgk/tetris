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

let current = false;
let position = { x: 0, y: 0 };
let move = 0;
let turn = 0;
let firstTurn = true;

function mapShape(shape, x = 3, y = 0) {
  current = shape;
  position.x = x;
  position.y = y;
  shape.forEach((row, i) => {
    row.forEach((column, j) => {
      grid.value[i + y][j + x] = (column) ? 2 : grid.value[i + y][j + x];
    });
  });
}

// Need data to be mutable.
setInterval(() => {
  tempGrid = [];
  let stop = false;

  if (turn) current = rotate(current, turn);

  for (let i = 0, row = position.y; row <= position.y + current.length; i++, row++) {
    if (typeof grid.value[row] === 'undefined') {
      stop = true;
      break;
    }

    tempGrid[i] = [];

    for (let j = 0, column = position.x + move; column < position.x + current[0].length + move; j++, column++) {
      if (typeof grid.value[row][column] === 'undefined') {
        row = position.y - 1, i = -1, move = 0;
        break;
      }

      tempGrid[i][j] = (grid.value[row][column] < 2) ? grid.value[row][column] : 0;
    }
  }

  if (!stop) {
    let spot = (move < 1) ? 0 : 1;
    for (let i = 0; i < current.length; i++) {
      for (let j = 0; j < current[0].length; j++) {
        if (current[i][j] !== 2) continue;

        if (tempGrid[i + 1][j + spot] === 1) {
          stop = true;
          i = tempGrid.length;
          break;
        }

        tempGrid[i + 1][j + spot] = 2;
      }
    }
  }

  if (!stop) {
    for (let row = 0; row < grid.value.length; row++) {
      for (let column = 0; column < grid.value[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 0;
      }
    }

    for (let i = 0, row = position.y; row <= position.y + current.length; i++, row++) {
      for (let j = 0, column = position.x; column < position.x + current[0].length; j++, column++) {
        if (tempGrid[i][j] === 2) grid.value[row][column] = 2;
      }
    }

    position.y++;
    position.x += move;
  } else {
    for (let row = 0; row < grid.value.length; row++) {
      for (let column = 0, count = 0; column < grid.value[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 1;
        if (grid.value[row][column] === 1) count++;
        if (count === COLS) {
          grid.value = grid.value.splice(row, 1);
          grid.value.unshift(Array(10).fill(0));
        }
      }
    }
  }

  if (move) move = 0;
  if (turn) turn = 0;

  if (stop || firstTurn) {
    if (firstTurn) firstTurn = false;
    let piece = shapes[Math.floor(Math.random() * shapes.length)];

    if (Math.random() >= 0.5) {
      piece = flipX(piece);
    }

    if (Math.random() >= 0.5) {
      piece = rotate(piece, (Math.random() >= 0.5) ? 1 : -1);
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

function rotate(shape, direction = 0) {
  const newShape = [];

  for (let i = 0; i < shape[0].length; i++) {
    newShape.push([]);
  }

  shape.forEach((row, i) => {
    for (let j = row.length; j; j--) {
      switch (direction) {
        case -1:
          newShape[j - 1][shape.length - i - 1] = row[j - 1];
          break;
        case 1:
          newShape[row.length - j][i] = row[j - 1];
          break;
        default:
          newShape = shape;
          j = 1;
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
  <button id="tetris"
    @keyup.left="move = -1"
    @keyup.right="move = 1"
    @keyup.up="turn = 1"
    @keyup.down="turn = -1"
  >
    <div v-for="row in grid" style="height: 16px;">
      <div v-for="value in row" style="display: inline-block; height: 16px;">
        <div style="background-color: green; height: 16px; width: 16px;" v-if="value == 2"></div>
        <div style="background-color: blue; height: 16px; width: 16px;" v-if="value == 1"></div>
        <div style="background-color: white; height: 16px; width: 16px;" v-if="value == 0"></div>
      </div>
    </div>
  </button>
</template>
