<template>
  <div class="pooch">
    <div class="pooch__photo">
      <img class="photo" alt="photo" :src="localPhoto" @error="setDefaultOnError">
    </div>
    <div class="pooch__data">
      <div class="pooch__info">
        <div class="pooch__headline">
          <div class="pooch__name" v-html="getName" />
          <a v-if="isCorrectEmail" :href="'mailto:'+email" class="pooch__email" v-html="getEmail" />
        </div>
        <div class="pooch__position" v-html="getPosition" />
        <div class="pooch__full-address">
          <span v-if="getAddress" class="pooch__address" v-html="getAddress" /><span v-if="getCity" class="pooch__city" v-html="getCity" />
        </div>
      </div>
      <div class="divider" :class="{'-transparent': marked}" />
      <div class="pooch__button-container">
        <button class="pooch__button" @click="toggleSelection">
          {{ buttonText }}
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Pooch',
  props: {
    name: {
      type: String,
      required: true,
      default: 'Unknown name'
    },
    photo: {
      type: String,
      default () {
        return require('~/assets/img/no-photo.png')
      }
    },
    email: {
      type: String,
      default: ''
    },
    position: {
      type: String,
      default: ''
    },
    address: {
      type: String,
      default: ''
    },
    city: {
      type: String,
      default: ''
    },
    searchValue: {
      type: String,
      default: ''
    }
  },
  data () {
    return {
      localPhoto: '',
      marked: false
    }
  },
  computed: {
    getName () {
      return this.markWord('name')
    },
    getEmail () {
      return this.markWord('email')
    },
    getPosition () {
      return this.markWord('position')
    },
    getAddress () {
      return this.markWord('address')
    },
    getCity () {
      return this.markWord('city')
    },
    isCorrectEmail () {
      return this.email && /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/.test(this.email)
    },
    buttonText () {
      return this.marked ? 'SKIP SELECTION' : 'MARK AS SUITABLE'
    }
  },
  watch: {
    photo: {
      handler (newPhoto) {
        this.localPhoto = newPhoto
      },
      immediate: true
    }
  },
  created () {
    console.log('pooch created')
  },
  methods: {
    markWord (name) {
      let result = this[name]
      if (result && this.searchValue) {
        const regexp = new RegExp(`(${this.searchValue.replace(/[-\\^$*+?.()|[\]{}]/g, '\\$&')})`, 'gim')
        result = result.replace(regexp, '<mark>$1</mark>')
      }
      return result
    },
    setDefaultOnError () {
      this.localPhoto = require('~/assets/img/no-photo.png')
    },
    toggleSelection () {
      this.marked = !this.marked
      this.$emit('selectionChange', this.marked)
    }
  }
}
</script>

<style lang="scss" scoped>
.pooch {
  display: flex;
  background: #fafafa;
  box-shadow: 0 0 2px rgba(0, 0, 0, 0.12), 0 2px 2px rgba(0, 0, 0, 0.24);
  border-radius: 3px;

  &__photo {
    flex: 0 0 134px;
    min-height: 136px;
    align-self: stretch;
  }

  &__data {
    flex-grow: 1;
  }

  &__headline {
    display: flex;
    justify-content: space-between;
    flex-wrap: wrap;
  }

  &__name {
    font-size: 24px;
    line-height: 32px;
    color: rgba(0, 0, 0, 0.87);
  }

  &__email {
    font-size: 14px;
    line-height: 16px;
    color: rgba(0, 0, 0, 0.54);
    text-decoration: none;
  }

  &__position {
    font-weight: bold;
    font-size: 14px;
    line-height: 20px;
  }

  &__full-address {
    font-size: 14px;
    line-height: 20px;
  }

  &__info {
    padding: 10px 9px 2px 27px;
  }

  &__button-container {
    padding: 14px 32px 19px 32px;
  }

  &__button {
    width: 138px;
  }
}

.pooch__address + .pooch__city::before {
  content: ', ';
}
</style>
