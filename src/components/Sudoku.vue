<template>
  <div class="sudoku">
    <h1 class="sudoku-title">
      Судоку
    </h1>
    <div
      v-if="!started"
      class="sudoku-config"
    >
      <div class="sudoku-config-initial">
        <p>
          Количество начальных подсказок (от {{ initialRange.min }} до {{ initialRange.max }})
        </p>
        <div class="sudoku-config-initial-range">
          <input
            v-model.number="initial"
            type="range"
            :min="initialRange.min"
            :max="initialRange.max"
          >
          <span class="sudoku-config-initial-value">
            {{ initial }}
          </span>
        </div>
      </div>
      <div>
        <button @click="init">
          Запустить
        </button>
      </div>
      <div class="sudoku-config-rules">
        <h2>
          Правила:
        </h2>
        <p>
          Игровое поле представляет собой квадрат размером 9×9, разделённый на меньшие квадраты со стороной в 3 клетки. Таким образом, всё игровое поле состоит из 81 клетки. В них уже в начале игры стоят некоторые числа (от 1 до 9), называемые подсказками. От игрока требуется заполнить свободные клетки цифрами от 1 до 9 так, чтобы в каждой строке, в каждом столбце и в каждом малом квадрате 3×3 каждая цифра встречалась бы только один раз.
        </p>
      </div>
      <div class="sudoku-config-controls">
        <h2>
          Управление:
        </h2>
        <p>
          Ячейки способны окрашиваться в 3 цвета: красный, желтый и зеленый:
        </p>
        <ul>
          <li>
            Красный указывает на наличие ошибки в ряде / колонке / квадрате 3х3
          </li>
          <li>
            Зеленый указывает на правильную полную заполненность в ряде / колонке / квадрате 3х3
          </li>
          <li>
            Желтый является комбинацией предыдущих 2 случаев
          </li>
        </ul>
        <p>
          Можно играть как с помощью мышки, так и с помощью клавиатуры:
        </p>
        <ul>
          <li>
            tab - переключение к следующей ячейке
          </li>
          <li>
            клавиша вверх / клавиша вниз позволяют менять значение текущей ячейки
          </li>
        </ul>
      </div>
    </div>
    <div
      v-if="started"
      class="sudoku-container"
    >
      <div
        v-for="(square, j) in finalSquares"
        :key="j"
        class="square"
      >
        <cell
          v-for="(cell, i) in square"
          :key="i"
          v-model.number="values[cell.index]"
          :class="{
            error: cell.error,
            success: cell.success
          }"
          :disabled="victory || disabled[cell.index]"
          :tabindex="cell.index + 1"
        />
      </div>
    </div>
    <div
      v-if="!victory && started"
      class="sudoku-retry"
    >
      <p>
        Не получается?
      </p>
      <button @click="started = false">
        Попробовать заново
      </button>
    </div>
    <div
      v-if="victory && started"
      class="sudoku-victory"
    >
      <h2>
        Вы победили
      </h2>
      <p>
        Попробовать снова?
      </p>
      <button @click="started = false">
        Еще раз!
      </button>
    </div>
  </div>
</template>

