<template>
  <form v-on:submit.prevent class="filter-form">
    <label for="search">
      <span>Фильтр по производителям</span>
      <i class="icon-filter" :class="{ 'active': filterSelectedItems.length }" aria-hidden="true"></i>
    </label>
    <div ref="input-wrapper" class="input-wrapper">
      <input
        type="text"
        id="search"
        ref="search"
        :placeholder="placeholder"
        :value="filterInput"
        v-on:input="filterInputChange($event.target.value)"
      >
      <div class="input-controls">
        <button
          v-on:click="resetInput"
          v-show="filterInput.length > 2"
          class="input-reset"
          aria-label="Сбросить ввод"
        >
          <i class="icon-times-circle" aria-hidden="true"></i>
        </button>
        <button
          v-on:click="dropdownIsShown = !dropdownIsShown"
          class="input-dropdown"
          aria-label="Переключить фильтр"
          aria-controls="filterByManufacturer"
          :aria-expanded="dropdownIsShown ? 'true' : 'false'"
        >
          <i
            :class="[ dropdownIsShown ? 'icon-chevron-circle-down' : 'icon-chevron-circle-up' ]"
            aria-hidden="true"
          >
          </i>
        </button>
      </div>
      <ul v-if="dropdownIsShown" id="filterByManufacturer" class="input-list">
        <li v-for="item in filterSearchItems" :key="item">
          <label class="checkbox">
            <input
              type="checkbox"
              :value="item"
              v-model="filterSelectedItems"
            >
            <span class="checkbox-text">{{ item }}</span>
          </label>
        </li>
      </ul>
    </div>
    <button class="filter-reset" v-on:click="resetFilter">Сбросить</button> 
    <span class="filter-info" :class="{ 'filter-active': filterSelectedItems.length }">
      {{ filterSelectedItems.length ? 'Фильтр включен' : 'Фильтр выключен' }}
      <i :class="[ filterSelectedItems.length ? 'icon-check' : 'icon-exclamation-circle' ]"></i>
    </span>
    <div class="break"></div>
  </form>
</template>

<script>
import axios from 'axios'

import { bus } from '@/main'

export default {
  name: 'FilterPanel',
  data() {
    return {
      placeholder: "",
      filterInput: "",
      dropdownIsShown: false,
      filterData: [],
      filterSearchItems: [],
      filterSelectedItems: [],
    }
  },
  created() {
    axios
      .get(`${process.env.VUE_APP_API_BASE_PATH}/filter-items`)
      .then(response => {
        const { data } = response;
        this.filterData = this.filterSearchItems = data;
      })
  },
  mounted() {
    document.addEventListener('click', this.onClickListener)
    bus.$on('updateTables', () => {
      this.resetFilter()
    })
  },
  beforeDestroy() {
    document.removeEventListener('click', this.onClickListener)
  },
  watch: {
    $route() {
      this.resetFilter()
    },
    filterSelectedItems() {
      this.$emit('selectedFilters', this.filterSelectedItems)
      if (!this.filterSelectedItems.length) {
        this.placeholder = ""
        return null
      }
      this.placeholder = `${this.filterSelectedItems.length} из ${this.filterData.length} выбрано`
    }
  },
  methods: {
    filterInputChange(value) {
      this.filterInput = value
      this.dropdownIsShown = true;
      this.filterSearchItems = this.filterData.filter(item => {
        if (item.toLowerCase().includes(value.toLowerCase())) {
          return item;
        }
      })
    },
    resetInput() {
      this.filterInputChange('')
      this.$refs.search.focus()
    },
    resetFilter() {
      this.resetOnClickOutside()
      this.filterSelectedItems = []
    },
    onClickListener(event) {
      const target = event.target;
      if (
        !this.$refs['input-wrapper'].contains(target)
        && this.dropdownIsShown
      ) {
        this.resetOnClickOutside()
      }
    },
    resetOnClickOutside() {
      this.filterInputChange('')
      this.dropdownIsShown = false
    },
  }
}
</script>

<style lang="scss" scoped>
@import '@/assets/scss/other.scss';

.filter {
  &-form {
    background-color: bisque;
    display: flex;
    align-items: center;
    position: sticky;
    top: 0px;
    box-shadow: 0 2px 2px -1px #aba9a9;
    padding: 10px 0;
    > label {
      font-weight: 500;
      font-size: 18px;
      margin: 0 20px;
      padding-right: 16px;
      border-right: 1px solid #333333;
      > .icon-filter {
        display: none;
      }
    }
  }
  &-reset {
    -webkit-appearance: none;
    background-image: linear-gradient(180deg, #ececec, #f0f0f0);
    font-size: inherit;
    padding: 5px 6px;
    margin-right: 10px;
    outline: none;
    border: 1px solid grey;
    cursor: pointer;
    &:active {
      box-shadow: inset 0px 0px 2px 1px rgba(181, 181, 181, 0.74);
    }
  }
  &-info {
    border: 1px solid grey;
    color: #7d7d7d;
    padding: 5px 6px;
    margin-right: 10px;
  }
  &-active {
    border-color: #39c522;
    color: #2faf3e;
  }
}

@media screen and (max-width: 1056px) {
  .filter-form > label {
    > span {
      @include visually-hidden;
    }
    > .icon-filter {
      display: inline-block;
    }
  }
}

#search {
  position: relative;
  font-size: 16px;
  width: 200px;
  border: 1px solid grey;
  padding: 5px 45px 5px 10px;
  outline: none;
  &::-ms-clear {
    display: none;
  }
}

.input {
  &-wrapper {
    position: relative;
    margin-right: 10px;
  }
  &-list {
    overflow-x: hidden;
    position: absolute;
    top: 29px;
    list-style: none;
    border: 1px solid grey;
    width: 200px;
    max-height: 300px;
    background: white;
    li {
      height: 30px;
    }
  }
  &-controls {
    position: absolute;
    top: 0;
    right: 0;
    height: 100%;
  }
  &-reset, &-dropdown {
    border: none;
    height: 100%;
    background: transparent;
    font-size: inherit;
    cursor: pointer;
    outline: none;
  }
  &-reset {
    margin-left: 6px;
  }
  &-dropdown {
    margin: 0 6px;
  }
}

@media screen and (max-width: 564px) {
  .filter {
    &-form {
      justify-content: center;
      > label {
        margin: 0 20px 0 10px;
        & > .icon-filter {
          color: #7d7d7d;
          &.active {
            color: #2faf3e;
          }
        }
      }
    }
    &-info {
    display: none;
    }
  }
}

@media screen and (max-width: 440px) {
  .input {
    &-wrapper {
    flex-grow: 1;
    }
    &-list {
    width: 100%;
    }
  }
  #search {
    width: 100%;
  }
}

.checkbox input {
  display: none;
  &:checked + .checkbox-text {
    background: #9FD468;
    color: white;
  }
}

.checkbox-text {
  padding: 6px 10px;
  cursor: pointer;
  display: block;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  &:hover {
    background: grey;
    color: white;
  }
}
</style>
