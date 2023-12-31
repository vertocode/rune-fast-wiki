<template>
  <div class="general-market-items">
    <div class="title-wrapper">
      <button class="back-button" @click="goBack">
        <svg
            xmlns="http://www.w3.org/2000/svg"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            class="back-icon"
        >
          <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M15 19l-7-7 7-7"
          />
        </svg>
        Back
      </button>
      <h1>All items according to the "{{ selectedCategory }}" category:</h1>
    </div>
    <LoadingSpinner v-if="isLoading"/>
    <div class="list-content" v-else>
      <ErrorToast v-if="showError"/>
      <BaseFilter label="Search by item name or ID" @input:value="filterItems"/>
      <div class="list" v-if="filteredItems?.length">
        <ItemCard
            v-for="(item, index) in filteredItems"
            :item="item"
            :key="index"
            @open-modal="selectedItem = item"
        />
      </div>
      <NoCategory title-type="items" v-else />
    </div>
    <ItemModal @close="selectedItem = {}" :item="selectedItem" v-if="selectedItem?.name"/>
  </div>
</template>

<script setup lang="ts">

import { useGeneralMarketItems } from "@/stores/useGeneralMarketItems"
import ItemCard from "@/components/GeneralMarket/ItemCard.vue"
import BaseFilter from "@/components/Input/BaseFilter.vue"
import NoCategory from "@/components/GeneralMarket/NoCategory.vue"
import {computed, onBeforeMount, ref} from "vue"
import { useRoute, useRouter } from "vue-router"
import LoadingSpinner from "@/components/Spinner/LoadingSpinner.vue"
import ItemModal from "@/components/Modal/ItemModal.vue"
import type { RouteParamValue } from 'vue-router'
import type { Ref } from "vue"
import ErrorToast from "@/components/Toast/ErrorToast.vue";

const store = useGeneralMarketItems()
const route = useRoute()
const router = useRouter()
const id: string | RouteParamValue[] = route.params.id
const items = ref([])
const filteredItems = ref([])
const isLoading = ref(true)
const selectedItem: any = ref({})
const showError = ref(false)

const selectedCategory = computed(() => {
  const [category] = store.state.allCategories.filter(cateogory => cateogory.id === Number(id))
  if (!category?.label) {
    goBack()
  }
  return category?.label || goBack()
})

const filterItems = (filterValue: string | Ref<string> | any) => {
  if (!filterValue) {
    filteredItems.value = items.value
  }
  filteredItems.value = items.value.slice().filter((item: any) => item.name.toUpperCase().includes(filterValue.toUpperCase()) || String(item.id).includes(filterValue))
}

const goBack = () => {
  router.push('/general-market')
}

onBeforeMount(async () => {
  try {
    if (!store.state.filteredCategories.length) {
      await store.getCategories()
    }
    const responseItems = await store.getItemsByCategory(id)
    items.value = responseItems.filter((item: any) => item)
    filteredItems.value = items.value
  } catch (e) {
    showError.value = true
    console.error(e)
  }
  isLoading.value = false
})


</script>

<style lang="scss">
.general-market-items {
  background: linear-gradient(135deg, #222, #646464);
  color: white;

  .title-wrapper {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 1em;

    .back-button {
      display: flex;
      align-items: center;
      background-color: #f1be2d;
      color: white;
      border: none;
      border-radius: 4px;
      padding: 5px 10px;
      font-size: 14px;
      cursor: pointer;
      transition: background-color 0.3s ease;

      .back-icon {
        width: 1.2em;
        height: 1.2em;
        margin-right: 0.5em;
      }
    }

    .back-button:hover {
      background-color: #e69915;
    }

    h1 {
      text-align: center;
      margin-left: 1em;
      font-size: 2.2rem;
      color: #f1be2d;
    }
  }

  .list-content {
    padding: 2em;

    .base-filter {
      width: 60%;
      padding-bottom: 2em;
      margin: auto;

      input {
        background-color: transparent;
        border: 2px solid #f1be2d;
        border-radius: 5px;
        color: white;
        font-size: 1rem;
        padding: 10px;
        width: 100%;
        transition: border-color 0.3s ease;

        &:focus {
          outline: none;
          border-color: #e69915;
        }
      }
    }

    .list {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
      gap: 2em;
    }
  }
}
</style>