<script>
import Cell from './Cell'
export default {
  components: {
    Cell
  },
  data () {
    const step = 9

    return {
      started: false,
      step: step,
      miniStep: step / 3,
      values: [],
      disabled: [],
      initial: 40,
      initialRange: {
        min: 17,
        max: 76
      }
    }
  },
  computed: {
    cells () {
      // combined array from values and indexes (row, column, square indexes)
      return this.values.map((value, index) => {
        return {
          value: value,
          ...this.indexes[index]
        }
      })
    },
    indexes () {
      // array of indexes (row, column. square indexes)
      return new Array(this.step * this.step).fill().map((value, index) => this.cellIndexes(index))
    },
    rows () {
      return this.fromCells('row')
    },
    columns () {
      return this.fromCells('column')
    },
    squares () {
      return this.fromCells('square')
    },
    finalSquares () {
      // final array of squares to render
      // has both error and success keys
      // also has index for binding with initial values array
      let finalSquares = this.arrayOfArrays(this.step)

      this.cells.forEach((cell, index) => {
        finalSquares[cell.squareIndex].push({
          error: this.isError(index),
          success: this.isSuccess(index),
          index: index
        })
      })

      return finalSquares
    },
    victory () {
      // the victory condition
      return (
        !this.rows.find(row => !row.success) &&
        !this.columns.find(column => !column.success) &&
        !this.squares.find(square => !square.success)
      )
    }
  },
  methods: {
    init () {
      this.started = false
      this.values = new Array(this.step * this.step).fill(0)
      this.generateValues()
      this.deleteSomeValues()
      this.disableInitialValues()
      this.started = true
    },
    // generation functions
    generateValues (index = 0, banned = []) {
      // first find 81 valid values
      if (index >= this.step * this.step) return true

      let localBanned = banned.slice()
      if (localBanned.length > 8) return false

      const value = this.generateValue(index, localBanned)
      if (!value) return false

      const next = this.generateValues(index + 1)

      if (!next) {
        localBanned.push(value)
        return this.generateValues(index, localBanned)
      }

      return true
    },
    generateValue (index, banned = []) {
      // finding value for one (!) cell
      let localBanned = banned.slice()
      if (localBanned.length > 8) return false

      const value = Math.floor(Math.random() * 9) + 1
      if (localBanned.includes(value)) return this.generateValue(index, localBanned)

      this.$set(this.values, index, value)

      if (this.isError(index)) {
        this.$set(this.values, index, 0)
        localBanned.push(value)
        return this.generateValue(index, localBanned)
      }

      return value
    },
    deleteSomeValues () {
      // second delete some of values
      const toDelete = this.step * this.step - this.initial
      for (let i = 0; i < toDelete; i++) {
        const index = this.getIndexToDelete()
        this.$set(this.values, index, 0)
      }
    },
    getIndexToDelete () {
      // finding index to delete
      const index = Math.floor(Math.random() * 80)
      if (!this.values[index]) return this.getIndexToDelete()
      return index
    },
    disableInitialValues () {
      // set initial values disabled
      this.disabled = this.values.map(value => !!value)
    },
    isError (index) {
      // condition of error in cell
      const cell = this.indexes[index]
      return this.rows[cell.rowIndex].error ||
             this.columns[cell.columnIndex].error ||
             this.squares[cell.squareIndex].error
    },
    isSuccess (index) {
      // condition of success in cell
      const cell = this.indexes[index]
      return this.rows[cell.rowIndex].success ||
             this.columns[cell.columnIndex].success ||
             this.squares[cell.squareIndex].success
    },
    fromCells (type) {
      // get array of type (row, column, square)
      const indexType = `${type}Index`
      let cellsArray = this.arrayOfArrays(this.step)

      this.cells.forEach(cell => {
        cellsArray[cell[indexType]].push(cell.value)
      })

      return cellsArray.map(cells => {
        const isClean = this.isClean(cells)
        const hasDuplicates = this.hasDuplicates(cells)

        return {
          values: cells,
          error: hasDuplicates,
          success: !hasDuplicates && isClean
        }
      })
    },

    // Additional functions
    cellIndexes (index) {
      // row, column, square indexes for cell from initial index
      const rowIndex = Math.floor(index / this.step)
      const columnIndex = index - rowIndex * this.step
      const squareIndex = Math.floor(rowIndex / this.miniStep) * this.miniStep + Math.floor(columnIndex / this.miniStep)

      return {
        rowIndex: rowIndex,
        columnIndex: columnIndex,
        squareIndex: squareIndex
      }
    },
    hasDuplicates (array) {
      // condition of having duplicate in array
      let cleanArray = array.filter(element => element)
      return (new Set(cleanArray)).size !== cleanArray.length
    },
    isClean (array) {
      // condition of not having zeros (or NaN or undefined) in array
      return array.filter(element => element).length === array.length
    },
    arrayOfArrays (size) {
      // returns array of 9 empty arrays
      return new Array(size).fill().map(column => [])
    }
  }
}
</script>

<style lang="scss" scoped>
.sudoku {
  text-align: center;
}
.sudoku-title {
  margin-bottom: 1rem;
}
.sudoku-config-initial {
  width: 260px;
  margin: 0 auto;
}
.sudoku-config-initial-range {
  display: flex;
  align-items: center;
  justify-content: center;
}
.sudoku-config-initial-value {
  margin-left: 0.4em;
}
.sudoku-config-rules,
.sudoku-config-controls {
  width: 600px;
  max-width: 100%;
  margin: 2em auto 0;
  text-align: left;
}
.sudoku-container {
  // using grid only in demo
  display: inline-grid;
  grid-template-rows: 1fr 1fr 1fr;
  grid-template-columns: 1fr 1fr 1fr;

  margin-bottom: 1em;
}
.square {
  border: 1px solid black;
  display: inline-grid;
  grid-template-rows: 1fr 1fr 1fr;
  grid-template-columns: 1fr 1fr 1fr;
}
</style>
