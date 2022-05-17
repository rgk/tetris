<script setup>
import { ref } from 'vue';

// 0 = empty, 1 = permanent, 2 = moving

const props = defineProps({
  speed: {
    type: Number,
    default: 1000
  },
  COLS: {
    type: Number,
    default: 10
  },
  ROWS: {
    type: Number,
    default: 20
  }
});

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

function emptyGrid(grid = []) {
  for (let i = 0; i < props.ROWS; i++) {
    grid.push([]);
    for (let j = 0; j < props.COLS; j++) {
      grid[i][j] = 0;
    }
  }

  return grid;
}

function mapShape(shape, x = 4, y = 0) {
  position.x = x;
  position.y = y;

  shape.forEach((row, i) => {
    row.forEach((column, j) => {
      grid.value[i + y][j + x] = (column) ? 2 : grid.value[i + y][j + x];
    });
  });

  return shape;
}

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
  let amount = shape.length;
  if (direction) {
    amount = shape[0].length;
  }

  const newShape = Array(amount).fill([]);

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
          newShape[i][j - 1] = row[j - 1];
      }
    }
  });

  return newShape;
}

// Create grid.
const grid = ref(emptyGrid());

let current = false;
let position = { x: 0, y: 0 };
let move = 0;
let turn = 0;

let score = ref(0);

// Need data to be mutable.
setInterval(() => {
  let tempGrid = [];
  let stop = false;

  // Hold data to make it immutable.
  let currentMove = move;
  let currentTurn = turn;

  // Reset moves every tick.
  if (move) move = 0;
  if (turn) turn = 0;

  if (!current) {
    current = shapes[Math.floor(Math.random() * shapes.length)];

    if (Math.random() >= 0.5) {
      current = flipX(current);
    }

    if (Math.random() >= 0.5) {
      current = rotate(current, (Math.random() >= 0.5) ? 1 : -1);
    }

    current = mapShape(current);
  }

  let shape = rotate(current, currentTurn);

  console.log(shape);
  console.log(current);

  Restart:
  for (let i = 0, row = position.y + 1, restart = false; i < shape.length; i++, row++) {
    // y bounds
    if (typeof grid.value[row] === "undefined" || stop) {
      if (currentTurn) {
        currentTurn = 0;
        shape = current;
        continue Restart;
      } else {
        stop = true;
        break;
      }
    }

    tempGrid[i] = [];

    for (let j = 0, column = position.x + currentMove; j < shape[0].length; j++, column++) {
      // x bounds
      if (typeof grid.value[row][column] === "undefined") {
        currentMove = 0;
        currentTurn = 0;
        shape = current;
        continue Restart;
        break;
      }

      if (shape[i][j] !== 2) {
        tempGrid[i][j] = grid.value[row][column];
      } else {
        if (grid.value[row][column] === 1) {
          if (currentMove || currentTurn) {
            currentMove = 0;
            currentTurn = 0;
            shape = current;
            continue Restart;
          } else {
            stop = true;
          }

          break;
        }

        tempGrid[i][j] = shape[i][j];
      }
    }
  }

  if (!stop) {
    // Clear the past moving values.
    for (let row = position.y; row < position.y + current.length; row++) {
      for (let column = position.x; column < position.x + current[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 0;
      }
    }

    current = shape;
    position.x += currentMove;
    position.y++;

    // Place moved values.
    for (let i = 0, row = position.y; row < position.y + current.length; i++, row++) {
      for (let j = 0, column = position.x; column < position.x + current[0].length; j++, column++) {
        if (tempGrid[i][j] === 2) grid.value[row][column] = 2;
      }
    }
  } else {
    for (let row = 0; row < grid.value.length; row++) {
      for (let column = 0, count = 0; column < grid.value[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 1;
        if (grid.value[row][column] === 1) count++;
        if (count !== props.COLS) continue;
        grid.value.splice(row, 1);
        grid.value.unshift(Array(10).fill(0));
        score.value++;

        break;
      }
    }

    current = false;
  }
}, props.speed);

</script>

<template>
  <div>Score: {{ score }}</div>
  <button id="tetris"
    @keyup.left="move = -1"
    @keyup.right="move = 1"
    @keyup.up="turn = 1"
    @keyup.down="turn = -1"
  >
    <div v-for="row in grid" style="height: 16px;">
      <div v-for="value in row" style="display: inline-block; height: 16px;">
        <div style="background-color: green; height: 16px; width: 16px; border: 1px solid #CCC;" v-if="value == 2"></div>
        <div style="background-color: blue; height: 16px; width: 16px; border: 1px solid #CCC;" v-if="value == 1"></div>
        <div style="background-color: white; height: 16px; width: 16px; border: 1px solid #CCC;" v-if="value == 0"></div>
      </div>
    </div>
  </button>
</template>
