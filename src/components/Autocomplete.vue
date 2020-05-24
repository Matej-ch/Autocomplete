<template>
  <div class="autocomplete">
    <input type="text" v-model="search" @input="onChange"  @keydown.down="onArrowDown" @keydown.up="onArrowUp" @keydown.enter="onEnter">
    <ul class="auto-results" v-show="isOpen">
      <li class="auto-result" :class="{ 'is-active': i === arrowCounter }" v-for="(result, i) in results" :key="i" @click="setResult(result)">
        {{result}}
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: 'Autocomplete',
  props: {
    items: {
      type: Array,
      required: false,
      default: () => []
    },
    itemsUrl: {
      type: String,
    },
    minChar: {
      type: Number,
      default: 3
    }
  },
  data() {
    return {
      search: '',
      results: [],
      isOpen: false,
      arrowCounter: -1,
      charCount: 0
    }
  },
  methods: {
    onChange() {
      if(this.search.length >= this.minChar) {
        this.isOpen = true;
        this.$http.get(this.itemsUrl).then(response => this.filterResults(response.data));

      }
    },
    filterResults(items) {
      this.results = items.filter(item => item.toLowerCase().indexOf(this.search.toLowerCase()) > -1);
    },
    setResult(result) {
      this.search= result;
      this.isOpen = false;
    },
    onArrowDown() {
      if (this.arrowCounter < this.results.length) {
        this.arrowCounter = this.arrowCounter + 1;
      }
    },
    onArrowUp() {
      if (this.arrowCounter > 0) {
        this.arrowCounter = this.arrowCounter - 1;
      }
    },
    onEnter() {
      this.search = this.results[this.arrowCounter];
      this.isOpen = false;
      this.arrowCounter = -1;
    },
    handleClickOutside(e) {
      if (!this.$el.contains(e.target)) {
        this.isOpen = false;
        this.arrowCounter = -1;
      }
    }
  },
  mounted() {
    document.addEventListener('click', this.handleClickOutside);
  },
  destroyed() {
    document.removeEventListener('click', this.handleClickOutside);
  }
}
</script>

<style scoped>
  .auto-result.is-active,
  .auto-result:hover {
    background-color: #4b7daf;
    color: white;
  }

  .autocomplete {
    display: flex;
    flex-direction: column;
  }

  .autocomplete input {
    display: block;
    width: 100%;
    padding: .375rem .75rem;
    font-size: 1rem;
    line-height: 1.5;
    color: #495057;
    background-color: #fff;
    background-clip: padding-box;
    border: 1px solid #ced4da;
    border-radius: .25rem;
    transition: border-color .15s ease-in-out,box-shadow .15s ease-in-out;
    box-sizing:border-box;
  }

  .auto-results {
    padding: 0;
    margin: 0;
    overflow: auto;
    height: 200px;
    border: 1px solid #eeeeee;
  }

  .auto-results .auto-result {
    list-style: none;
    text-align: left;
    padding: 4px 2px;
    cursor: pointer;
  }

  .auto-results .auto-result:hover {
    background-color: #4b7daf;
    color: white;
  }
</style>
