<script>
  import Header from "$components/Header.svelte";
  import { selectAll, select, easeLinear } from "d3";
  // import rough from 'roughjs';
  import Stains from "$components/Stains.svelte";
  import { onMount } from "svelte";

  import copy from "$data/doc.json";

  let cutoutPhoto;
  let largeUSPaths;
  let smallUSPaths;
  let filledPath;
  let headlineSpan;
  let introBlockW;
  let w;
  let h;
  //let rc = rough.svg('svg');

  function drawPaths(pathCollection) {
    return new Promise((resolve) => {
      const paths = pathCollection;
      const lineNodes = pathCollection._groups[0];
      const lineLengths = [...lineNodes].map((el) => el.getTotalLength());

      paths
        .attr("stroke-dasharray", (d, i) => lineLengths[i])
        .attr("stroke-dashoffset", (d, i) => lineLengths[i])
        .transition()
        //.delay((d, i) => i * 100)
        .duration(750)
        .ease(easeLinear)
        .attr("stroke-dashoffset", 0)
        .on("end", resolve);
    });
  }

  function popIn(item, property, val, delay, dur) {
    return new Promise((resolve) => {
      item
        .transition()
        .delay(delay)
        .duration(dur)
        .ease(easeLinear)
        .style(property, val)
        .on("end", resolve);
    });
  }

  async function run() {
    await popIn(largeUSPaths, "opacity", 1, 0, 0);
    await drawPaths(largeUSPaths);
    //await roughen('#large_x5F_us', '#D8284F', 0);
    await popIn(smallUSPaths, "opacity", 1, 0, 0);
    await drawPaths(smallUSPaths);
    //await roughen('#small_x5F_us', '#282828', 1);
    await popIn(filledPath, "fill", "#262626", 0, 0);
    await popIn(cutoutPhoto, "opacity", 1, 250, 0);
    await popIn(headlineSpan, "top", "-100px", 250, 250);
  }

  onMount(() => {
    cutoutPhoto = selectAll(".intro-cutout img");
    filledPath = selectAll("#map_x5F_filled");
    largeUSPaths = selectAll("#large_x5F_us path");
    smallUSPaths = selectAll("#small_x5F_us path");
    headlineSpan = selectAll(".hed-block");
    run();
  });
</script>

<svelte:window bind:innerWidth={w} />

<div class="relative z-50 w-full mb-6 h-4/5" bind:clientHeight={h}>
  <div class="intro-wrapper">
    <!-- <div class="absolute w-full"> -->
    <Header />
    <!-- </div> -->
    <div class="overlays">
      <div class="intro-svg">
        <img src="assets/img/00_CAPA_SEM_FUNDO.png" alt="Map of the United States"/>
       </div>
   
    </div>
    {#if w < 700}
      <div class="hed">
        <p style='margin-top:5rem; font-size:50px !important' class="hed-text">Museu Renovado, Parque Vivo</p>
        <div class="intro-block">
          <p class="intro-text">{copy.description}</p>   
          <p class="intro-text">{copy.description_continuation}</p>
          <p class="byline sm">
            Desenvolvido por &nbsp&nbsp<a href="https://www.dados.rio"  target="_blank">Escritório de Dados da Prefeitura do Rio de Janeiro </a> 
          </p>
        </div>
      </div>
    {:else}
      <div class="hed">
        <!-- <div><p class="hed-text">On</p></div> -->
        <div class="hed-block">
          <p class="hed-text"><span bind:clientWidth={introBlockW}>Museu Renovado, Parque Vivo</span></p>
          <div class="intro-block">
            <p class="intro-text" style="max-width: {introBlockW}px">{copy.description}</p>
           
          <p class="intro-text"style="max-width: {introBlockW}px" >{copy.description_continuation}</p>
            <p class="mb-1 byline sm">
              Desenvolvido por <span><br></span><a href="https://www.dados.rio"  target="_blank">Escritório de Dados da Prefeitura do Rio de Janeiro </a> 
            </p>
            
          </div>
        </div>
        <!-- <div><p class="hed-text">Mobility</p></div> -->
      </div>
    {/if}
    <h1 aria-label="On Upward Mobility">On Upward Mobility</h1>
  </div>
  {#if h != undefined}
    <Stains height={h} />
  {/if}
</div>

<style>
  .intro-svg img {
 margin-top:-12rem;
 display: block;
 margin-left: auto;
 margin-right: auto;
 width: 100%; 
 transform: scale(1.5);
 padding-top:5rem;
 padding-bottom:5rem;
}

  .intro-wrapper {
    position: relative;
    z-index: 999;
  }
  .overlays {
    position: relative;
    height: calc(100vw / 1.75);
    max-width: 80rem;
    margin: -3rem auto 0 auto;
    pointer-events: none;
  }
  .intro-cutout {
    width: 100%;
    position: relative;
    top: 0;
    left: 0;
  }

  .hed {
    display: flex;
    flex-direction: column;
    justify-content: center;
    padding: 0 1rem;
    max-width: 80rem;
    margin: 3rem auto 0 auto;
  }

  .hed-block {
    display: flex;
    flex-direction: column;
    max-width: 400px;
    position: relative;
  }

  .hed-text {
    font-family: var(--dubois-wide);
    text-transform: uppercase;
    font-size: 4rem !important;
    padding: 0;
    line-height: 1;
    margin-top:-50px 
  }

  .hed-text span {
    display: inline-block;
   
  }

  .intro-text,
  .byline {
    font-size: 16px;
    font-family: var(--dubois);
    text-transform: uppercase;
    padding: 0 0 2rem 0;
  }

  .byline {
    padding: 0;
    font-size: 16px;
  }

  .byline a {
    font-weight: 700;
    color: var(--color-off-black);
  }

  .byline a:hover {
    color: var(--color-db-red);
  }

  .sm {
    font-size: 14px;
    padding: 0;
  }

  .intro-block {
    border-top: none;
    position: relative;
    padding: 1rem 0;
    margin-top: 0rem;
  }

  .intro-cutout img {
    opacity: 0;
  }

  :global(.intro-cutout) {
    z-index: 1000;
  }

  :global(.intro-svg svg) {
    width: 200%;
  }

  :global(.intro-svg svg #large_x5F_us path) {
    stroke-width: 2px;
  }

  h1 {
    font-family: var(--dubois-wide);
    text-align: center;
    text-transform: uppercase;
    font-size: 86px;
    opacity: 0;
    pointer-events: none;
    position: absolute;
  }

  @media only screen and (min-width: 550px) {
    .intro-text {
      font-size: 20px;
    }

    .hed {
      padding: 0 2rem;
    }
  }

  @media only screen and (min-width: 700px) {
    .hed-text {
      font-size: 3.5rem;
      padding: 0 1rem;
    }

    .hed {
      display: flex;
      flex-direction: row;
      margin: 6rem auto 0 auto;
    }

    .intro-block {
      /* border-top: 2px solid var(--color-db-red); */
      position: relative;
      padding: 1rem 1rem 0 1rem;
      margin-top: 0rem;
    }
  }

  @media only screen and (min-width: 800px) {
    .hed-text {
      font-size: 4.125rem;
    }
  }

  @media only screen and (min-width: 1000px) {
    .hed-text {
      font-size: 5.5rem;
    }
  }

  @media only screen and (min-width: 1500px) {
    .overlays {
      height: calc(100vw / 2);
    }
  }
</style>
