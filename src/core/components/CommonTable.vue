<template>
  <table class="w-full text-sm text-center text-gray-500 dark:text-gray-400">
    <thead class="text-xs text-gray-700 uppercase bg-gray-100 dark:bg-gray-700 dark:text-gray-400">
      <tr>
        <th scope="col" class="py-3 px-6" v-for="header in headers">
          {{ header }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr v-if="!isHorizonal" class="bg-white border-b dark:bg-gray-800 dark:border-gray-700" v-for="row in rows">
        <slot name="body" :item="row"></slot>
      </tr>
      <tr v-else class="bg-white border-b dark:bg-gray-800 dark:border-gray-700">
        <slot name="body" :item="row" v-for="row in rows"></slot>
      </tr>
    </tbody>
  </table>
</template>

<script setup lang="ts">
import { defineProps } from 'vue'

interface Props {
  isHorizonal?: boolean
  headers: string[]
  rows: any[],
}

defineProps<Props>()
</script>