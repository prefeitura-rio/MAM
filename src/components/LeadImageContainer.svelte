<script>
  import ImageRaw from "$components/ImageRaw.svelte";
  import StreetSvg from "$components/StreetSvg.svelte";
  import locator from "$svg/story/ca_locator.svg";
  import { beforeUpdate } from "svelte";
  import { onMount } from "svelte";
  import { selectAll, select, easeLinear } from "d3";
  import Stains from "$components/Stains.svelte";
  // import rough from 'roughjs';
  let section
  export let stepIndex
  export let h;

  // beforeUpdate(() => {
  //   console.info("Mobility", stepIndex)
  // })
  let calPath;
  let maderaDot;
  let gardenaDot;
  let fremontDot;
  let m2gLine;
  let g2fLine;
  let m2gArrow;
  let g2fArrow;
  let w;
  //let rc = rough.svg('svg');

  const positioningClasses = {
    "madera-ggp":  {
      triggerStep: 1,
      endStep: 2,
      position: "row-start-1 col-start-2 col-span-5",
      positionMobile: "row-start-2 col-start-1 col-span-12",
      altText: "A black and white photo of the author's grandmother and grandfather posing for a picture together in front of an old car. The grandmother is wearing a dress, long coat, glasses, and a bonnet. The grandfather is wearing jeans with a long sleeve button up tucked in."
    },
    "madera-mom": {
      triggerStep: 1,
      endStep: 2,
      position:"row-start-2 col-start-6 col-span-7",
      positionMobile: "row-start-4 col-start-1 col-span-12",
      altText: "A black and white photo of the author's mother and grandmother sitting on a lawn with three small children."
    },
    "cimeira3": {
      triggerStep: 1,
      endStep: 2,
      position:"row-start-4 col-start-8 col-span-4",
      positionMobile: "row-start-5 col-start-1 col-span-12",
      altText: "A black and white photo of the author's mother and grandmother sitting on a lawn with three small children."
    },
    "gardena-mom-and-me": {
      triggerStep: 3,
      endStep: 4,
      position: "row-start-1 col-start-2 col-span-5",
      positionMobile: "row-start-2 col-start-1 col-span-7",
      altText: "A color Polaroid photo of the author as a young boy being hugged by his mother in 1994. The author is wearing a purple tank top with patterned shorts and his mother is wearing a white tank top and striped pants."
    },
    "gardena-preschool": {
      triggerStep: 3,
      endStep: 4,
      position: "row-start-4 col-start-4 col-span-5",
      positionMobile: "row-start-3 col-start-4 col-span-7",
      altText: "A school portrait of the author from preschool. He is smiling with his front two teeth missing and is wearing a navy, red, green, and yellow striped shirt against a crushed blue background."
    },
    "3incendio": {
      triggerStep: 3,
      endStep: 4,
      position: "row-start-5 col-start-7 col-span-5",
      positionMobile: "row-start-4 col-start-6 col-span-7",
      altText: "A school portrait of the author from preschool. He is smiling with his front two teeth missing and is wearing a navy, red, green, and yellow striped shirt against a crushed blue background."
    },
    "fremont": {
      triggerStep: 5,
      endStep: 6,
      position: "row-start-1 col-start-2 col-span-6",
      positionMobile: "row-start-2 col-start-1 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    },
    "2acervo": {
      triggerStep: 5,
      endStep: 6,
      position: "row-start-2 col-start-7 col-span-4",
      positionMobile: "row-start-3 col-start-3 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    },
    "3acervo": {
      triggerStep: 5,
      endStep: 6,
      position: " row-start-4 col-start-2 col-span-4",
      positionMobile: "row-start-4 col-start-5 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    },
    "cimeira1": {
      triggerStep: 6,
      endStep: 7,
      position: "row-start-1 col-start-2 col-span-5",
      positionMobile: "row-start-1 col-start-3 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    },
    "cimeira2": {
      triggerStep: 6,
      endStep: 7,
      position:"row-start-2 col-start-6 col-span-7",
      positionMobile: "row-start-4 col-start-6 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    },
    "cimeira33": {
      triggerStep: 6,
      endStep: 7,
      position:"row-start-5 col-start-6 col-span-4",
      positionMobile: "row-start-3 col-start-1 col-span-8",
      altText: "A photo of the author in an elemenary school classroom smiling and holding up a written report in 1999. the author is wearing a navy swish jacket with a navy and white plaid button-up shirt underneath."
    }
  }

  // function roughen(group, color) {
  //   select(group).each(function() {
  //     let gParent = this;
  //     select(group).selectAll('#ca_x5F_outline').each(function() {
  //       let roughPath = gParent.appendChild( rc.path(select(this).node().getAttribute('d'), {
  //           stroke: color,
  //           fillStyle: 'hachure',
  //           strokeWidth: 0.55,
  //           roughness: 0.75,
  //         })
  //       )
  //     })
  //   })
  // }

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
        .duration(500)
        .ease(easeLinear)
        .attr("stroke-dashoffset", 0)
        .on("end", resolve);
    })
  }

  onMount(() => {
    calPath = selectAll("#ca_x5F_outline");
    maderaDot = selectAll("#madera_dot");
    gardenaDot = selectAll("#gardena_dot");
    fremontDot = selectAll("#fremont_dot");
    m2gLine = selectAll("#m2g_line");
    g2fLine = selectAll("#g2f_line");
    m2gArrow = selectAll("#m2g_arrow");
    g2fArrow = selectAll("#g2f_arrow");

    // roughen(".locator-svg-wrapper svg", "#282828")
  });

  beforeUpdate(() => {
    document.querySelectorAll("#wrapper > div").forEach((node, index) => {
      const dataIndex = parseInt(node.dataset.index);
      const dataEnd = parseInt(node.dataset.end);
      if (stepIndex === dataIndex) {
        //selectAll("#wrapper > div").classed("opacity-0", true)
        node.classList.remove("opacity-0")
        node.classList.add("opacity-100")
      } else if (stepIndex === dataEnd)  {
        selectAll("#wrapper > div").classed("opacity-100", false)
        selectAll("#wrapper > div").classed("opacity-0", true)
      }
    })

  })

  function updateLocator(stepIndex) {

    if (stepIndex != undefined && stepIndex < 1) {
      maderaDot
        .transition()
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);

      gardenaDot.style("opacity", 0);
      fremontDot.style("opacity", 0);
      m2gLine.style("opacity", 0);
      m2gArrow.style("opacity", 0);
      g2fLine.style("opacity", 0);
      g2fArrow.style("opacity", 0);
    } else if (stepIndex >= 0 && stepIndex < 3) {
      gardenaDot.style("opacity", 0);
      fremontDot.style("opacity", 0);
      m2gLine.style("opacity", 0);
      m2gArrow.style("opacity", 0);
      g2fLine.style("opacity", 0);
      g2fArrow.style("opacity", 0);
    } else if (stepIndex == 3) {
      m2gLine.style("opacity", 1);
      drawPaths(m2gLine)
      m2gArrow
        .transition()
        .delay(500)
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);
      gardenaDot
        .transition()
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);
      
      fremontDot.style("opacity", 0);
      g2fLine.style("opacity", 0);
      g2fArrow.style("opacity", 0);
    }
    else if (stepIndex >= 3 && stepIndex < 6) {
      gardenaDot
        .transition()
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);

      fremontDot.style("opacity", 0);
      g2fLine.style("opacity", 0);
      g2fArrow.style("opacity", 0);
    } else if (stepIndex == 6) {
      g2fLine.style("opacity", 1);
      drawPaths(g2fLine)
      g2fArrow
        .transition()
        .delay(500)
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);
      fremontDot
        .transition()
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);
    } else if (stepIndex >= 6) {
      fremontDot
        .transition()
        .duration(250)
        .ease(easeLinear)
        .style("opacity", 1);
    }
  }

  $: updateLocator(stepIndex)
