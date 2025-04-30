<script>
  import "./app.css";
  import "/public/css/main.css";
  import "./main.js";
  import * as d3 from "d3";
  import { onMount } from 'svelte';

  // Primer grupo de jugadores
  let jugadoresData = [
    { src: "./images/messi71.svg", alturaReal: 24, color: '#de2c35', posicion: 'Extremo Izquierdo', nombre: 'Lionel Andrés Messi' },
    { src: "./images/maxi.svg", alturaReal: 33, color: '#de2c35', posicion: 'Extremo Derecho', nombre: 'Maximiliano Rodríguez' },
    { src: "./images/alejotabares.svg", alturaReal: 42, color: '#de2c35', posicion: 'Lateral Derecho', nombre: 'Marcelo Bielsa' },
    { src: "./images/miltoncasco.svg", alturaReal: 54, color: '#de2c35', posicion: 'Lateral Izquierdo', nombre: 'Milton Casco' },
    { src: "./images/keylor.svg", alturaReal: 63, color: '#de2c35', posicion: 'Arquero', nombre: 'Keylor Navas' },
    { src: "./images/heinze.svg", alturaReal: 71, color: '#de2c35', posicion: 'Defensor Central', nombre: 'Gabriel Heinze' },
    { src: "./images/maradona.svg", alturaReal: 77, color: '#de2c35', posicion: 'Mediocampista Central', nombre: 'Diego Armando Maradona' },
    { src: "./images/scocco.svg", alturaReal: 87, color: '#de2c35', posicion: 'Delantero', nombre: 'Ignacio Scocco' },
    { src: "./images/pabloperez.svg", alturaReal: 92, color: '#de2c35', posicion: 'Mediocampista Derecho', nombre: 'Pablo Perez' },
    { src: "./images/sensini.svg", alturaReal: 98, color: '#de2c35', posicion: 'Mediocampista Izquierdo', nombre: 'Américo Gallego' }
  ];

  // Segundo grupo de jugadores
  let jugadoresData2 = [
    { src: "./images/danilo_gerlo.svg", alturaReal: 23, color: '#F59E0B', posicion: 'Extremo Izquierdo', nombre: 'Santiago López' },
    { src: "./images/dimaria.svg", alturaReal: 35, color: '#F59E0B', posicion: 'Extremo Derecho', nombre: 'Ángel Di Maria' },
    { src: "./images/hernan_diaz.svg", alturaReal: 45, color: '#F59E0B', posicion: 'Lateral Derecho', nombre: 'Hernán Díaz' },
    { src: "./images/lautaroblanco.svg", alturaReal: 56, color: '#F59E0B', posicion: 'Lateral Izquierdo', nombre: 'Lautaro Blanco' },
    { src: "./images/jorgebroun.svg", alturaReal: 50, color: '#F59E0B', posicion: 'Arquero', nombre: 'Jorge Broun' },
    { src: "./images/edgardo_bauza.svg", alturaReal: 43, color: '#F59E0B', posicion: 'Defensor Central', nombre: 'Edgardo Bauza' },
    { src: "./images/locelso77.svg", alturaReal: 38, color: '#F59E0B', posicion: 'Mediocampista Central', nombre: 'Giovani Lo Celso' },
    { src: "./images/ruben.svg", alturaReal: 32, color: '#F59E0B', posicion: 'Delantero', nombre: 'Marcos Ruben' },
    { src: "./images/coudet.svg", alturaReal: 28, color: '#F59E0B', posicion: 'Mediocampista Derecho', nombre: 'Eduardo Coudet' },
    { src: "./images/gino_infantino.svg", alturaReal: 25, color: '#F59E0B', posicion: 'Mediocampista Izquierdo', nombre: 'Gino Infantino' }
  ];

  // Nueva paleta de colores para los jugadores (moderna, más armoniosa)
  const actualizarColores = () => {
    const nuevaPaleta = [
      '#6366F1', // Indigo
      '#8B5CF6', // Violet
      '#EC4899', // Pink
      '#14B8A6', // Teal
      '#2DD4BF', // Turquoise
      '#F59E0B', // Amber
      '#10B981', // Emerald
      '#6366F1', // Indigo
      '#8B5CF6', // Violet
      '#F43F5E'  // Rose
    ];
    
    // Actualiza los colores pero mantiene las imágenes intactas
    jugadoresData = jugadoresData.map((jugador, index) => {
      return { ...jugador, color: nuevaPaleta[index] };
    });
    
    jugadoresData2 = jugadoresData2.map((jugador, index) => {
      return { ...jugador, color: nuevaPaleta[index] };
    });
  };

  // Combinar ambos grupos para la tabla
  let todosJugadores = [...jugadoresData, ...jugadoresData2];

  // Ajustar la escala de altura para que las siluetas sean más grandes
  let scaleHeight = d3.scaleLinear().domain([24, 100]).range([170, 350]);
  
  // Aumentar espaciado entre jugadores para evitar superposición completamente
  let spacing = 170; // Espacio entre jugadores
  
  let jugadorActivo = null; // jugador actualmente seleccionado por hover
  
  // Calcular el ancho total necesario para todos los jugadores
  let maxJugadores = Math.max(jugadoresData.length, jugadoresData2.length);
  
  // Ancho del SVG (ajustado para mejor centrado)
  let svgWidth = maxJugadores * spacing;
  
  // Asegurar un ancho mínimo
  svgWidth = Math.max(svgWidth, 1900);
  
  // Altura para el SVG 
  let svgHeight = 850;

