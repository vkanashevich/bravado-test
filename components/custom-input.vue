<template>
  <div class="search">
    <div class="search__icon" @click="goToInput">
      <slot><i class="icon -search" /></slot>
    </div>
    <input
      ref="search"
      :value="value"
      type="text"
      class="search__input"
      placeholder="Type search text here (min. 2 chars)"
      @keyup="onChange"
    >
  </div>
</template>

<script>
export default {
  name: 'CustomInput',
  props: {
    value: {
      type: String,
      default: ''
    }
  },
  methods: {
    onChange () {
      clearTimeout(this.emitTemeout)
      this.emitTemeout = setTimeout(() => {
        this.$emit('input', this.$refs.search.value)
      }, 200)
    },
    goToInput () {
      this.$refs.search && this.$refs.search.focus()
    }
  }
}
</script>

<style lang="scss" scoped>
.search {
  background: #fafafa;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.12), 0 2px 2px rgba(0, 0, 0, 0.24);
  border-radius: 2px;
  padding: 12px 16px 7px 16px;
  display: flex;

  &__icon {
    cursor: pointer;
    margin-right: 10px;
    flex: 0 0 24px;
    height: 24px;
  }

  &__input {
    font-size: 24px;
    line-height: 28px;
    border: none;
    background: none;
    flex-grow: 1;

    &:focus {
      outline: none;
    }
  }
}

</style>
