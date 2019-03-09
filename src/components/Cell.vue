<template>
  <input
    v-model.number="current"
    pattern="[1-9]"
    type="text"
    class="cell"
    @keydown.up="current++"
    @keydown.down="current--"
  >
</template>

<script>
export default {
  props: {
    value: {
      type: Number,
      required: true
    }
  },
  data () {
    return {
      current: ''
    }
  },
  watch: {
    value: {
      handler (val) {
        this.current = val || ''
      },
      immediate: true
    },
    current (val, oldVal) {
      // validation
      if (val === '') {
        this.$emit('input', 0)
        return
      }

      if (val <= 0) {
        this.current = ''
        return
      }

      if (typeof val === 'string') {
        this.current = oldVal
        return
      }

      if (val > 9) this.current = val % 10

      this.$emit('input', this.current)
    }
  }
}
</script>

<style lang="scss" scoped>
.cell {
  text-align: center;
  width: 50px;
  height: 50px;
  border: 1px solid black;

  background-color: white;
  transition: background-color 0.3s;
  &[disabled] {
    cursor: not-allowed;
    color: rgba(black, 0.5);
    font-weight: bold;
  }
  &.error {
    background-color: rgba(red, 0.4);
  }
  &.success {
    background-color: rgba(green, 0.4);

    &.error {
      background-color: rgba(yellow, 0.4);
    }
  }
}
</style>
