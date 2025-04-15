<!-- Timeline.svelte -->
<script lang="ts">
  import type { TimelineItem } from '../types/timeline';
  import { slide } from 'svelte/transition';

  export let items: TimelineItem[] = [];
</script>

<div class="w-full max-w-6xl mx-auto px-2 sm:px-4 md:px-8">
  <div class="relative py-8">
    <!-- timeline vertical line -->
    <div
      class="hidden sm:block absolute left-1/2 h-full w-0.5 bg-gray-200 transform -translate-x-1/2"
    ></div>
    {#each items as item}
      <div class="flex flex-col sm:flex-row justify-between mb-8 relative" transition:slide={{ duration: 400 }}>
        <!-- インジケーター（丸ドット）: sm以上で中央線上に表示 -->
        <div class="hidden sm:block absolute left-1/2 top-6 -translate-x-1/2 w-4 h-4 bg-blue-500 border-4 border-white rounded-full z-20 shadow"></div>
        <!-- 日付 -->
        <div class="sm:w-32 text-right sm:pr-8 text-gray-600 flex-shrink-0 mb-2 sm:mb-0">
          {item.date}
        </div>
        <!-- コンテンツカード -->
        <div
          class="flex-1 max-w-full sm:max-w-[calc(50%-4rem)] sm:ml-8 p-4 bg-white rounded-lg shadow-md"
        >
          <h3 class="text-lg sm:text-xl font-medium text-gray-800 mb-2">{item.title}</h3>
          <p class="text-gray-600 mb-4">{item.description}</p>
          {#if item.image}
            <img src={item.image} alt={item.title} class="w-full rounded-md mb-4" />
          {/if}
          {#if item.category}
            <span class="inline-block px-2 py-1 bg-gray-100 rounded text-sm text-gray-600"
              >{item.category}</span
            >
          {/if}
        </div>
      </div>
    {/each}
  </div>
</div>
