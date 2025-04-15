<script lang="ts">
  import Timeline from '$lib/components/Timeline.svelte';
  import { timelineData } from '$lib/data/sampleData';
</script>

<main class="w-full min-h-screen bg-gray-50">
  <h1 class="text-2xl sm:text-3xl md:text-4xl font-bold text-gray-800 text-center py-8">
    芸術作品の歴史タイムライン
  </h1>
  <Timeline items={timelineData} />
</main>
