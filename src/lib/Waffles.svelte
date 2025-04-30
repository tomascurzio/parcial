<script>
  import * as d3 from "d3"
  export let numbers = []
  export let title = ""
</script>

<h3 class="headline">{title}</h3>
<div class="waffles_container">
  {#each numbers as n}

  <div class="waffle_wrapper">
    <div class="waffle">
      {#each d3.range(0, 100) as dot}
        {#if dot < n}
          <div class="cuad lleno"></div>
        {:else}
          <div class="cuad vacio"></div>
        {/if}
      {/each}
    </div>
    <p class="number">{n}%</p>
  </div>

  {/each}
</div>

<style>
  .waffles_container {
    display: flex;
    align-items: center;
    gap: 10px;
  }

  .waffle {
    display: flex;
    flex-wrap: wrap;
    width: 118px; /* 100 cuadros de 10px + 2px * 9 de gap */
    height: 118px;
    gap: 2px;
  }
  .cuad {
    width: 10px;
    height: 10px;
  }
  .vacio {
    background-color: #ddd;
  }
  .lleno {
    background-color: red;
  }
  .number {
    text-align: center;
    margin-top: 10px;
  }
</style>