</script>

<svelte:window bind:innerWidth={w}/>

<!-- class="grid grid-cols-12 grid-rows-2"  -->
<!-- <div id="grid-photos" class="w-full h-screen relative z-[60] p-4 max-w-5xl my-0 mx-auto"> -->
  <section class="sticky top-0 flex h-screen" bind:clientHeight={h}>
    <div 
      bind:this={section} 
      id="wrapper" 
      class="grid w-full grid-cols-12 grid-rows-6 gap-3"
    >
      {#each Object.keys(positioningClasses) as key, index}
        {#if w > 900}
        <div data-index={positioningClasses[key].triggerStep} data-end={positioningClasses[key].endStep} class={`${positioningClasses[key].position} transition-opacity duration-500 opacity-0`}>
          <div class="border-2 border-black">
            <ImageRaw 
              className="w-full"
              src={`assets/img/intro/${key}.jpg`} 
              alt={positioningClasses[key].altText}
            />
          </div>
        </div> 
        {:else}
        <div data-index={positioningClasses[key].triggerStep} data-end={positioningClasses[key].endStep} class={`${positioningClasses[key].positionMobile} imagem-teste transition-opacity duration-500 opacity-0`}>
          <div class="border-2 border-black">
            <ImageRaw 
              className="w-full"
              src={`assets/img/intro/${key}.jpg`} 
              alt={positioningClasses[key].altText}
            />
          </div>
        </div> 
        {/if}     
      {/each}
      </div>
      <div class="city-svg-wrapper">
        <StreetSvg 
          step={stepIndex}
        />
      </div>
      <!-- <div class="locator-svg-wrapper">
        {@html locator}
      </div> -->
      {#if h != undefined}
        <Stains height={h}/>
      {/if}
  </section>
<!-- </div> -->

<style>
  section {
    width: 100%;
  }

  #wrapper {
    z-index: 1000;
    width: 100%;
    padding: 2rem;
    margin: 0;
  }
  .city-svg-wrapper {
    width: 100%;
    height: 100vh;
    top: 50%;
    left: 50%;
    position: absolute;
    transform: translate(-50%, -50%);
    z-index: 1;
    margin: 0 auto;
    max-width: 50rem;
  }

  .locator-svg-wrapper {
    position: absolute;
    left: 0;
    top: 50%;
    z-index: 1;
    width: 8rem;
    height: 8rem;
    transform: translate(0, -50%);
    display: none;
    justify-content: center;
  }

  :global(.locator-svg-wrapper svg) {
    width: 100%;
  }

  @media only screen and (min-width: 700px) {
    #wrapper {
      /* border-left: 1px solid var(--color-off-black); */
      margin: 2rem 2rem 2rem 8rem;
    }
    .locator-svg-wrapper {
      display: flex;
    }
    .city-svg-wrapper {
      margin: 0 0 0 8rem;
      max-width: 50rem;
    }
  }

  @media only screen and (min-width: 900px) {
    #wrapper {
      margin: 2rem 2rem 2rem 12rem;
    }
    .city-svg-wrapper {
      margin: 0 0 0 12rem;
      max-width: 50rem;
    }
    .locator-svg-wrapper {
      width: 12rem;
      height: 12rem;
    }
  }
  @media only screen and (max-width: 620px) {
    section{
        width: 110%;
        height: 50rem;
    
      }
    }

  
</style>
