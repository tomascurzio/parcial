<script>
  import * as d3 from "d3"

  export let title = ""
  export let numbers = ""

  /* 
    d3.scaleLinear() retorna una función,
    la guardamos en alturaColCoffe
    para poder usarla en el template
  */
 let alturaColCoffe = d3.scaleLinear()
 .domain([0, d3.max(numbers)])
 .range([0, 100])
 
 /* 
   d3.scaleLinear() puede ser usada para
   asignar colores a los elementos,
   la guardamos en color
   para poder usarla en el template
 */
  let color =  d3.scaleLinear()
      .domain([24, 98]) 
      .range(["yellow", "red"])

</script>

<h3 class="headline">{title}</h3>

<div class="coffes-container">
  {#each numbers as n}
  <div>
    <div class="column-wrapper">
      <div class="column-coffe" style="height: {alturaColCoffe(n)}%; background-color: {color(n)}"></div>
      <img class="coffe" src="./images/coffe-cup-mask.svg" alt="">
    </div>
    <p class="number">{n}</p>
  </div>
  {/each}
</div>

<style>
  .coffes-container {
    display: flex;
  }
  .column-wrapper {
    position: relative;
    width: 160px;
    height: 200px;
    /* background-color: #ccc;
    border: black 1px solid; */
  }
  .column-coffe {
    position: absolute;
    height: 50%;
    left: 10%;
    right: 10%;
    bottom: 0;
    /* width: 100%; */
    background-color: red;
  }
  .coffe {
    position: absolute;
    bottom: -10px;
    width: 100%;
  }
  .number {
    text-align: center;
  }
</style>
