<script setup>
import { ref } from 'vue';

// 0 = empty, 1 = permanent, 2 = moving, 3 = moved

const COLS = 10;
const ROWS = 20;

const grid = ref([]);

let somekey = 0;

function mapShape(shape, x = 4, y = 3) {
  shape.forEach((row, i) => {
    row.forEach((column, j) => {
      grid.value[i + y][j + x] = (column) ? 2 : grid.value[i + y][j + x];
    });
  });
}

setInterval(() => {
  for (let i = 0; i < grid.value.length; i++) {
    for (let j = 0; j < grid.value[i].length; j++) {
      if (typeof grid.value[i + 1] === 'undefined') {
        grid.value[i][j] = 1;
      } else if (grid.value[i][j] === 2) {
        grid.value[i][j] = 0;
        grid.value[i + 1][j] = 3;
      } else if (grid.value[i][j] === 3) {
        grid.value[i][j] = 2;
      }
    }
  }
}, 1000);

for (let i = 0; i < ROWS; i++) {
  grid.value.push([]);
  for (let j = 0; j < COLS; j++) {
    grid.value[i][j] = 0;
  }
}

function flipX(shape, flip) {
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
    [ 2, 2, 2 ]],
  [[ 2, 2 ],
    [ 2, 2 ]],
  [[ 0, 2, 2 ],
    [ 2, 2, 0 ]],
  [[ 2, 2, 2 ],
    [ 0, 2, 0 ]]
];

mapShape(shapes[0]);

</script>

<template>
  <div id="tetris">
    <div v-for="row in grid">
      <div v-for="value in row" style="display: inline-block">
        {{ value }}
      </div>
    </div>
  </div>
</template>
