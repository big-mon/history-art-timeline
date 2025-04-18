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
  // フェーズ2-1: 年代範囲フィルタ（例：476年〜1600年）
  export let startYear: number = 476;
  export let endYear: number = 1600;
  let filteredItems: TimelineItem[] = [];
  $: filteredItems = items.filter(item => {
    const year = parseInt(item.date, 10);
    return !isNaN(year) && year >= startYear && year <= endYear;
  }).sort((a, b) => parseInt(a.date, 10) - parseInt(b.date, 10));

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
    '中世': 'bg-green-50',
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
      const distances = eraRefs.map((el, idx) => {
        const top = el?.getBoundingClientRect().top ?? Infinity;
        return { era: grouped[idx]?.era ?? 'その他', top };
      });
      const closest = distances.reduce((prev, curr) =>
        Math.abs(curr.top - center) < Math.abs(prev.top - center) ? curr : prev
      , distances[0]);
      currentEra = closest.era;
    };
    window.addEventListener('scroll', onScroll, { passive: true });
    onScroll();
    return () => window.removeEventListener('scroll', onScroll);
  });
</script>

<div class={`w-full max-w-6xl mx-auto px-2 sm:px-4 md:px-8 transition-colors duration-500 ${eraBgMap[currentEra] || 'bg-gray-50'}`}>  
  <!-- 画面端タイムラインインジケーター -->
  <div class="hidden sm:block fixed right-0 top-1/2 transform -translate-y-1/2 pr-4 pointer-events-none">
    <ul class="flex flex-col space-y-2">
      {#each grouped as grp}
        <li class="transition-transform duration-300 {currentEra === grp.era ? 'text-blue-700 font-bold scale-110' : 'text-gray-400'}">
          {grp.era}
        </li>
      {/each}
    </ul>
  </div>
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
              <!-- 日付（中央表示） -->
              <div class="hidden sm:block absolute left-1/2 top-6 translate-y-6 -translate-x-1/2 text-gray-600 text-sm font-medium z-20">
                {item.date}
              </div>
              <!-- コンテンツカード（左側） -->
              <div
                role="button"
                tabindex="0"
                on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); expanded[i][j] = !expanded[i][j]; } }}
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
                {#if item.credit}
                  <p class="text-xs text-gray-500 mb-4">{item.credit}</p>
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
              <!-- 日付（中央表示） -->
              <div class="hidden sm:block absolute left-1/2 top-6 translate-y-6 -translate-x-1/2 text-gray-600 text-sm font-medium z-20">
                {item.date}
              </div>
              <!-- コンテンツカード（右側） -->
              <div
                role="button"
                tabindex="0"
                on:keydown={(e) => { if (e.key === 'Enter' || e.key === ' ') { e.preventDefault(); expanded[i][j] = !expanded[i][j]; } }}
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
                {#if item.credit}
                  <p class="text-xs text-gray-500 mb-4">{item.credit}</p>
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