// Configuración para ejes
let alturaDominio = [0, 2.00]; // Rango de altura en metros para el eje Y
let scaleY = d3.scaleLinear().domain(alturaDominio).range([400, 50]);

// Crear ticks para el eje Y (alturas) - solo 9 líneas, con texto en la primera y última
let yTicks = d3.range(alturaDominio[0], alturaDominio[1] + 0.01, alturaDominio[1] / 8)
    .map(d => ({
      value: parseFloat(d.toFixed(2)),
      showText: d === 0 || Math.abs(d - alturaDominio[1]) < 0.01 // Solo mostrar texto para 0 y 2
    }));
// Función de ayuda para convertir altura real a metros
function convertirAMetros(alturaReal) {
    return (alturaReal + 100) / 100;
}
  
  // Función para calcular el centrado de las siluetas
  function calcularOffsetX(numJugadores) {
    // Esta función calcula el desplazamiento necesario para centrar perfectamente
    // el grupo de jugadores en el SVG
    return (svgWidth - (numJugadores * spacing)) / 2;
  }
</script>

<header class="dashboard-header">
  <div class="header-container">
    <h1> Alturas del Clásico Rosarino</h1>
    <p>Visualización comparativa entre diferentes jugadores según su posición en la cancha</p>
  </div>
</header>

