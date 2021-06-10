<template>
  <div class="pooch">
    <div class="pooch__photo">
      <img class="photo" alt="photo" :src="photo" @error="loadPhotoError">
    </div>
    <div class="pooch__data">
      <div class="pooch__info">
        <div class="pooch__headline">
          <div class="pooch__name">
            {{ name }}
          </div>
          <!-- проверку на email нужно впихнуть тоже -->
          <a v-if="isCorrectEmail" :href="'mailto:'+email" class="pooch__email">
            {{ email }}
          </a>
        </div>
        <div class="pooch__position">
          {{ position }}
        </div>
        <div class="pooch__address">
          {{ address }}
        </div>
      </div>
      <div class="divider" />
      <div class="pooch__button-container">
        <button class="pooch__button">
          MARK AS SUITABLE | SKIP SELECTION {{ test }}
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
    }
  },
  data () {
    return {
      test: ''
    }
  },
  computed: {
    isCorrectEmail () {
      return this.email && /^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/.test(this.email)
    }
  },
  methods: {
    loadPhotoError () {
      this.test = 'zzz'
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

  &__address {
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
</style>
