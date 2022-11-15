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
              <button @click="cleanLogs" type="submit" class="justify-center rounded-md border border-transparent bg-gray-600 py-2 px-4 text-sm font-medium text-white hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500 focus:ring-offset-2">
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

const isShowItemsTable = computed(() => items.value.length > 0)
const isShowItemsMountTable = computed(() => itemsMount.value.length > 0)
const isShowItemsFactoryTable = computed(() => itemsFactory.value.length > 0)
const headersMountTable = computed(() => Array.from({ length: itemsMount.value.length }, (v, i) => `${i + 1}º`))
const headersFactoryTable = computed(() => Array.from({ length: itemsFactory.value.length }, (v, i) => `${i + 1}º`))
const totalFactoryInMinutes = computed(() => itemsFactory.value?.reduce((p, c) => p + c.factoryInMinutes!, 0))
const totalMountInMinutes = computed(() => itemsMount.value?.reduce((p, c) => p + c.mountInMinutes!, 0))
const totalIdlenessInMinutes = computed(() => itemsMount.value?.reduce((p, c) => p + (c.idlenessInMinutes || 0), 0))

function newItem() {
  items.value.push(form.value)
  form.value = {}
}

function cleanLogs() {
  items.value = []
  itemsMount.value = []
  itemsFactory.value = []
}

function calculate() {
  itemsFactory.value = calculateItemsFactory()
  itemsMount.value = calculateItemsMount()
}

function calculateItemsFactory(): Item[] {
  let auxItems: Item[] = []
  let valuesFromItems = sortValuesFromItems()
  let countRankedItemsInMount = 0
  let countRankedItemsInFactory = 0

  valuesFromItems.forEach((value) => {
    let itemsInFactory = items.value.filter(e => e.factoryInMinutes === value && !auxItems.includes(e))
    if (itemsInFactory.length >= 2) {
      itemsInFactory.sort((a, b) => a.mountInMinutes! < b.mountInMinutes! ? 1 : -1) 
    }

    itemsInFactory.forEach(item => {
      item.rank = nextRankFactory(countRankedItemsInFactory)
      countRankedItemsInFactory++
      auxItems.push(item)
    })

    let itemsInMount = items.value.filter(e => e.mountInMinutes === value && !auxItems.includes(e))
    if (itemsInMount.length >= 2) {
      itemsInMount.sort((a, b) => a.factoryInMinutes! < b.factoryInMinutes! ? 1 : -1)
    }

    itemsInMount.forEach(item => {
      item.rank = nextRankMount(countRankedItemsInMount, items.value)
      countRankedItemsInMount++
      auxItems.push(item)
    })
  })

  auxItems.sort((a, b) => a.rank! - b.rank!)
  auxItems.forEach((item, i) => item.totalInMinutes = (auxItems[i - 1]?.totalInMinutes || 0) + item.factoryInMinutes! )

  return auxItems
}

function sortValuesFromItems(): number[] {
  let mountValues = items.value.map(e => e.mountInMinutes) as number[]
  let factoryValues = items.value.map(e => e.factoryInMinutes) as number[]
  let valuesFromItems = Array.from(new Set<number>([...mountValues, ...factoryValues]).values())
  valuesFromItems.sort((a, b) => a - b)
  return valuesFromItems
}

function nextRankFactory(countItems: number): number {
  return ++countItems
}

function nextRankMount(countItems: number, items: Item[]): number {
  return items.length - countItems
}

function calculateItemsMount(): Item[] {
  let auxItemsMount: Item[] = []
  let totalMountInMinutes: number = 0

  itemsFactory.value.forEach((itemFactory, i) => {
    if (totalMountInMinutes < itemFactory.totalInMinutes!) {
      let idlenessInMinutes = itemFactory.totalInMinutes! - totalMountInMinutes
      totalMountInMinutes += idlenessInMinutes

      let idlenessItem: Item = buildIdlenessItem(itemFactory, auxItemsMount.length + 1, totalMountInMinutes, idlenessInMinutes)
      auxItemsMount.push(idlenessItem)
    }

    totalMountInMinutes += itemFactory.mountInMinutes!

    auxItemsMount.push({...itemFactory, totalInMinutes: totalMountInMinutes})
  })

  return auxItemsMount
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