<main id="container">
  <div class="grafico">
    <div class="visualization-container">
      <svg width="100%" height= 700 style="overflow: visible" viewBox={`0 0 ${svgWidth} ${svgHeight}`} preserveAspectRatio="xMidYMid meet">
        <!-- Eje Y para el primer grupo (Newell's) -->
        <g class="eje-y" transform={`translate(${calcularOffsetX(jugadoresData.length) - 50}, 30)`}>
          {#each yTicks as tick}
            <line 
              x1="-10" 
              y1={scaleY(tick.value)} 
              x2={jugadoresData.length * spacing + 70} 
              y2={scaleY(tick.value)} 
              stroke="#4B5563" 
              stroke-width="1" 
              stroke-dasharray="3,3" 
            />
            {#if tick.showText}
            <text 
              x="-15" 
              y={scaleY(tick.value)} 
              fill="#94A3B8" 
              font-size="12" 
              text-anchor="end" 
              dominant-baseline="middle"
            >
              {tick.value}m
            </text>
            {/if}
          {/each}
        </g>
        
        
        <text 
          x={svgWidth / 2} 
          y="40"
          fill="#de2c35" 
          font-size="48" 
          font-family="Poppins, sans-serif"
          font-weight="bold"
          text-anchor="middle"
          class="equipo-titulo"
          >
        Newell's Old Boys 
        </text>
        
        <!-- Primer grupo de jugadores - Centrado horizontalmente -->
        <g class="jugadores" transform={`translate(${calcularOffsetX(jugadoresData.length)}, 30)`}> 
          {#each jugadoresData as jugador, index}
            <g 
              class="jugador-container"
              role="group"
              on:mouseenter={() => jugadorActivo = jugador}
              on:mouseleave={() => jugadorActivo = null}
            >
            <line 
            x1={-60}  
            y1={400 - scaleHeight(jugador.alturaReal)} 
            x2={(index * spacing) + 60} 
            y2={400 - scaleHeight(jugador.alturaReal)} 
            class="dashed-line" 
            style="stroke: {jugador.color}; stroke-dasharray: 5,5; stroke-width: 2;" 
          />
                                  <image 
                href={jugador.src} 
                x={(index * spacing) + 30} 
                y={400 - scaleHeight(jugador.alturaReal)} 
                height={scaleHeight(jugador.alturaReal)}
                class="silueta" 
                style="--color: {jugador.color};" 
              />
              <text 
                class="altura-text" 
                x={(index * spacing) + 60} 
                y={390 - scaleHeight(jugador.alturaReal)} 
                fill={jugador.color} 
                font-size="16" 
                font-family="Poppins, sans-serif"
                font-weight="bold"
              >
                {(jugador.alturaReal + 100) / 100} m
              </text>
            </g>
          {/each}
        </g>

          <!-- Eje Y para el segundo grupo (Central) -->
          <g class="eje-y" transform={`translate(${calcularOffsetX(jugadoresData2.length) - 50}, 530)`}>
            {#each yTicks as tick}
              <line 
                x1="-10" 
                y1={scaleY(tick.value)} 
                x2={jugadoresData2.length * spacing + 70} 
                y2={scaleY(tick.value)} 
                stroke="#4B5563" 
                stroke-width="1" 
                stroke-dasharray="3,3" 
              />
              {#if tick.showText}
              <text 
                x="-15" 
                y={scaleY(tick.value)} 
                fill="#94A3B8" 
                font-size="12" 
                text-anchor="end" 
                dominant-baseline="middle"
              >
                {tick.value}m
              </text>
              {/if}
            {/each}
          </g>
          
        <text 
          x={svgWidth / 2} 
          y="540" 
          fill="#F59E0B" 
          font-size="48" 
          font-family="Poppins, sans-serif"
          font-weight="bold"
          text-anchor="middle"
          class="equipo-titulo"
        >
        Rosario Central
        </text>
        
        <g class="jugadores-grupo2" transform={`translate(${calcularOffsetX(jugadoresData2.length)}, 530)`}>          {#each jugadoresData2 as jugador, index}
            <g 
              class="jugador-container"
              role="group"
              on:mouseenter={() => jugadorActivo = jugador}
              on:mouseleave={() => jugadorActivo = null}
            >
            <line 
            x1={-60}
            y1={400 - scaleHeight(jugador.alturaReal)} 
            x2={(index * spacing) + 60} 
            y2={400 - scaleHeight(jugador.alturaReal)} 
            class="dashed-line" 
            style="stroke: {jugador.color}; stroke-dasharray: 5,5; stroke-width: 2;" 
          />
                                  <image 
                href={jugador.src} 
                x={(index * spacing) + 30} 
                y={400 - scaleHeight(jugador.alturaReal)} 
                height={scaleHeight(jugador.alturaReal)}
                class="silueta" 
                style="--color: {jugador.color};" 
              />
              <text 
                class="altura-text" 
                x={(index * spacing) + 60} 
                y={390 - scaleHeight(jugador.alturaReal)} 
                fill={jugador.color} 
                font-size="16" 
                font-family="Poppins, sans-serif"
                font-weight="bold"
              >
                {(jugador.alturaReal + 100) / 100} m
              </text>
            </g>
          {/each}
        </g>
      </svg>
    </div>
  </div>

  {#if jugadorActivo}
    <div class="tooltip">
      <h4>{jugadorActivo.nombre}</h4>
      <p>Altura: {((jugadorActivo.alturaReal + 100) / 100).toFixed(2)} m</p>
      <p>Posición: {jugadorActivo.posicion !== '-' ? jugadorActivo.posicion : 'Sin posición'}</p>
    </div>
  {/if}

  <div class="narrativa">
    <p>
      Esta visualización compara la altura de diferentes tipos de jugadores de fútbol, desde niños hasta profesionales que ocupan diversas posiciones dentro del campo. Se puede observar que los defensores y arqueros tienden a ser más altos, lo que les otorga ciertas ventajas físicas en sus roles. En cambio, los delanteros y mediocampistas presentan una mayor variedad de estaturas. Esto refleja que cada rol en el campo tiene exigencias físicas distintas, y demuestra que el talento y el rendimiento pueden encontrarse en jugadores de cualquier altura. La adición de un segundo grupo de comparación permite analizar estas tendencias con una muestra más amplia y observar similitudes y diferencias entre distintos equipos o ligas. Esto proporciona una perspectiva más enriquecedora sobre la diversidad física presente en el fútbol moderno.
    </p>
  </div>

  <div class="graficos-container">
    <!-- Gráfico 1 -->
    <div class="chart-section">
      <h2 class="chart-title">Diferencia de Altura con respecto al Promedio</h2>
      <p class="chart-description">
        Visualización comparativa de las alturas promedio de jugadores de Newell’s, Central y el promedio general en cada posición. Se evidencian patrones claros según la función táctica: los arqueros y defensores centrales presentan mayor estatura, mientras que extremos y mediocampistas tienden a ser más bajos.
      </p>
      <div class="chart-wrapper">
        <div class="flourish-embed" data-src="story/3070642"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/story/3070642/thumbnail" width="100%" alt="visualization" /></noscript></div>
          </div> 
        </div> 
    <!-- Gráfico 2 -->
    <div class="chart-section">
      <h2 class="chart-title">Comparación de Altura por Posición</h2>
      <p class="chart-description">
        Este gráfico de barras horizontales muestra la altura en centímetros de los jugadores de Newell's y Central según sus posiciones en el campo. Se observa que Newell's tiene jugadores notablemente más altos en posiciones ofensivas y de mediocampo, mientras que las estaturas son más parejas en posiciones defensivas y laterales.      </p>
      <p></p>

      <div class="chart-wrapper">
        <iframe title="" aria-label="Gráfico de bala" id="datawrapper-chart-6kdGs" src="https://datawrapper.dwcdn.net/6kdGs/1/" scrolling="no" frameborder="0" style="width: 0; min-width: 100% !important; border: none;" height="387" data-external="1"></iframe><script type="text/javascript">!function(){"use strict";window.addEventListener("message",(function(a){if(void 0!==a.data["datawrapper-height"]){var e=document.querySelectorAll("iframe");for(var t in a.data["datawrapper-height"])for(var r,i=0;r=e[i];i++)if(r.contentWindow===a.source){var d=a.data["datawrapper-height"][t]+"px";r.style.height=d}}}))}();
      </script> 
      </div>
    </div>
  </div>
 
  <div class="graficos-container">
    <!-- Gráfico 3 -->
    <div class="chart-section">
      <h2 class="chart-title">Alturas vs Promedio por Posición</h2>
      <p class="chart-description">
        Este gráfico de barras compara las alturas de jugadores de Newell’s y Central en cada posición con el promedio general. Permite observar en qué roles los equipos se desvían más o menos de la media, destacando diferencias físicas y posibles decisiones tácticas.        </p>
      <div class="chart-wrapper">
        <div class="flourish-embed flourish-chart" data-src="visualisation/22901264"><script src="https://public.flourish.studio/resources/embed.js"></script><noscript><img src="https://public.flourish.studio/visualisation/22901264/thumbnail" width="100%" alt="chart visualization" /></noscript></div>
      </div>
    </div>
  
    <!-- Gráfico 4 -->
    <div class="chart-section">
      <h2 class="chart-title">¿Quién es más alto? Alturas por posición</h2>
      <p class="chart-description">
        Este gráfico compara la altura promedio de los jugadores de Newell's y Central por posición. Muestra que los ofensivos, especialmente los extremos, suelen ser más altos, mientras que los arqueros tienen menor estatura promedio. Newell's destaca por una mayor presencia en posiciones ofensivas.      <div class="chart-wrapper">
        <div
          class="flourish-embed flourish-chart"
          data-src="visualisation/22572117"
          data-background-color="transparent"
        >
          <script src="https://public.flourish.studio/resources/embed.js"></script>
          <noscript>
            <img
              src="https://public.flourish.studio/visualisation/22572117/thumbnail"
              width="100%"
              alt="Gráfico de evolución de altura en el fútbol mundial"
            />
          </noscript>
        </div>
      </div>
    </div>
  </div>
  
    <div class="tablas-container">
 
      <div class="cuadro-comparativo cuadro-newells">
        <h3>Jugadores de Newell's Old Boys</h3>
        <table>
          <thead>
            <tr>
              <th>Jugador</th>
              <th>Altura</th>
              <th>Posición</th>
            </tr>
          </thead>
          <tbody>
            {#each jugadoresData as { nombre, alturaReal, posicion }}
              <tr>
                <td>{nombre}</td>
                <td>{((alturaReal + 100) / 100).toFixed(2)} m</td>
                <td>{posicion}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
  
     
      <div class="cuadro-comparativo cuadro-central">
        <h3>Jugadores de Rosario Central</h3>
        <table>
          <thead>
            <tr>
              <th>Jugador</th>
              <th>Altura</th>
              <th>Posición</th>
            </tr>
          </thead>
          <tbody>
            {#each jugadoresData2 as { nombre, alturaReal, posicion }}
              <tr>
                <td>{nombre}</td>
                <td>{((alturaReal + 100) / 100).toFixed(2)} m</td>
                <td>{posicion}</td>
              </tr>
            {/each}
          </tbody>
        </table>
      </div>
    </div>

<footer class="footer">
  <div class="footer-content">
    <p>Proyecto de Visualización de Datos en el Fútbol</p>
    <p>Integrantes:</p>
    <ul>
      <li><a href="https://github.com/lulicubilledo" target="_blank">Josefina Casas Pardo</a></li>
      <li><a href="https://github.com/lulicubilledo" target="_blank">Luisina Cubilledo</a></li>
    </ul>
    <p>&copy; 2025 · Universidad Torcuato Di Tella </p>
  </div>
</footer>

<style>
  * {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
  }

  html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow-x: hidden;
  background-color: #0F172A;
}

body {
  display: flex;
  flex-direction: column;
  font-family: "Poppins", "DM Sans", sans-serif;
  color: #F8FAFC;
  min-height: 100vh;
}

#container {
  width: 100%;
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 0 25px;
  flex: 1;
  box-sizing: border-box;
}


  .grafico {
    width: 100%;
    overflow-x: auto;
    overflow-y: visible;
    margin: 2rem 0; /* Reducido de 2.5rem a 2rem */
    padding-bottom: 30px; /* Reducido de 40px a 30px */
    scrollbar-width: thin;
    scrollbar-color: #6366F1 #1E293B; /* Actualizado a un tono indigo */
    -webkit-overflow-scrolling: touch;
    position: relative;
    border-radius: 12px;
    padding: 20px;
  }

  .grafico::-webkit-scrollbar {
    height: 10px;
  }

  .grafico::-webkit-scrollbar-track {
    background: #1E293B;
    border-radius: 5px;
  }

  .grafico::-webkit-scrollbar-thumb {
    background-color: #6366F1; /* Actualizado a indigo */
    border-radius: 5px;
  }

  .visualization-container {
    min-width: 100%;
    padding: 10px 0; /* Reducido de 20px a 10px */
  }

  .dashed-line {
    opacity: 0;
    transition: opacity 0.3s ease-in-out;
    pointer-events: none;
  }

  .jugador-container:hover .dashed-line {
    opacity: 0.8;
  }

  .silueta {
    filter: invert(100%) brightness(100%);
    transition: filter 0.3s ease-in-out, transform 0.2s ease-out;
    width: auto;
    max-width: 100px; /* Aumentado para mejor visualización */
    object-fit: contain;
    transform-origin: bottom center;
  }

  .jugador-container:hover .silueta {
    filter: invert(100%) brightness(100%) drop-shadow(0 0 12px var(--color));
    z-index: 10;
  }

  .altura-text {
    opacity: 0;
    transition: opacity 0.3s ease-in-out;
    font-weight: bold;
    text-anchor: middle;
    filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.8));
  }

  .jugador-container:hover .altura-text {
    opacity: 1;
  }

  .equipo-titulo {
    text-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
    filter: drop-shadow(0 0 6px rgba(255, 255, 255, 0.1));
  }

  .tooltip {
    position: fixed;
    bottom: 40px;
    right: 40px;
    background-color: #1E293B;
    color: #F8FAFC;
    padding: 1.4rem 2rem;
    border-radius: 14px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.3), 0 0 15px rgba(99, 102, 241, 0.5); /* Actualizado a indigo */
    font-family: 'Poppins', sans-serif;
    transition: all 0.3s ease;
    z-index: 100;
    border-left: 4px solid #6366F1; /* Actualizado a indigo */
    backdrop-filter: blur(10px);
    min-width: 220px;
    transform: translateY(0);
    animation: tooltip-appear 0.3s ease-out;
  }

  @keyframes tooltip-appear {
    0% { opacity: 0; transform: translateY(10px); }
    100% { opacity: 1; transform: translateY(0); }
  }

  .tooltip h4 {
    margin: 0;
    margin-bottom: 0.7rem;
    font-size: 1.3rem;
    color: #6366F1; /* Actualizado a indigo */
    border-bottom: 1px solid rgba(99, 102, 241, 0.3); /* Actualizado a indigo */
    padding-bottom: 0.5rem;
  }

  .tooltip p {
    margin: 0.4rem 0;
    font-size: 1.05rem;
  }

  .narrativa {
    max-width: 1000px;
    margin: 2.5rem auto; 
    padding: 2.5rem;
    color: #F8FAFC;
    font-family: 'Poppins', sans-serif;
    line-height: 2;
    text-align: left;
    column-gap: 3rem !important; 
    background-color: rgba(30, 41, 59, 0.4);
    border-radius: 12px;
    box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  }

  .narrativa p {
    margin: 0;
    margin-bottom: 1.4rem;
  }
    
  /* Media query para narrativa en pantallas pequeñas */
  @media (max-width: 768px) {
    .narrativa {
      columns: 1;
      padding: 1.5rem;
    }
  }

  /* Actualizado: Header a ancho completo sin gradiente */
  .dashboard-header {
  width: 100vw;    /* Vista completa del ancho */
  padding: 2rem 0;
  text-align: center;
  background-color: #1E293B;
  margin: 0;       /* Eliminar márgenes */
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  position: relative; /* Para asegurar posicionamiento adecuado */
  left: 0;           /* Alineación a la izquierda */
  right: 0;          /* Alineación a la derecha */
  box-sizing: border-box;
}

  .header-container {
    position: relative;
    z-index: 1;
    max-width: 1000px;
    margin: 0 auto;
  }

  .header-container h1 {
    font-size: 2.8rem;
    margin-bottom: 0.5rem;
    color: #6366F1; /* Color sólido en lugar de gradiente */
    font-weight: 700;
    letter-spacing: -0.5px;
  }

  .header-container p {
    font-size: 1.2rem;
    opacity: 0.85;
    margin: 0 auto;
  }

  .tablas-container {
    display: flex;
    justify-content: space-between;
    gap: 2rem;
    width: 100%;
    max-width: 1200px;
    margin: 2rem auto;
    flex-wrap: wrap;
  }

  .cuadro-comparativo {
    flex: 1 1 45%;
    min-width: 300px;
    padding: 1.5rem;
    border-radius: 12px;
    background-color: rgba(30, 41, 59, 0.4);
    box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  }

  .cuadro-newells {
    border-left: 4px solid #de2c35;
  }

  .cuadro-central {
    border-left: 4px solid #F59E0B;
  }

  .cuadro-comparativo h3 {
    margin-top: 0;
    margin-bottom: 1.5rem;
    font-size: 1.4rem;
    color: #F8FAFC;
    text-align: center;
    padding-bottom: 0.8rem;
    border-bottom: 1px solid rgba(203, 213, 225, 0.2);
  }

  table {
    width: 100%;
    border-collapse: collapse;
    margin-top: 1rem;
    font-size: 0.95rem;
  }

  th {
    text-align: left;
    padding: 0.8rem 1rem;
    border-bottom: 2px solid rgba(99, 102, 241, 0.3);
    color: #CBD5E1;
    font-weight: 600;
  }

  td {
    padding: 0.8rem 1rem;
    border-bottom: 1px solid rgba(203, 213, 225, 0.1);
    color: #F8FAFC;
  }

  tr:last-child td {
    border-bottom: none;
  }

  tr:hover td {
    background-color: rgba(99, 102, 241, 0.1);
  }

  .chart-section {
  flex: 1;
  width: 100%;
  max-width: 1200px;
  padding: 2rem;
  border-radius: 12px;
  background-color: rgba(30, 41, 59, 0.4);
  box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
  display: flex;
  flex-direction: column;
}
.chart-title {
  font-size: 1.8rem;
  color: #F8FAFC;
  margin-bottom: 1rem;
  text-align: center;
  padding-bottom: 2rem;
}

.chart-description {
  font-size: 1.05rem;
  line-height: 1.6;
  color: #E2E8F0;
  margin: 0 auto 2rem auto;
  text-align: center;
  max-width: 900px;
}

.chart-wrapper {
  width: 100%;
  flex-grow: 1;
  min-height: 400px;
  background-color: transparent;
  border-radius: 8px;
  overflow: hidden;
}

.graficos-container {
  display: flex;
  width: 100%;
  max-width: 1400px;
  margin: 2rem auto;
  gap: 2rem;
}

.flourish-embed {
  width: 100%;
  height: 100%;
  min-height: 350px;
}

/* Responsive para tablets y móviles */
@media (max-width: 992px) {
  .graficos-container {
    flex-direction: column;
  }

  .chart-section {
    margin-bottom: 2rem;
  }

  .chart-section:last-child {
    margin-bottom: 0;
  }
}


    .footer {
  width: 100vw;
  margin: 0;
  padding: 2rem 0;
  background-color: #1E293B;
  border-top: 1px solid rgba(99, 102, 241, 0.2);
  box-sizing: border-box;
  position: relative;
  left: 0;
  right: 0;
}

    .footer-content {
      width: 100%;
      max-width: 1200px;
      margin: 0 auto;
      padding: 0 2rem;
      text-align: center;
      color: #CBD5E1;
    }

    .footer p {
      margin: 0.5rem 0;
    }

    .footer ul {
      list-style: none;
      padding: 0;
      margin: 1rem 0;
      display: flex;
      justify-content: center;
      gap: 2rem;
      flex-wrap: wrap;
    }

    .footer a {
      color: #6366F1;
      text-decoration: none;
      transition: color 0.2s ease;
    }

    .footer a:hover {
      color: #818CF8;
      text-decoration: underline;
    }

    @media (max-width: 768px) {
      .tablas-container {
        flex-direction: column;
      }
      
      .cuadro-comparativo {
        margin-bottom: 1.5rem;
      }
      
      .footer ul {
        flex-direction: column;
        gap: 0.8rem;
      }
      
      h1 {
        font-size: 2rem;
      }
      
      .chart-section {
        padding: 1.5rem;
      }
    }

  </style>
</main>