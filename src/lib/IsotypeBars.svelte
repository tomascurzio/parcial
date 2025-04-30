<script>
  import * as d3 from "d3"
  export let title = ""
  export let numbers = []
  let numbersDivided5 = numbers.map(n => Math.floor(n / 5))
</script>

<h3 class="headline">{title}</h3>
<div class="iso__container-iso">
  {#each numbersDivided5 as n, index}
    <div class="row">
      {#each Array(n) as m}
        <img
          style="height: 50px; padding: 2px"
          src="./images/person.png"
          alt="person"
        />
      {/each}
      <p>{numbers[index]}</p>
    </div>
  {/each}
</div>

<style>
  .iso__container-iso {
    display: flex;
    flex-direction: column;
    max-width: 1000px;
    align-items: start;
    justify-content: space-around;
  }
</style>
