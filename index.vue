<script setup>
import { ref } from 'vue';

// 0 = empty, 1 = permanent, 2 = moving

const COLS = 10;
const ROWS = 20;

const grid = [];

for (let i = 0; i < ROWS; i++) {
  grid.push([]);
  for (let j = 0; j < COLS; j++) {
    grid[i].push(0);
  }
}

function flipX(shape, flip) {
  const newShape = [];
  shape.forEach((row, i) => {
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
  [[1, 1, 1, 1]],
  [[ 1, 0, 0 ],
    [ 1, 1, 1 ]],
  [[ 1, 1 ],
    [ 1, 1 ]],
  [[ 0, 1, 1 ],
    [ 1, 1, 0 ]],
  [[ 1, 1, 1 ],
    [ 0, 1, 0 ]]
];

</script>

<template>
  <div id="tetris">
  </div>
</template>
