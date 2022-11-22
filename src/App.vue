<template>
  <div class="container mx-auto px-8">
    <div class="bg-white my-4 rounded-lg shadow overflow-hidden">
      <div class="bg-gray-700 p-2 border-b border-gray-900">
        <div class="text-base text-gray-100 text-center font-semibold">
          Regra de Johnson
        </div>
      </div>
      <div class="p-6 space-y-8">
        <div>
          <form @submit.prevent="newItem">
            <div class="grid grid-cols-12 gap-6 content-end">
              <div class="col-span-3 max-sm:col-span-2">
                <label for="item" class="block text-sm font-medium text-gray-700">Item</label>
                <div class="mt-1 flex rounded-md shadow-sm">
                  <input type="text" v-model="form.item" @input="form.item = uppercase(form.item)" name="item" id="item" required class="block w-full flex-1 rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="Ex.: A">
                </div>
              </div>
              <div class="col-span-3 max-sm:col-span-2">
                <label for="factoryInMinutes" class="block text-sm font-medium text-gray-700">Fabricação</label>
                <div class="mt-1 flex rounded-md shadow-sm">
                  <input type="number" v-model="form.factoryInMinutes" name="factoryInMinutes" id="factoryInMinutes" required class="block w-full flex-1 rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="Ex.: 10">
                </div>
              </div>
              <div class="col-span-3 max-sm:col-span-2">
                <label for="mountInMinutes" class="block text-sm font-medium text-gray-700">Montagem</label>
                <div class="mt-1 flex rounded-md shadow-sm">
                  <input type="number" v-model="form.mountInMinutes" name="mountInMinutes" id="mountInMinutes" required class="block w-full flex-1 rounded-md border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 sm:text-sm" placeholder="Ex.: 10">
                </div>
              </div>
              <div class="col-span-3 max-sm:col-span-2">
                <div class="flex h-full items-end">
                  <button type="submit" class="justify-center rounded-md border border-transparent bg-indigo-600 py-2 px-4 text-sm font-medium text-white hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-offset-2">
                    Adicionar
                  </button>
                </div>
              </div>
            </div>
          </form>
        </div>

        <div>
          <div v-if="isShowItemsTable" class="overflow-x-auto relative">
            <CommonTable :headers="headersItemsTable" :rows="items">
              <template #body="{ item }: { item: Item }">
                <th scope="row" class="py-4 px-6 font-medium text-gray-900 whitespace-nowrap dark:text-white">
                  {{ item.rank ? item.rank : '-/-' }}
                </th>
                <td class="py-4 px-6">
                  {{ item.item }}
                </td>
                <td class="py-4 px-6">
                  {{ item.factoryInMinutes }}
                </td>
                <td class="py-4 px-6">
                  {{ item.mountInMinutes }}
                </td>
              </template>
            </CommonTable>
          </div>
        </div>

        <div>
          <div v-if="isShowItemsTable" class="flex flex-row-reverse justify-between">
            <div>
              <button @click="cleanItems" type="submit" class="justify-center rounded-md border border-transparent bg-gray-600 py-2 px-4 text-sm font-medium text-white hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2">
                Limpar
              </button>
            </div>
            <div>
              <button @click="calculate" type="submit" class="justify-center rounded-md border border-transparent bg-green-600 py-2 px-4 text-sm font-medium text-white hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2">
                Calcular
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="isShowItemsFactoryTable" class="bg-white w-2/3 my-4 rounded-lg shadow overflow-hidden">
      <div class="bg-gray-700 p-2 border-b border-gray-900">
        <div class="text-base text-gray-100 text-center font-semibold">
          Linha de Fabricação
        </div>
      </div>
      <div class="p-6 space-y-4">
        <CommonTable is-horizontal :headers="headersFactoryTable" :rows="itemsFactory">
          <template #body="{ item }: { item: Item }">
            <th class="py-4 px-6">
              {{ `${item.item}${item.factoryInMinutes}` }}
            </th>
          </template>
        </CommonTable>

        <div class="text-sm">
          Tempo Total de Fabricação:
          <span class="font-medium">
            {{ totalFactoryInMinutes ? totalFactoryInMinutes : 0 }} minutos
          </span>
        </div>
      </div>
    </div>

    <div v-if="isShowItemsMountTable" class="bg-white w-2/3 my-4 rounded-lg shadow overflow-hidden">
      <div class="bg-gray-700 p-2 border-b border-gray-900">
        <div class="text-base text-gray-100 text-center font-semibold">
          Linha de Montagem
        </div>
      </div>
      <div class="p-6 space-y-4">
        <CommonTable is-horizontal :headers="headersMountTable" :rows="itemsMount">
          <template #body="{ item }: { item: Item }">
            <th class="py-4 px-6">
              {{ `${item.item}${item.idlenessInMinutes ? item.idlenessInMinutes : item.mountInMinutes}` }}
            </th>
          </template>
        </CommonTable>

        <div class="text-sm">
          Tempo Total de Montagem:
          <span class="font-medium">
            {{ totalMountInMinutes ? totalMountInMinutes : 0 }} minutos
          </span>
        </div>
        
        <div class="text-sm">
          Tempo Total de Ociosidade:
          <span class="font-medium">
            {{ totalIdlenessInMinutes ? totalIdlenessInMinutes : 0 }} minutos
          </span>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { uppercase } from '@/core/utils'
import type { Item, ItemForm } from '@/core/types'
import CommonTable from '@/core/components/CommonTable.vue'

const IDLENESS_ITEM_NAME = 'OC'
const headersItemsTable = ['Rank', 'Item', 'Fabricação', 'Montagem'];

