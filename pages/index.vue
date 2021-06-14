<template>
  <div class="container">
    <custom-input @change="filter" />
    <div ref="scroller" class="pooch-container" @scroll.passive="poochScroll">
      <pooch
        v-for="(item, index) in outputList"
        :key="item.name+item.email"
        class="-mb-20"
        :name="item.name"
        :address="item.address"
        :city="item.city"
        :position="item.title"
        :email="item.email"
        :photo="item.avatar"
        :search-value="searchValue"
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
      isError: false,
      searchValue: ''
    }
  },
  created () {
    // init non reactive fields
    this.sourceList = []
    this.filteredList = []
    this.memoization = new Map()
    this.prevsearchValue = ''
    // get data
    this.getData()
  },
  methods: {
    async getData () {
      const response = await fetch('https://gist.githubusercontent.com/allaud/093aa499998b7843bb10b44ea6ea02dc/raw/c400744999bf4b308f67807729a6635ced0c8644/users.json')
      if (response.ok) {
        this.sourceList = await response.json()
        if (Array.isArray(this.sourceList)) {
          this.outputList = this.sourceList.slice(0, this.outputCount)
        }
      } else {
        this.isError = true
      }
    },
    poochScroll () {
      window.requestAnimationFrame(this.resizeOutputList)
    },
    // рабочая но по моему не самая хорошая реализация
    // потому что рано или поздно записей будет ну ооочень много, и в теории сам по себе проход по и проверка ключей по 10000+ записей будет долгим
    // но если виртуальный скролл будет работать хорошо, то можно оставить
    resizeOutputList () {
      const scroller = this.$refs.scroller
      if (scroller.scrollHeight - scroller.scrollTop < 1200) {
        this.outputCount += this.step
        this.outputList = this.searchValue ? this.filteredList.slice(0, this.outputCount) : this.sourceList.slice(0, this.outputCount)
      }
    },
    filter (value) {
      this.searchValue = value
      if (value) {
        if (value.length > 1) {
          const cachedList = this.getFromMemoization(value)
          if (!cachedList) {
            if (this.prevsearchValue && value.includes(this.prevsearchValue)) {
              this.filteredList = this.filterList(value, this.filteredList)
            } else {
              this.filteredList = this.filterList(value, this.sourceList)
            }
            this.addToMemoization(value, this.filteredList)
          } else {
            this.filteredList = cachedList
          }
          this.outputCount = this.step
          this.outputList = this.filteredList.slice(0, this.outputCount)
          this.prevsearchValue = value
        }
      } else {
        // как-то не весело. вот проскролили мы весь список
        // чо то там отфильтровали
        // а теперь опять сначала?
        // можно для фильтрации совсем другой список показывать
        // но тогда верстка и объем памяти в два раза возрастут...
        // и большой вопрос, а нормально ли отрендерит браузер к примеру сразу 1000 элементов?
        // так то мы из по чуть чуть хуярим
        this.outputCount = this.step
        this.outputList = this.sourceList.slice(0, this.outputCount)
      }
    },
    filterList (value, array) {
      const result = []
      if (Array.isArray(array) && value) {
        const lowerValue = value.toLowerCase()
        for (let i = 0; i < array.length; i++) {
          const item = array[i]
          for (const key in item) {
            if (key !== 'avatar' && item[key].toLowerCase().includes(lowerValue)) {
              result.push(item)
              break
            }
          }
        }
      }
      return result
    },
    addToMemoization (value, dataList) {
      if (value && value.length > 2 && Array.isArray(dataList)) {
        const key = value.toLowerCase()
        !this.memoization.has(key) && this.memoization.set(value, dataList.slice())
      }
    },
    getFromMemoization (value) {
      return this.memoization.get(value)
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
  overflow: hidden;
}

.pooch-container {
  overflow: auto;
  height: 100%;
  margin-top: 20px;
}

.poach {
  display: none;
}
</style>
