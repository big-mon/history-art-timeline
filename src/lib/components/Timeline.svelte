<!-- Timeline.svelte -->
<script lang="ts">
  import type { TimelineItem } from '../types/timeline';

  export let items: TimelineItem[] = [];
</script>

<div class="timeline-container">
  <div class="timeline">
    {#each items as item}
      <div class="timeline-item">
        <div class="timeline-date">{item.date}</div>
        <div class="timeline-content">
          <h3>{item.title}</h3>
          <p>{item.description}</p>
          {#if item.image}
            <img src={item.image} alt={item.title} />
          {/if}
          {#if item.category}
            <span class="category">{item.category}</span>
          {/if}
        </div>
      </div>
    {/each}
  </div>
</div>

<style>
  .timeline-container {
    width: 100%;
    max-width: 1200px;
    margin: 0 auto;
    padding: 2rem;
  }

  .timeline {
    position: relative;
    padding: 2rem 0;
  }

  .timeline::before {
    content: '';
    position: absolute;
    left: 50%;
    transform: translateX(-50%);
    width: 2px;
    height: 100%;
    background-color: #e0e0e0;
  }

  .timeline-item {
    display: flex;
    justify-content: space-between;
    margin-bottom: 2rem;
    position: relative;
  }

  .timeline-date {
    width: 120px;
    text-align: right;
    padding-right: 2rem;
    color: #666;
  }

  .timeline-content {
    flex: 1;
    max-width: calc(50% - 60px);
    margin-left: 2rem;
    padding: 1rem;
    background: #fff;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .timeline-content h3 {
    margin: 0 0 0.5rem;
    color: #333;
  }

  .timeline-content p {
    margin: 0 0 1rem;
    color: #666;
  }

  .timeline-content img {
    max-width: 100%;
    border-radius: 4px;
    margin-bottom: 1rem;
  }

  .category {
    display: inline-block;
    padding: 0.25rem 0.5rem;
    background: #f0f0f0;
    border-radius: 4px;
    font-size: 0.875rem;
    color: #666;
  }
</style>
