---
<!-- title: Ramda-powered Life -->
separator: \n---\n
verticalSeparator: \n----\n
theme: moon
center: false
revealOptions:
    transition: 'fade'
---

# Ramda-powered Life

![](https://upload.wikimedia.org/wikipedia/en/d/d0/Game_of_life_animated_glider_2.gif)

### _With code samples_

---

## Summary

We will review:

* Rules of Conway's Game of Life <!-- .element: class="fragment" data-fragment-index="1" -->
* Origins of Conway's Game of Life <!-- .element: class="fragment" data-fragment-index="2" -->
* An implementations of Conway's Game of Life using Ramda-powered functional JavaScript <!-- .element: class="fragment" data-fragment-index="3" -->

---

## Say?

> The "game" is a zero-player game, meaning that its evolution is determined by its initial state, requiring no further input. One interacts with the Game of Life by creating an initial configuration and observing how it evolves, or, for advanced "players", by creating patterns with particular properties.
>
> —Wikipedia

---

# Start with

---

![](http://www.math.cornell.edu/~lipa/mec/banner.png)

---

# Rules

---

## Rule one

Any live cell with fewer than two live neighbors dies, as if caused by underpopulation.

---

## Rule two

Any live cell with two or three live neighbors lives on to the next generation.

---

## Rule three

Any live cell with more than three live neighbors dies, as if by overpopulation.

---

## Rule four

Any dead cell with exactly three live neighbors becomes a live cell, as if by reproduction.

---

## How it works

* The initial input constitutes the seed of the system.
* The first generation is created by applying our rules simultaneously to every cell furnished by our seed.
* Births and deaths occur simultaneously, and the discrete moment at which this happens is often referred to as a _tick_ or _step._
* Such is to say: each generation is a _pure function_ of the preceding one.
* The rules continue to be applied repeatedly to spawn further generations.

---

## Why bother?

* Each _tick_ is a pure function, operating on the output of the previous tick.
* Great exercise for learning a new language or function library.

---

# History

---

## Origins

Late 1940, John von Neumann set forth a definition for life, namely as a creation (as a being or organism) which:

1.  Can reproduce itself, and
2.  Can simulate a Turing machine.

---

## Later

* Formalized by British mathematician John Horton Conway in 1970
* Conway's specification succeeded in fulfilling von Neumann's two axioms

---

Conway's set forth his own set of rules:

1.  There should be no explosive growth.
1.  There should exist small initial patterns with chaotic, unpredictable outcomes.
1.  There should be potential for von Neumann universal constructors.
1.  The rules should be as simple as possible, whilst adhering to the above constraints.

---

* The game made its first public appearance in the October 1970 issue of _Scientific American,_ in Martin Gardner's "Mathematical Games" column.
* Conway's specification is remarkably elegant, especially when compared with its potential outcomes.

---

# Visualizations

---

## Randomized

![](http://www.diga.me.uk/LifeAnimation.gif)

---

## Breeder

![](https://upload.wikimedia.org/wikipedia/commons/e/e6/Conways_game_of_life_breeder_animation.gif)

---

## Halfmax (Spacefiller)

![](https://i.imgur.com/7ois83S.gif)

---

# Let's code

---

## On matters of libraries and of style

In the vein of functional programming:

* We'll employ use of RamdaJS, which includes a lot of utilities for evaluating _relations,_ such as `contains()` and `intersection()`.
* We'll lean _heavily_ on `reduce()`. All, excepting one, of our functions will return calculations derived from `reduce()`.

---

# Our algorithm

---

## Most commonly

* Many implementations are _egocentric,_ such that:
* Cells are evaluated in light of each of their neighbors, and the rules accordingly applied.

---

## On the other hand

* We can abstract different rules that evaluate entire neighborhoods.

---

## Accordingly

1.  If the sum of all nine fields is 3, the inner field state for the next generation will be life (regardless its previous contents).
2.  If the all-field sum is 4, the inner field retains its current state.
3.  For any other sum, we delegate death to the inner field.

---

## Our representation

---

Our grid, and our stores for all cells currently alive, shall assume the following shape:

```javascript
[[0, 0], [0, 1] ...]
```

---

The crux of our implementation will reside with two functions, namely:

* `neighborhood()`, which, when provided a reference to a cell, returns values for the provided cell _and_ values for all of its neighboring cells.
* `step()`, which, when provided a representation of all coordinates, and a list of all coordinates currently alive, will recursively apply new calculated states.

---

## `neighborhood()`

```javascript
function neighborhood([x = 0, y = 0] = []) {
  return reduce(
    (acc, dx) => concat(acc, map(dy => [dx, dy], range(y - 1, y + 2))),
    [],
    range(x - 1, x + 2)
  )
}
```

---

## Helpers

We'll also make use of some helper functions to set up our game, namely:

* `consGrid()`, which, when provided a value for length/width, returns a representation of our world, i.e., our _grid._
* `seedRandom()`, which, when provided a _grid,_ returns a list of values for populating our game.
* `paint()`, which, when provided a list of all cells currently alive, leverages HTML's `canvas` API to render our game.

---

## `consGrid()`

```javascript
function consGrid(size = 1) {
  return reduce(
    (acc, x) => concat(acc, map(y => [x, y], range(0, size))),
    [],
    range(0, size)
  )
}
```

---

## `seedRandom()`

```javascript
function seedRandom(grid = [], odds = 0.5) {
  return reduce(
    (acc, [x, y]) => (Math.random() > odds ? [[x, y], ...acc] : acc),
    [],
    grid
  )
}
```

---

## `paint()`

```javascript
function paint(arrIn = [], w = 10) {
  const context = document.getElementById('canvas').getContext('2d')
  context.clearRect(0, 0, 1000, 1000)
  return forEach(
    cell => context.fillRect(head(cell) * w, tail(cell) * w, 1 * w, 1 * w),
    arrIn
  )
}
```

---

## `step()`

```javascript
function step(world = [], arrIn = []) {
  const arrOut = reduce(
    (acc, curr) =>
      cond([
        [equals(3), always([curr, ...acc])],
        [equals(4), () => (contains(curr, arrIn) ? [curr, ...acc] : acc)],
        [T, always(acc)],
      ])(intersection(neighborhood(curr), arrIn).length),
    [],
    world
  )
  paint(arrOut)
  return setTimeout(() => step(world, arrOut), 250)
}
```

---

## Notable variations

---

## SmoothLife

Floating point game of life

![](http://moiscript.weebly.com/uploads/3/9/3/8/3938813/glups.gif)

---

## Videos of SmoothLife

* http://arxiv.org/pdf/1111.1567v2.pdf
* https://www.youtube.com/watch?v=KJe9H6qS82I
* https://www.youtube.com/watch?v=eaIgKXwzTQQ

---

## Libraries

* _**Ramda -**_ A practical functional library for JavaScript programmers: http://ramdajs.com/

---

## Further reading

* _A Discussion of the Game of Life,_ http://web.stanford.edu/~cdebs/GameOfLife/
* Jake Barnwell, _The Game of Life!,_ an interactive sandbox, Massachusetts Institute of Technology, http://web.mit.edu/jb16/www/6170/gameoflife/gol.html?
* _Conway's Game of Life,_ implementations of Game of Life in virtually every language, Rosetta Code, http://rosettacode.org/wiki/Conway%27s_Game_of_Life

---

## Created using

# Reveal-MD
