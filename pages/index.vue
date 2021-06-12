<template>
  <div class="container">
    <custom-input />
    <div ref="scroller" class="pooch-container" @scroll="resizeOutputList">
      <pooch
        v-for="item in outputList"
        :key="item.name+item.email"
        class="-mt-20"
        :name="item.name"
        :address="item.address"
        :city="item.city"
        :position="item.title"
        :email="item.email"
        :photo="item.avatar"
      />
    </div>
  </div>
</template>

<script>
export default {
  // "name":"Ms. Alfonzo Stoltenberg","email":"keven@grady.net","title":"Direct Implementation Producer","city":"Norvalville","address":"9138 Stiedemann Ports","avatar":"https://robohash.org/possimusdoloresid.png?size=300x300\u0026set=set1"
  // потом эту ебалу надо пернести в стэйт по хорошему
  // хотя есть вопрос, в нужен ли стэйт или нет на такие маленькие объемы?
  // для понтов и и обратимости посикат олько что...
  data () {
    return {
      list: [],
      outputCount: 50,
      step: 50,
      // надо бы красоту для ошибок сделать
      isError: false
    }
  },
  computed: {
    // плохое решение, весь массив завново пересобираем
    outputList () {
      return this.list.slice(0, this.outputCount)
    }
  },
  created () {
    this.getData()
  },
  methods: {
    async getData () {
      const response = await fetch('https://gist.githubusercontent.com/allaud/093aa499998b7843bb10b44ea6ea02dc/raw/c400744999bf4b308f67807729a6635ced0c8644/users.json')
      if (response.ok) {
        this.list = await response.json()
      } else {
        this.isError = true
      }
    },
    resizeOutputList (event) {
      const scroller = this.$refs.scroller
      if (scroller.scrollHeight - scroller.scrollTop < 1200) {
        this.outputCount += this.step
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.container {
  max-width: 564px;
  margin: 0 auto;
  height: 100%;
  background: #fff;
  padding: 19px 12px 0;
}

.pooch-container {
  overflow: auto;
  height: 100%;
}

.poach {
  display: none;
}
</style>
