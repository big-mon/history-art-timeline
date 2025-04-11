<!-- Timeline.svelte -->
<script lang="ts">
  import { Timeline } from 'svelte-timeline';
  import type { TimelineItem } from '../types/timeline';

  export let items: TimelineItem[] = [];

  const events = items.map(item => ({
    title: item.title,
    date: item.date,
    content: `
      <div class="timeline-item">
        <h3>${item.title}</h3>
        <p>${item.description}</p>
        ${item.image ? `<img src="${item.image}" alt="${item.title}" style="max-width: 200px;">` : ''}
        ${item.category ? `<span class="category">${item.category}</span>` : ''}
      </div>
    `
  }));
</script>

<div class="timeline-container">
  <Timeline {events} />
</div>

<style>
  .timeline-container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  :global(.timeline-item) {
    padding: 1rem;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  :global(.timeline-item h3) {
    margin: 0 0 0.5rem;
    color: #333;
  }

  :global(.timeline-item p) {
    margin: 0 0 1rem;
    color: #666;
  }

  :global(.timeline-item img) {
    border-radius: 4px;
    margin-bottom: 1rem;
  }

  :global(.timeline-item .category) {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    background: #f0f0f0;
    border-radius: 4px;
    font-size: 0.875rem;
    color: #666;
  }
</style>