const form = ref<ItemForm>({ rank: 0 })
const items = ref<Item[]>([])
const itemsMount = ref<Item[]>([])
const itemsFactory = ref<Item[]>([])

const isShowItemsTable = computed(() => !!items.value.length)
const isShowItemsMountTable = computed(() => !!itemsMount.value.length)
const isShowItemsFactoryTable = computed(() => !!itemsFactory.value.length)
const headersMountTable = computed(() => Array.from({ length: itemsMount.value.length }, (v, i) => `${i + 1}º`))
const headersFactoryTable = computed(() => Array.from({ length: itemsFactory.value.length }, (v, i) => `${i + 1}º`))
const totalMountInMinutes = computed(() => itemsMount.value?.reduce((p, c) => p + c.mountInMinutes!, 0))
const totalFactoryInMinutes = computed(() => itemsFactory.value?.reduce((p, c) => p + c.factoryInMinutes!, 0))
const totalIdlenessInMinutes = computed(() => itemsMount.value?.reduce((p, c) => p + (c.idlenessInMinutes || 0), 0))

function newItem() {
  items.value.push(form.value)
  form.value = {}
}

function cleanItems() {
  items.value = []
  itemsMount.value = []
  itemsFactory.value = []
}

function calculate() {
  let valuesFromItems = getUniqueValuesFromItems()
  itemsFactory.value = calculateItemsFactory(valuesFromItems)
  itemsMount.value = calculateItemsMount()
}

function getUniqueValuesFromItems(): number[] {
  let mountValues = getOnlyMountInMinutesFromItems()
  let factoryValues = getOnlyFactoryInMinutesFromItems()
  let valuesFromItems = removeRepeatedValuesFromItems(mountValues, factoryValues)
  return sortAscValuesFromItems(valuesFromItems)
}

function getOnlyMountInMinutesFromItems(): number[] {
  return items.value.map(e => e.mountInMinutes) as number[]
}

function getOnlyFactoryInMinutesFromItems(): number[] {
  return items.value.map(e => e.factoryInMinutes) as number[]
}

function removeRepeatedValuesFromItems(mountValues: number[], factoryValues: number[]): number[] {
  return Array.from(new Set<number>([...mountValues, ...factoryValues]).values())
}

function sortAscValuesFromItems(valuesFromItems: number[]): number[] { 
  valuesFromItems.sort((a, b) => a - b)
  return valuesFromItems
}

function calculateItemsFactory(valuesFromItems: number[]): Item[] {
  let sortedItems: Item[] = []
  let countRankedItemsByMountCriteria = 0
  let countRankedItemsByFactoryCriteria = 0

  valuesFromItems.forEach((value) => {
    let itemsInFactory = filterItemsByFactoryInMinutes(sortedItems, value)
    itemsInFactory.sort((a, b) => a.mountInMinutes! < b.mountInMinutes! ? 1 : -1) 

    itemsInFactory.forEach(item => {
      item.rank = nextRankFactory(countRankedItemsByFactoryCriteria)
      countRankedItemsByFactoryCriteria++
      sortedItems.push(item)
    })

    let itemsInMount = filterItemsByMountInMinutes(sortedItems, value)
    itemsInMount.sort((a, b) => a.factoryInMinutes! < b.factoryInMinutes! ? 1 : -1)

    itemsInMount.forEach(item => {
      item.rank = nextRankMount(countRankedItemsByMountCriteria, items.value)
      countRankedItemsByMountCriteria++
      sortedItems.push(item)
    })
  })

  sortedItems.sort((a, b) => a.rank! - b.rank!)
  setTotalInMinutesInItems(sortedItems)

  return sortedItems
}

function filterItemsByFactoryInMinutes(sortedItems: Item[], value: number): Item[] {
  return items.value.filter(e => e.factoryInMinutes === value && !sortedItems.includes(e))
}

function filterItemsByMountInMinutes(sortedItems: Item[], value: number): Item[] {
  return items.value.filter(e => e.mountInMinutes === value && !sortedItems.includes(e))
}

function setTotalInMinutesInItems(sortedItems: Item[]) {
  sortedItems.forEach((item, i) => item.totalInMinutes = (sortedItems[i - 1]?.totalInMinutes || 0) + item.factoryInMinutes! )
}

function nextRankFactory(countItems: number): number {
  return ++countItems
}

function nextRankMount(countItems: number, items: Item[]): number {
  return items.length - countItems
}

function calculateItemsMount(): Item[] {
  let itemsMount: Item[] = []
  let totalMountInMinutes: number = 0

  itemsFactory.value.forEach((itemFactory) => {
    let isTotalMountIsLowerTotalFactory = totalMountInMinutes < itemFactory.totalInMinutes! 
    if (isTotalMountIsLowerTotalFactory) {
      let idlenessInMinutes = itemFactory.totalInMinutes! - totalMountInMinutes
      totalMountInMinutes += idlenessInMinutes

      let idlenessItem: Item = buildIdlenessItem(itemFactory, itemsMount.length + 1, totalMountInMinutes, idlenessInMinutes)
      itemsMount.push(idlenessItem)
    }

    totalMountInMinutes += itemFactory.mountInMinutes!

    itemsMount.push({...itemFactory, totalInMinutes: totalMountInMinutes})
  })

  return itemsMount
}

function buildIdlenessItem(itemFactory: Item, rank: number, totalInMinutes: number, idlenessInMinutes: number): Item {
  return {
    ...itemFactory,
    rank: rank,
    item: IDLENESS_ITEM_NAME,
    mountInMinutes: idlenessInMinutes,
    totalInMinutes,
    idlenessInMinutes
  };
}
</script>