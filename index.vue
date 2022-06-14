<script setup>
import { ref, onMounted } from 'vue';

// Grid Values:
// 0 = empty, 1 = permanent, 2 = moving

const props = defineProps({
  speed: {
    type: Number,
    default: 500
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

const startShape = [
  [ 2, 2, 2, 2 ],
  [ 2, 0, 0, 0 ],
  [ 2, 2, 2, 2 ],
  [ 0, 0, 0, 2 ],
  [ 2, 2, 2, 2 ],
  [ 0, 0, 0, 0 ],
  [ 2, 2, 2, 2 ],
  [ 0, 2, 0, 0 ],
  [ 0, 2, 0, 0 ],
  [ 0, 2, 0, 0 ],
  [ 0, 0, 0, 0 ],
  [ 2, 2, 2, 0 ],
  [ 2, 0, 0, 2 ],
  [ 2, 2, 2, 0 ],
  [ 2, 0, 0, 2 ],
  [ 0, 0, 0, 0 ],
  [ 2, 2, 2, 2 ],
  [ 0, 2, 0, 0 ],
  [ 0, 2, 0, 0 ],
  [ 0, 2, 0, 0 ]
]

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

function mapShape(shape, x = 3, y = 0) {
  position.x = x;
  position.y = y;

  shape.forEach((row, i) => {
    row.forEach((column, j) => {
      grid.value[i + y][j + x] = (column) ? 2 : grid.value[i + y][j + x];
    });
  });

  return shape;
}

function emptyGrid(grid = []) {
  for (let i = 0; i < props.ROWS; i++) {
    grid.push([]);
    for (let j = 0; j < props.COLS; j++) {
      grid[i][j] = 0;
    }
  }

  return grid;
}

function transform(shape, direction = 0, flipX = false) {
  let newShape = [];

  if (flipX) {
    shape.forEach((row) => {
      const part = [];
      for (let i = row.length; i; i--) {
        part.push(row[i - 1]);
      }
      newShape.push(part);
    });
  } else {
    // Do not use Array.fill for arrays.
    newShape = Array.from(
      { length: (direction) ? shape[0].length : shape.length },
      () => []
    );

    for (let i = 0; i < shape.length; i++) {
      for (let j = 0; j < shape[i].length; j++) {
        if (direction) {
          newShape[j][i] = (direction === 1) ? shape[i][shape[i].length - j - 1] : shape[shape.length - i - 1][j];
        } else {
          newShape[i][j] = shape[i][j];
        }
      }
    }
  }

  return newShape;
}

// Create grid.
const grid = ref(emptyGrid());

// Need DOM.
onMounted(() => {
  mapShape(startShape);
});

let loop = false;

let current = false;
let position = { x: 0, y: 0 };
let move = 0;
let turn = 0;
let fast = 0;

const score = ref([0]);

function logic() {
  let tempGrid = [];
  let stop = null;

  // Hold data to make it immutable.
  const currentMove = [ move, turn ];

  // Reset moves every tick.
  if (move) move = 0;
  if (turn) turn = 0;

  if (!current) {
    current = shapes[Math.floor(Math.random() * shapes.length)];

    if (Math.random() >= 0.5) {
      current = transform(current, (Math.random() >= 0.5) ? 1 : -1, (Math.random() <= 0.5) ? true : false);
    }

    mapShape(current);
  }

  let shape = transform(current, currentMove[0]);

  for (let i = 0, row = position.y + 1, temp = [], restart = false; i < shape.length; i++, row++) {
    // y bounds
    if (typeof grid.value[row] === "undefined" || stop) {
      if (currentMove[0] || currentMove[1]) {
        restart = true;
      } else {
        stop = true;
        break;
      }
    }

    temp[i] = [];

    for (let j = 0, column = position.x + currentMove[0]; j < shape[0].length; j++, column++) {
      // x bounds
      if (restart || typeof grid.value[row][column] === "undefined") {
        restart = true;
        break;
      }

      if (shape[i][j] !== 2) {
        temp[i][j] = (grid.value[row][column] === 2) ? 0 : grid.value[row][column];
      } else {
        if (grid.value[row][column] === 1) {
          if (currentMove[0] || currentMove[1]) {
            restart = true;
          } else {
            stop = true;
          }
          break;
        }

        temp[i][j] = shape[i][j];
      }
    }

    // TODO: Cleaner way to restart this logic.
    if (restart) {
      restart = false;
      currentMove[0] = 0;
      currentMove[1] = 0;
      shape = transform(current);
      i = -1;
      temp = [];
      row = position.y;
    } else {
      tempGrid = temp;
    }
  }

  if (!stop) {
    // Clear the past moving values.
    for (let row = position.y; row < position.y + current.length; row++) {
      for (let column = position.x; column < position.x + current[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 0;
      }
    }

    current = transform(shape);
    position.x += currentMove[0];
    position.y++;

    // Place moved values.
    for (let i = 0, row = position.y; row < position.y + current.length; i++, row++) {
      for (let j = 0, column = position.x; column < position.x + current[0].length; j++, column++) {
        if (tempGrid[i][j] === 2) grid.value[row][column] = 2;
      }
    }
  } else {
    if (!position.y) {
      grid.value = emptyGrid();
      score.value.push(score.value[0]);
      score.value[0] = 0;
      return;
    }

    for (let row = 0; row < grid.value.length; row++) {
      for (let column = 0, count = 0; column < grid.value[0].length; column++) {
        if (grid.value[row][column] === 2) grid.value[row][column] = 1;
        if (grid.value[row][column] === 1) count++;

        if (count !== props.COLS) continue;
        grid.value.splice(row, 1);
        grid.value.unshift(Array(props.COLS).fill(0));
        score.value[0]++;

        break;
      }
    }

    // Trigger new piece drop.
    current = false;
  }

  loop = null;
  start();
}

function start(first = false) {
  if (first) {
    grid.value = emptyGrid();
    current = false;
  }

  if (!loop) return loop = setTimeout(logic, props.speed / ( 1 + fast ));
  return false;
}

</script>

<template>
  <button id="tetris"
    @keyup.left="move = -1"
    @keyup.right="move = 1"
    @keyup.up="turn = 1"
    @keydown.down="fast = 1"
    @keyup.down="fast = 0"
    @click="start(true)"
  >
    <span>Best: {{ Math.max(..score) }} | Score: {{ score[0] }}</span>
    <div v-for="row in grid" style="height: 16px; border: 1px solid #111;">
      <div v-for="value in row" style="display: inline-block; height: 16px; border: 1px solid #CCC;">
        <div style="background-color: green; height: 13px; width: 13px;" v-if="value == 2"></div>
        <div style="background-color: blue; height: 13px; width: 13px;" v-if="value == 1"></div>
        <div style="background-color: white; height: 13px; width: 13px;" v-if="value == 0"></div>
      </div>
    </div>
    <div>
      <button class="display: inline-block" @click="move = -1">Left</button>
      <button class="display: inline-block" @click="move = 1">Right</button>
      <button class="display: inline-block" @click="turn = 1">Turn</button>
    </div>
  </button>
</template>
