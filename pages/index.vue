<template>
  <div class="container">
    <custom-input @change="filter" />
    <div ref="scroller" class="pooch-container" @scroll="resizeOutputList">
      <pooch
        v-for="(item, index) in outputList"
        :key="item.name+item.email"
        class="-mt-20"
        :name="item.name"
        :address="item.address"
        :city="item.city"
        :position="item.title"
        :email="item.email"
        :photo="item.avatar"
        @selectionChange="selectionChange($event, item, index)"
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
  //
  // тут есть мысль что нахуй стэйт. потому что весит дохера из-за реактивности
  data () {
    return {
      outputList: [],
      outputCount: 50,
      step: 50,
      // надо бы красоту для ошибок сделать
      isError: false
    }
  },
  created () {
    // init non reactive fields
    this.list = []
    this.prevFilterValue = ''
    // get data
    this.getData()
  },
  methods: {
    async getData () {
      const response = await fetch('https://gist.githubusercontent.com/allaud/093aa499998b7843bb10b44ea6ea02dc/raw/c400744999bf4b308f67807729a6635ced0c8644/users.json')
      if (response.ok) {
        this.list = await response.json()
        if (Array.isArray(this.list)) {
          this.outputList = this.list.slice(0, this.outputCount)
        }
      } else {
        this.isError = true
      }
    },
    // рабочая но по моему не самая хорошая реализация
    // потому что рано или поздно записей будет ну ооочень много, и в теории сам по себе проход по и проверка ключей по 10000+ записей будет долгим
    // но если виртуальный скролл будет работать хорошо, то можно оставить
    resizeOutputList () {
      const scroller = this.$refs.scroller
      if (scroller.scrollHeight - scroller.scrollTop < 1200) {
        this.outputCount += this.step
        this.outputList = this.list.slice(0, this.outputCount)
      }
    },
    filter (value) {
      if (value && value.length > 1) {
        if (this.prevFilterValue && value.includes(this.prevFilterValue)) {
          this.outputList = this.findAndMarkWord(value, this.outputList, true)
        } else {
          this.outputList = this.findAndMarkWord(value, this.list.slice(0, 50))
        }
        this.prevFilterValue = value
      } else {
        // как-то не весело. вот проскролили мы весь список
        // чо то там отфильтровали
        // а теперь опять сначала?
        // можно для фильтрации совсем другой список показывать
        // но тогда верстка и объем памяти в два раза возрастут...
        // и большой вопрос, а нормально ли отрендерит браузер к примеру сразу 1000 элементов?
        // так то мы из по чуть чуть хуярим
        this.outputCount = this.step
        this.outputList = this.list.slice(0, this.outputCount)
      }
    },
    findAndMarkWord (value, array, clearMarked) {
      if (Array.isArray(array) && value) {
        // переделать на цикл
        // тут же в случае совпадения поискового слова создавать новый объект и созранять его
        return array.filter((item) => {
          if (clearMarked) {
            for (const key in item) {
              item[key] = item[key].replace(/<\/?mark>/g, '')
            }
          }
          let counter = 0
          for (const key in item) {
            if (item[key] && item[key].toLowerCase().includes(value.toLowerCase())) {
              const regexp = new RegExp(value.replace(/[-\\^$*+?.()|[\]{}]/g, '\\$&'), 'gim')
              item[key] = item[key].replace(regexp, `<mark>${value}</mark>`)
              counter++
            }
          }
          return counter
        })
      }
      return []
    },
    selectionChange (event, item, index) {}
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
