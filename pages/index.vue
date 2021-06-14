<template>
  <div class="container" :class="{loader: isLoading}">
    <custom-input v-model="searchValue" @input="filter" />
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
      <div v-if="searchValue && !outputList.length" class="found-nothing">
        Found nothing
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // array that contains result list
      outputList: [],
      // cols of all items in page
      outputCount: 50,
      // step of increment items
      step: 50,
      isError: false,
      // model for search inout
      searchValue: '',
      isLoading: true
    }
  },
  created () {
    console.log('index created')
    // init non reactive fields
    // non reactive list of all data
    this.sourceList = []
    // non reactive list with filtered data only
    this.filteredList = []
    // simple storage for caching search results. without clearing unfortunately
    this.memoization = new Map()
    // non reactive field with previous filtered value to improve filtering speed
    this.prevsearchValue = ''
    // to support stateful search
    if (this.$route.params.value) {
      this.searchValue = this.$route.params.value
    }
    this.getData()
  },
  methods: {
    // getting data
    async getData () {
      const response = await fetch('https://gist.githubusercontent.com/allaud/093aa499998b7843bb10b44ea6ea02dc/raw/c400744999bf4b308f67807729a6635ced0c8644/users.json')
      if (response.ok) {
        this.sourceList = await response.json()
        if (Array.isArray(this.sourceList)) {
          this.filter()
        }
      } else {
        this.isError = true
      }
      this.isLoading = false
    },
    // function to increase items on scroll
    poochScroll () {
      window.requestAnimationFrame(this.resizeOutputList)
    },
    // check scroll height and add new items to output list here
    resizeOutputList () {
      const scroller = this.$refs.scroller
      if (scroller.scrollHeight - scroller.scrollTop < 1200) {
        this.outputCount += this.step
        this.outputList = this.searchValue ? this.filteredList.slice(0, this.outputCount) : this.sourceList.slice(0, this.outputCount)
      }
    },
    // method is called when search input updates searchValue
    filter () {
      const value = this.searchValue
      if (value) {
        // work only with words longer than 1
        if (value.length > 1) {
          // to show user link to search url without reload page
          history.pushState('', 'searching for ' + value, '/search/' + value)
          // try to get from cache
          const cachedList = this.getFromMemoization(value)
          // filtering list
          if (!cachedList) {
            // if value is substring of previous search word we can work with current filtered list
            if (this.prevsearchValue && value.includes(this.prevsearchValue)) {
              this.filteredList = this.filterList(value, this.filteredList)
            } else {
              this.filteredList = this.filterList(value, this.sourceList)
            }
            // add to cache
            this.addToMemoization(value, this.filteredList)
          } else {
            this.filteredList = cachedList
          }
          // scroll list to top
          this.$refs.scroller.scrollTop = 0
          // update counters and output list
          this.outputCount = this.step
          this.outputList = this.filteredList.slice(0, this.outputCount)
          this.prevsearchValue = value
        }
      } else { // if no value - reset list and show first items
        this.$refs.scroller.scrollTop = 0
        history.pushState('', '', '/')
        this.outputCount = this.step
        this.outputList = this.sourceList.slice(0, this.outputCount)
      }
    },
    // filter function. just searching for first match
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
    // simple caching function without removing obsolete data
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
  display: flex;
  flex-direction: column;
}

.pooch-container {
  overflow: auto;
  margin-top: 20px;
}

.found-nothing {
  text-align: center;
  font-size: 24px;
  line-height: 28px;
}
</style>
