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
      isFilter: false
    }
  },
  created () {
    // init non reactive fields
    this.list = []
    this.filteredList = []
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
        this.outputList = this.isFilter ? this.filteredList.slice(0, this.outputCount) : this.list.slice(0, this.outputCount)
      }
    },
    filter (value) {
      if (value) {
        if (value.length > 1) {
          if (this.prevFilterValue && value.includes(this.prevFilterValue)) {
            this.filteredList = this.findAndMarkWord(value, this.filteredList, true)
          } else {
            this.filteredList = this.findAndMarkWord(value, this.list)
          }
          this.isFilter = true
          this.outputCount = this.step
          this.outputList = this.filteredList.slice(0, this.outputCount)
          this.prevFilterValue = value
        }
      } else {
        // как-то не весело. вот проскролили мы весь список
        // чо то там отфильтровали
        // а теперь опять сначала?
        // можно для фильтрации совсем другой список показывать
        // но тогда верстка и объем памяти в два раза возрастут...
        // и большой вопрос, а нормально ли отрендерит браузер к примеру сразу 1000 элементов?
        // так то мы из по чуть чуть хуярим
        this.isFilter = false
        this.outputCount = this.step
        this.outputList = this.list.slice(0, this.outputCount)
      }
    },
    findAndMarkWord (value, array, clearMarked) {
      const result = []
      if (Array.isArray(array) && value) {
        // переделать на цикл
        // тут же в случае совпадения поискового слова создавать новый объект и созранять его
        const regexp = new RegExp(`(${value.replace(/[-\\^$*+?.()|[\]{}]/g, '\\$&')})`, 'gim')
        for (let i = 0; i < array.length; i++) {
          let item = array[i]
          if (clearMarked) {
            for (const key in item) {
              item[key] = item[key].replace(/<\/?mark>/g, '')
            }
          }
          let isCopied = false
          for (const key in item) {
            if (key !== 'avatar' && item[key] && item[key].toLowerCase().includes(value.toLowerCase())) {
              if (!isCopied) {
                item = Object.assign({}, item)
                array[i] = item
                isCopied = true
              }
              console.log(i, item[key])
              console.log(i, item)
              item[key] = item[key].replace(regexp, '<mark>$1</mark>')
            }
          }
          isCopied && result.push(item)
        }
        // return array.filter((item, index) => {
        //   if (clearMarked) {
        //     for (const key in item) {
        //       item[key] = item[key].replace(/<\/?mark>/g, '')
        //     }
        //   }
        //   let counter = 0
        //   let isCopied = false
        //   for (const key in item) {
        //     if (key !== 'avatar' && item[key] && item[key].toLowerCase().includes(value.toLowerCase())) {
        //       if (!isCopied) {
        //         item = Object.assign({}, item)
        //         isCopied = true
        //       }
        //       item[key] = item[key].replace(regexp, '<mark>$1</mark>')
        //       counter++
        //     }
        //   }
        //   return counter
        // })
      }
      return result
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
