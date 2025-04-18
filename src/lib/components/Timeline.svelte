<!-- Timeline.svelte -->
<script lang="ts">
  import { onMount } from 'svelte';
  import { slide } from 'svelte/transition';
  import type { TimelineItem } from '../types/timeline';

  // Svelte action: Intersection Observer で表示検知
  function intersection(node: HTMLElement, { onChange }: { onChange: (visible: boolean) => void }) {
    const observer = new IntersectionObserver(([entry]) => onChange(entry.isIntersecting), { threshold: 0.1 });
    observer.observe(node);
    return { destroy() { observer.unobserve(node); } };
  }

  export let items: TimelineItem[] = [];
  // フェーズ1-2: 特定時代フィルタ（デフォルト：ルネサンス）
  export let filterEra: string = 'ルネサンス';
  let filteredItems: TimelineItem[] = [];
  $: filteredItems = items.filter(item => item.era === filterEra);

  // eraごとにグループ化
  $: grouped = (() => {
    if (!filteredItems || filteredItems.length === 0) return [];
    const map = new Map<string, TimelineItem[]>();
    for (const item of filteredItems) {
      const era = item.era || 'その他';
      if (!map.has(era)) map.set(era, []);
      map.get(era)?.push(item);
    }
    return Array.from(map, ([era, items]) => ({ era, items }));
  })();

  // 背景色マップ（必要に応じて追加・変更可）
  const eraBgMap: Record<string, string> = {
    'ルネサンス': 'bg-blue-50',
    'ポスト印象派': 'bg-yellow-50',
    'その他': 'bg-gray-50',
  };

  let currentEra: string = 'その他';
  $: if (grouped && grouped.length > 0) currentEra = grouped[0].era;
  let eraRefs: HTMLElement[] = [];

  // 各アイテムの表示フラグ（Intersection Observer 用）
  let visibility: boolean[][] = [];
  $: visibility = grouped.map(g => g.items.map(() => false));

  let expanded: boolean[][] = [];
  $: expanded = grouped.map(g => g.items.map(() => false));

  onMount(() => {
    if (typeof window === 'undefined') return;
    const onScroll = () => {
      if (!eraRefs || eraRefs.length === 0) return;
      const center = window.innerHeight * 0.35;
      const visible = eraRefs
        .map((el, idx) => ({
          era: grouped[idx]?.era ?? 'その他',
          top: el?.getBoundingClientRect().top ?? Infinity
        }))
        .filter(({ top }) => top < center && top > 0);
      if (visible.length > 0) {
        visible.sort((a, b) => a.top - b.top);
        currentEra = visible[0].era;
      } else {
        currentEra = grouped[0]?.era ?? 'その他';
      }
    };
    window.addEventListener('scroll', onScroll, { passive: true });
    onScroll();
    return () => window.removeEventListener('scroll', onScroll);
  });
</script>

<div class={`w-full max-w-6xl mx-auto px-2 sm:px-4 md:px-8 transition-colors duration-500 ${eraBgMap[currentEra] || 'bg-gray-50'}`}>  
  <div class="relative py-8">
    <!-- timeline vertical line -->
    <div
      class="hidden sm:block absolute left-1/2 h-full w-0.5 bg-gray-200 transform -translate-x-1/2"
    ></div>
    {#each grouped as group, i (group.era)}
      <!-- 時代区分見出し -->
      <div
        bind:this={eraRefs[i]}
        data-era={group.era}
        class="relative z-10 text-center font-bold text-xl sm:text-2xl text-blue-700 mb-8 mt-12 sm:mt-16 tracking-wide"
      >
        {group.era}
      </div>
      {#each group.items as item, j}
        <div
          use:intersection={{ onChange: (v) => visibility[i][j] = v }}
          class="transition-opacity duration-300 ease-in-out"
          class:opacity-0={!visibility[i][j]}
        >
          {#if item.category === '歴史'}
            <div class="flex flex-col sm:flex-row-reverse justify-between mb-8 relative" in:slide={{ duration: 400 }}>
              <!-- インジケーター（丸ドット）: sm以上で中央線上に表示 -->
              <div class="hidden sm:block absolute left-1/2 top-6 -translate-x-1/2 w-4 h-4 bg-blue-500 border-4 border-white rounded-full z-20 shadow"></div>
              <!-- 日付（左側） -->
              <div class="sm:w-32 text-left sm:pl-8 text-gray-600 flex-shrink-0 mb-2 sm:mb-0">
                {item.date}
              </div>
              <!-- コンテンツカード（左側） -->
              <div
                class="flex-1 max-w-full sm:max-w-[calc(50%-4rem)] sm:mr-8 p-4 bg-white rounded-lg shadow-md cursor-pointer"
                on:click={() => expanded[i][j] = !expanded[i][j]}
              >
                <h3 class="text-lg sm:text-xl font-medium text-gray-800 mb-2">{item.title}</h3>
                {#if expanded[i][j]}
                  <p class="text-gray-600 mb-4">{item.description}</p>
                {/if}
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
          {:else}
            <div class="flex flex-col sm:flex-row justify-between mb-8 relative" in:slide={{ duration: 400 }}>
              <!-- インジケーター（丸ドット）: sm以上で中央線上に表示 -->
              <div class="hidden sm:block absolute left-1/2 top-6 -translate-x-1/2 w-4 h-4 bg-blue-500 border-4 border-white rounded-full z-20 shadow"></div>
              <!-- 日付（右側） -->
              <div class="sm:w-32 text-right sm:pr-8 text-gray-600 flex-shrink-0 mb-2 sm:mb-0">
                {item.date}
              </div>
              <!-- コンテンツカード（右側） -->
              <div
                class="flex-1 max-w-full sm:max-w-[calc(50%-4rem)] sm:ml-8 p-4 bg-white rounded-lg shadow-md cursor-pointer"
                on:click={() => expanded[i][j] = !expanded[i][j]}
              >
                <h3 class="text-lg sm:text-xl font-medium text-gray-800 mb-2">{item.title}</h3>
                {#if expanded[i][j]}
                  <p class="text-gray-600 mb-4">{item.description}</p>
                {/if}
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
          {/if}
        </div>
      {/each}
    {/each}
  </div>
</div>
