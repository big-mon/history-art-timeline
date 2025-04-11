<script lang="ts">
  import Timeline from '$lib/components/Timeline.svelte';
  import { timelineData } from '$lib/data/sampleData';
</script>

<main>
  <h1>芸術作品の歴史タイムライン</h1>
  <Timeline items={timelineData} />
</main>

<style>
  main {
    width: 100%;
    min-height: 100vh;
    background-color: #f5f5f5;
  }

  h1 {
    text-align: center;
    padding: 2rem 0;
    color: #333;
    font-size: 2rem;
  }
</style>
