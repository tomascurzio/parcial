<script>
  import {onMount} from "svelte"
  import * as d3 from "d3"

  export let numbers = []
  export let title = ""
  let pathLength
  let largo

  onMount(() => {
    // @ts-ignore
    // longitud total del path
    pathLength = document.querySelector("#path").getTotalLength()
    // longitud total del path
    let maxGap = pathLength - (pathLength * d3.min(numbers)) / d3.max(numbers)
    let minGap = 0

    largo =  d3
        .scaleLinear()
        .domain(d3.extent(numbers)) // extent returns [min, max]
        .range([maxGap, minGap])
  })
</script>

{#if numbers.length > 0}
<h3 class="headline">{title}</h3>
<div class="hearts-container">
    {#each numbers as n}
      <div>
        <svg width="100" height="100">
          <!-- Corazón gris -->
          <path
            id="path"
            d="M 10,30
  A 20,20 0,0,1 50,30
  A 20,20 0,0,1 90,30
  Q 90,60 50,90
  Q 10,60 10,30 z"
            fill="none"
            stroke="#ccc"
            stroke-width="10"
          />
          <!-- Corazón rojo -->
          {#if pathLength}
            <path
              id="path"
              d="M 10,30
    A 20,20 0,0,1 50,30
    A 20,20 0,0,1 90,30
    Q 90,60 50,90
    Q 10,60 10,30 z"
              fill="none"
              stroke="red"
              stroke-width="10"
              stroke-dasharray={pathLength}
              stroke-dashoffset={largo(n)}
            />
          {/if}
        </svg>
        <p class="number">{n}</p>
      </div>
    {/each}
  </div>
  {/if}

<style>
  .hearts-container {
    display: flex;
    /* justify-content: center;
    align-items: center; */
  }
  .number {
    text-align: center;
  }
</style>
