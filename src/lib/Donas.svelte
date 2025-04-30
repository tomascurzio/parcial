<script>
  import * as d3 from "d3"

  export let numbers = []
  export let title = ""

  function arcGenerator({startAngle, endAngle, innerRadius, outerRadius}) {
    return d3.arc()({
      startAngle,
      endAngle,
      innerRadius,
      outerRadius,
    })
  }
</script>

<h3 class="headline">{title}</h3>
<div class="donas_container">
  {#each numbers as n}
    <div class="donas_wrapper">
    <svg width="100" height="100">
      <path
        d={arcGenerator({
          startAngle: 0,
          endAngle: 2 * Math.PI,
          innerRadius: 25,
          outerRadius: 50,
        })}
        fill="#ccc"
        transform="translate(50, 50)"
      />
      <path
        d={arcGenerator({
          startAngle: 0,
          endAngle: (n / 100) * 2 * Math.PI,
          innerRadius: 25,
          outerRadius: 50,
        })}
        fill="red"
        transform="translate(50, 50)"
      />
    </svg>
    <p class="number">{n}</p>
  </div>
  {/each}
</div>

<style>
  .donas_container {
    display: flex;
    gap: 20px;
  }
  .number {
    text-align: center;
    margin-top: 10px;
  }/* svg {
    margin: 10px;
  }
   */
</style>
