<script>
  // IMPORT STATEMENTS
  // Importing necessary D3 modules and Svelte components
  import { forceSimulation, forceX, forceY, forceCollide } from "d3-force";
  import { scaleLinear, scaleSqrt, scaleBand, scaleOrdinal } from "d3-scale";
  import { nest } from "d3-collection";
  import { group } from "d3-array";

  import * as d3 from 'd3'
  
  // Importing custom Svelte components
  import Scrolly from "./Scrolly.svelte";
  import Force from './Force.svelte'
  import ScatterPlot from './ScatterPlot.svelte'
  import Histogram from './Histogram.svelte'
  import AxisY from './AxisY.svelte'
  // import nodes  from "./Force.svelte"
  // console.log(nodes)

  // Importing data
  import data from "./data.js";
  console.log(data)



class SVGFader extends HTMLElement {
  constructor() {
    super();
    this.attachShadow({ mode: 'open' });
  }

  connectedCallback() {
    const src = this.getAttribute('src');
    const fadeDuration = parseInt(this.getAttribute('fade-duration')) || 1000;
    const maxWidth = this.getAttribute('max-width') || '100%';

    this.shadowRoot.innerHTML = `
      <style>
        :host {
          display: block;
          width: 100%;
          max-width: ${maxWidth};
          margin: 0 auto;
        }
        .svg-container {
          position: relative;
          width: 100%;
          padding-bottom: 75%; /* Adjust this value based on your SVG's aspect ratio */
          overflow: hidden;
        }
        svg {
          position: absolute;
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          width: 100%;
          height: 100%;
          max-height: 100%;
          object-fit: contain;
        }
        @media (max-width: 768px) { /* Adjust this breakpoint as needed */
          .svg-container {
            padding-bottom: 0;
            height: 70vh; /* Adjust this value to control the height on mobile */
          }
        }
      </style>
      <div class="svg-container"></div>
    `;

    this.loadSVG(src, fadeDuration);
  }

  async loadSVG(src, fadeDuration) {
    try {
      const response = await fetch(src);
      const svgText = await response.text();
      const container = this.shadowRoot.querySelector('.svg-container');
      container.innerHTML = svgText;

      const svg = container.querySelector('svg');
      if (svg) {
        svg.setAttribute('width', '100%');
        svg.setAttribute('height', '100%');
        svg.setAttribute('preserveAspectRatio', 'xMidYMid meet');
      }

      const layers = Array.from(container.querySelectorAll('g[id]')).reverse();
      layers.forEach(layer => {
        layer.style.opacity = '0';
        layer.style.transition = `opacity ${fadeDuration}ms`;
      });

      this.fadeInLayers(layers, fadeDuration);
    } catch (error) {
      console.error('Error loading SVG:', error);
    }
  }

  fadeInLayers(layers, fadeDuration, index = 0) {
    if (index < layers.length) {
      layers[index].style.opacity = '1';
      setTimeout(() => this.fadeInLayers(layers, fadeDuration, index + 1), fadeDuration);
    }
  }
}

customElements.define('svg-fader', SVGFader);
  
  

  // CONSTANTS & SCALES
  // Setting up constants and scales
  let width = 1000;
  let height = 600;
  let circleRadius = 15; 
  // const performanceCategories = ["Below Average", "Average", "Good", "Excellent"]

  var circles = d3.select("svg");
  console.log(circles)

  const colorRange = [
    "#C6458D",
    "#808080",
    "#044892",
    "#D0842E",
    "#00502F"
    ];

  // const colorScale = scaleOrdinal().domain("1", "2", "3", "4", "5", "6").range(colorRange);

  // Setting up dimensions
  let margin = { top: 50, left: 0, right: 10, bottom: 50 };
  $: chartWidth = width - margin.left - margin.right
  $: chartHeight = height - margin.top - margin.bottom

  // Setting up scales using D3
  $: xScale = scaleLinear().domain([0, 100]).range([0, chartWidth]);
  $: yScale = scaleLinear().domain([0, 60]).range([chartHeight, 0]);
  $: roleScale = scaleBand().domain(["1", "2", "3", "4", "5"]).range([chartWidth*0.2, chartWidth * 1]);
  $: monsterScale = scaleBand().domain(["1", "2"]).range([chartWidth*0.2, chartWidth * 1]);
  $: raceScale = scaleBand().domain(["1", "2"]).range([chartWidth*0.2, chartWidth * 1]);
  $: surviveScale = scaleBand().domain(["1", "2"]).range([chartWidth*0.2, chartWidth * 1]);
  $: themeScale = scaleBand().domain(["1", "2", "3", "4", "5"]).range([chartWidth*0.2, chartWidth * 1]);
  $: empowerScale = scaleBand().domain(["1", "2"]).range([chartWidth*0.2, chartWidth * 1]);
  $: directorScale = scaleBand().domain(["1", "2"]).range([chartWidth*0.2, chartWidth * 1]);

  // Setting up additional scales for the histogram
  // $: yHistScale = d3.scaleLinear().domain([0, 10]).range([chartHeight, 0])
  // $: xHistScale = d3.scaleBand().domain(performanceCategories).range([0, chartWidth])
  
  // DECLARATIONS
  // Setting up variables for scrolly telling
  let currentStep;
  let simulation = forceSimulation(data)
  

</script>	

<section>
    <div class="full-bleed-banner">
      <img src="./img/women-in-horror-banner.png" alt="Women in Horror Banner">
        <figcaption class="caption">Natalie Portman in Black Swan (2010)</figcaption>
    </div>
</section>

<section>		
  <!-- Section content with scrolly telling -->
  <div class='hero'>
    <!-- <h1>Women in Horror</h1> -->
    <h1>How do the most popular horror movies of all time depict their leading ladies?</h1>
    <h2>By Christina Li</h2>
  </div>
  <div class="content">
     <p> I didn't always like horror movies. In fact, I tended to avoid the genre altogether. But some time in recent years, I started to take notice in how entertaining yet insightful
      horror movies could be in their social commentary. My appreciation for the horror genre has only grown as I discovered more horror movies with complex female characters that captured
      all of the intense emotions that women experience in their lives – their fear, rage, claustrophobia, quest for revenge or justice. Horror movies of today are unlike the ones in the
      1970s and prior, which were slasher flicks that targeted women as victims to be disposed of.
    <br>
    <br>
    By analyzing the depictions of women in horror movies, we can see how certain ideas are reinforced about women, including deep-seated fears and anxieties about womanhood.
    The top 400 movies of all time<sup id="fnref1"><a href="#fn1" class="footnote-ref">1</a></sup> were evaluated because they are the most watched and most discussed.
    These movies are the most culturally relevant, contributing to ongoing discussions on the real-life treatment of women.
    <br>
    <br>
    First, by comparing the top 400 movies in horror to other genres, it's clear that horror is a genre that is quite gendered.
    Similar to the romance genre, nearly half of the top movies have a woman in the leading role<sup id="fnref2"><a href="#fn2" class="footnote-ref">2</a></sup>.
    <br>
    <br>
    </p>
    </div>
    <div class="chart">
      <div class="chart-container">
        <img src="./img/barChart.svg" alt="Bar Chart of Gender Breakdown by Genre" class="chart-img">
      </div>
      </div>
    <br>
    <br>
    <div class="content">
    <p>Next, let's break down the top female-centric horror movies to get a better understanding of how women are depicted on screen, the roles that they play, and what kinds of female-centered stories are being told.
      The following movie analysis was conducted with the responses and explanations generated by the OpenAI API in conjuction with ChatGPT 4.0. (More on the methodology at the end of the story.)
      <br>
      <br>
      WARNING: Spoilers for movies ahead!</p>
  </div>

  <div class= 'section-container'>
    <div class='steps-container'>
      <!-- Scrolly component to handle steps -->
      <Scrolly bind:value = {currentStep}>
        {#each ['a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p'] as text, index}
          <div class='step' class:active = {currentStep === index}>
            <div class='step-content'>
              <!-- Conditional content for each step -->
              {#if text === 'a'}
                <p>Out of the top 400 horror movies, 187 have a woman in the leading role.
                  <br>
                  <br>
                  Each circle represents a female-centric horror movie.
                  <br>
                  <br>
                  Hover to see each movie title.
                </p>
              {:else if text === 'b'}
                <p>What role do these women play? 
                  <br>
                  <br>
                Categories from left to right<sup id="fnref3"><a href="#fn3" class="footnote-ref">3</a></sup>:</p>
                <p><a class='bg-tag pink-bg'>Mother or housewife</a></p>
                <p><a class='bg-tag yellow-bg'>Adolescent girl</a></p>
                <p><a class='bg-tag blue-bg'>Professional role associated with beauty</a>
                  <br>
                  (such as dancer or actress)</p>
                <p><a class='bg-tag green-bg'>Professional role associated with caretaking</a>
                  <br>
                (such as nun or nurse)</p>
                <p><a class='bg-tag gray-bg'>Other</a>
                  <br>
                  <br>
                  A lot of horror movies cast the leading lady in a "traditional" gender role associated with femininity and vulnerability.
                  Out of the 187 films, 105 star a protagonist that adheres to one of the traditionally female roles, while 82 do not and fall into the 'Other' category. 
                  </p>
              {:else if text === 'c'}
                <p>Let's look at some examples of female protagonists who fall in the <a class='bg-tag gray-bg'>Other</a> category, playing a role that is not traditionally seen as feminine.
                <br>
                <br>
                The text box next to each movie includes an explanation generated by OpenAI API with ChatGPT 4.0 as the large language model version<sup id="fnref4"><a href="#fn4" class="footnote-ref">4</a></sup>.</p>
              {:else if text === 'd'}
              <p>
                Next, let's look at how many women are portrayed as monsters.
                <br>
                <br>
                In the book <a href="https://www.routledge.com/The-Monstrous-Feminine-Film-Feminism-Psychoanalysis/Creed/p/book/9780415052597"><i>The Monstrous-Feminine</i></a>,
                Barbara Creed argues that women in horror movies are often shown as monstrous figures. One of her key insights is that the female body, especially in its maternal and reproductive capacities, becomes a site of horror.
                These depictions tap into cultural fears and taboos about women's bodies, such as having powers that threaten male characters or giving birth to something monstrous.
                <br>
                <br>
                Using her book as a framework, movies were evaluated by how closely the female protagonist fits into one of the categories of "the monstrous-feminine":
                archaic mother, monstrous womb, vampire, witch, possessed body, monstrous mother, and castrator<sup id="fnref5"><a href="#fn5" class="footnote-ref">5</a></sup>.
                <br>
                <br>
                According to this definition, fewer of the most popular horror movies depict the female protagonist as a <a class='bg-tag gray-bg'>monster</a> (71) than <a class='bg-tag pink-bg'>not a monster</a> (116).</p>
              {:else if text === 'e'}
              <p>These are some examples of movies where the female protagonist is a <a class='bg-tag gray-bg'>monster</a>.
                <br>
                <br>
                The "monstrous mother" and the "monstrous womb" are common types of monsters, with female characters symbolizing fear and revulsion due to their biological functions.
                <br>
                <br>
                Women in horror are also often shown as castrators or possessing powers that destabilize the male symbolic order,
                which aligns with historical fears of women as witches or demonic figures.
                <br>
                <br>
                These depictions in horror films serve as metaphors for real-life gender dynamics. By consistently framing women as monstrous or uncontrollable,
                these films reinforce cultural fears of female sexuality and autonomy, as well as broader societal anxieties around femininity and female power.</p>
              {:else if text === 'f'}
              <p>The horror genre can hold up a mirror to society, as a way to explore real fears and anxieties that people experience.
                In particular, the depiction of the female body as grotesque or horrifying reflects deep-rooted anxieties around womanhood and female sexuality. 
                <br>
                <br>
                What fears and anxieties related to womanhood, especially as it relates to the female body, do horror movies reflect?
                <br>
                <br>
                Categories from left to right<sup id="fnref6"><a href="#fn6" class="footnote-ref">6</a></sup>:
                <br>
                <p><a class='bg-tag pink-bg'>Motherhood</a>
                <br>
                (including pregnancy or childbirth)</p>
                <p><a class='bg-tag yellow-bg'>Gender-based violence</a></p>
                <p><a class='bg-tag green-bg'>Female sexuality</a>
                  <br>
                  (including coming of age)</p>
                <p><a class='bg-tag blue-bg'>Beauty and perfection</a></p>
                <p><a class='bg-tag gray-bg'>Other</a>
                <br>
                <br>
                Perhaps unsurprisingly, the theme of gender-based violence is the most prominent in horror movies, followed by the theme of motherhood<sup id="fnref7"><a href="#fn7" class="footnote-ref">7</a></sup>.
              </p>
              {:else if text === 'g'}
              <p>Here are some examples of movies about <a class='bg-tag pink-bg'>motherhood</a>.
              <br>
            <br>
            Some movies may revolve around more than one theme, but I selected the theme that best underscores the plotline.
            <br>
            <br>
            For example, <i>Rosemary's Baby</i> (1968) includes themes of both motherhood and gender-based violence, but the plot ultimately revolves around the protagonist as an expectant mother.
            <br>
          <br>
          Pregnancy is commonly portrayed as a source of physical and psychological horror in films, symbolizing the loss of control over one’s body.
          In <i>Rosemary's Baby</i>, the horror stems from the idea that her body is being used against her will, reflecting societal anxieties about women’s reproductive rights and the invasive nature of pregnancy.
          In the <i>Alien</i> franchise, the alien creature’s reproductive process, which involves parasitic implantation, reflects a grotesque version of pregnancy. </p>
              {:else if text === 'h'}
              <p>Examples of movies about <a class='bg-tag yellow-bg'>gender-based violence</a>, where the violence is the main driver of the plot or the movie acts as commentary
                on the violence women have to suffer through in society.
              <br>
            <br>
            Many horror films historically exploit the fear of violence against women, especially in the "slasher" subgenre. Movies like <i>The Texas Chainsaw Massacre</i> (1974) and <i>Friday the 13th</i> (1984)
            depict young women being stalked, brutalized, and killed by male figures. These films often follow the "final girl" trope, where a lone female survivor faces the killer at the end.
            They tend to fetishize the violence leading up to her victory, presenting women as passive victims to male aggression and voyeuristic violence.
          </p>
              {:else if text === 'i'}
              <p>Examples of movies about <a class='bg-tag green-bg'>female sexuality</a>, where sexual awakening or agency is linked to violence or chaos.
              <br>
              <br>
              This can be seen in <i>Carrie</i> (1976), where the protagonist’s first menstruation coincides with the development of destructive telekinetic powers.
              Similarly, in <i>Ginger Snaps</i> (2000), Ginger's werewolf transformation starts with her first period, linking menstruation to the horror of becoming monstrous.
              As Ginger becomes more sexually aware, her behavior becomes increasingly aggressive and predatory.
              <br>
              <br>
              These films play with the idea that female puberty is something both natural and frightening, when a girl's body and desires begin to change in ways that challenge patriarchal norms.</p>
              {:else if text === 'j'}
              <p>Examples of movies about <a class='bg-tag blue-bg'>beauty and perfection</a>.
              <br>
              <br>
              Films like <i>Neon Demon</i> (2016) critique the pressures placed on women to maintain a flawless appearance, with beauty becoming a form of currency that leads to violent competition and destruction.
              <br>
              <br>
              Films like <i>Black Swan</i> (2010) portray how the relentless pursuit of perfection can be psychologically and physically destructive, reflecting larger cultural anxieties around the impossible ideals imposed on women.
            </p>
              {:else if text === 'k'}
              <p>For movies that do not fit in the previous categories, these are examples of themes in the <a class='bg-tag gray-bg'>Other</a> category, where the theme is not specifically tied
                to womanhood.</p>
              {:else if text === 'l'}
                <p>The good news is that more female protagonists <a class='bg-tag pink-bg'>survive</a> than <a class='bg-tag gray-bg'>die</a> at the end.</p>
              {:else if text == 'm'}
                <p>But that doesn't always mean their story is empowering to watch.
                  <br>
                  <br>
                  Even when there is a woman leading the movie, more of their stories are ultimately about <a class='bg-tag pink-bg'>female victimization</a> (98) than <a class='bg-tag gray-bg'>female empowerment</a> (89).
                <br>
                <br>
                In order to define if a movie provides empowerment to its main character, I used a prompt from Five Thirty Eight's <a href="https://projects.fivethirtyeight.com/next-bechdel/"><i>The Next Bechdel Test</i></a>, which has the following conditions:
                  <br>
                  <span class="code-font">- The female lead has dimension and exists authentically with needs and desires that she pursues through dramatic action
                <br>
                - And the audience can empathize with or understand the female lead's desires and actions</span>
                <br>
                <br>
                I then added more definitions for empowerment and victimization for ChatGPT 4.o to more accurately categorize each movie<sup id="fnref8"><a href="#fn8" class="footnote-ref">8</a></sup>: 
                <br>
                <span class="code-font">- Additionally, empowerment is defined as the protagonist is able to overcome physical or psychological threats, and she is able to gain agency and autonomy during the movie's runtime. The movie is more about her taking control over her own story
                <br>
                  - Victimization, on the other hand, can be defined as the protagonist is subjected to physical or psychological threats for most of the movie's runtime, and she is not able to gain agency or autonomy. The movie is more about her immediate survival</span>
                </p>
              {:else if text == 'n'}
                <p>Here are some examples of stories centered around <a class='bg-tag pink-bg'>female victimization</a>.
                  <br>
                  <br>
                It is not always the case that because the woman survives at the end that her story is empowering. For example, the protagonist in <i>Mother!</i> (2017) survives at the end, but she is subjected to threats
              repeatedly throughout the movie without the ability to change her circumstances.</p>
              {:else if text == 'o'}
                <p>And here are some examples of stories centered around <a class='bg-tag gray-bg'>female empowerment</a>.
                  <br>
                  <br>
                  Interestingly, some movies in this category subvert gender norms, reversing the typical male predator and female victim dynamic.
                  <br>
                  <br>
                  For example, <i>A Girl Walks Home Alone At Night</i> (2014) defies expectations, even in its title. The main character is a female vampire who freely walks alone at night, and instead of
                  the one in peril, she is the source of fear for the predatory men she encounters.
                  <br>
                  <br>
                  <i>Revenge</i> (2017) overturns the "rape-revenge" genre, which typically focuses on women as helpless victims who reclaim power through violence,
                  by focusing less on the exploitative aspects and more on the woman’s transformation and empowerment.
                  <br>
                  <br>
                  These movies attempt to reclaim the genre by telling stories about women that reflect autonomy, complexity, and resistance.
                </p>
              {:else if text == 'p'}
                <p>
                  The portrayal of women's stories in horror movies offers a complex and often unsettling reflection of societal fears and gender dynamics.
                  <br>
                  <br>
                  So, who gets to tell stories about womanhood?
                  <br>
                  <br>
                  Far more of the most popular horror movies of all time have been made by a <a class='bg-tag pink-bg'>male director</a> (164) than a 
                  <a class='bg-tag gray-bg'>female director</a> (23).
                </p>
              {/if}
            </div>
          </div>
        {/each}
      </Scrolly>
    </div>
    
    <div class= 'sticky' bind:clientWidth = {width}>
      <!-- Dynamic content based on scrolly step -->
      {#if currentStep === 0}
        <Force {width} {height} {margin} {simulation} X ={width / 2} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05}/>
      {:else if currentStep === 1}
        <Force {width} {height} {margin} {simulation} X ={d => roleScale(d.Role_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Role_Code"}/>
          <!-- {#each nodes as node}
            <circle fill={colorScale(node.Role_Code)}/>
          {/each} -->
      {:else if currentStep === 2}
      <svg-fader src="./img/Photo-Boards-Roles.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 3}
        <Force {width} {height} {margin} {simulation} X ={d => monsterScale(d.Monster_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Monster_Code"}/>
      {:else if currentStep === 4}
      <svg-fader src="./img/Photo-Boards-Monsters.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 5}
        <Force {width} {height} {margin} {simulation} X ={d => themeScale(d.Theme_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Theme_Code"}/>
      {:else if currentStep === 6}
      <svg-fader src="./img/Photo-Boards-Motherhood.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 7}
      <svg-fader src="./img/Photo-Boards-Violence.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 8}
      <svg-fader src="./img/Photo-Boards-Sexuality.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 9}
      <svg-fader src="./img/Photo-Boards-Beauty.svg" width="1000px"fade-duration="1000"></svg-fader>
      {:else if currentStep === 10}
      <svg-fader src="./img/Photo-Boards-Other.svg" width="1000px" fade-duration="1000"></svg-fader>
      {:else if currentStep === 11}
        <Force {width} {height} {margin} {simulation} X ={d => surviveScale(d.Die_Survive_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Die_Survive_Code"}/>
      {:else if currentStep === 12}
        <Force {width} {height} {margin} {simulation} X ={d => empowerScale(d.Victim_Empower_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Victim_Empower_Code"}/>
      {:else if currentStep === 13}    
      <svg-fader src="./img/Photo-Boards-Victim.svg" width="1000px"fade-duration="1000" ></svg-fader>  
      {:else if currentStep === 14}    
      <svg-fader src="./img/Photo-Boards-Empower.svg" width="1000px" fade-duration="1000"></svg-fader>    
      {:else}
        <Force {width} {height} {margin} {simulation} X ={d => directorScale(d.Director_Code)} Y ={height/2} circleRadius={10} alpha={0.7} adecay={0.05} fills={"Director_Code"}/>
      {/if}
    </div>
  </div>

  
  <div class="footnote">
    Author's Note
    <br>
    <br>
    This is a prototype for a longer project looking at how the portrayal of women in horror movies has changed throughout history.
    Given this project is focused on a limited number of most popular horror movies, there are limitations such as the dataset skewing towards more recently released movies.
    The popularity of movies can also shift over time. This project is focused on the most popular horror movies dated to July 2024.
    <br>
    <br>
    This project was inspired by <a href="https://pudding.cool/2024/07/scifi/"><i>Who Killed the World?</i></a> by The Pudding and
    <a href="https://projects.fivethirtyeight.com/next-bechdel/"><i>The Next Bechdel Test</i></a> by Five Thirty Eight.
    The idea behind it was to conduct a movie analysis by assigning each movie to different categories to look at broader trends.
    <br>
    <br>
    Methodology
    <br>
    <br>
    <div id="fn1" class="footnote-item">
      <sup><a href="#fnref1" class="footnote-backref">1</a></sup>The top 400 movies were scraped from Letterboxd since the platform allows for movies to be sorted by Film Popularity - All Time. The original idea was to scrape the top 400 movies
    from both Letterboxd and IMDB and combine them into one dataset, which could account for potentially different audiences. However, IMDB movies sorted by Popularity includes
    movies yet to be released (but are popular due to buzzworthiness) and would require more filtering based on a certain number of user votes. In my opinion, Letterboxd provided the more straightforward
    criteria for movie popularity.
    </div>
    <div id="fn2" class="footnote-item">
      <sup><a href="#fnref2" class="footnote-backref">2</a></sup>The gender of the protagonist was determined by scraping the first name of the cast list of each movie, then manually checking each actor's gender. 
      A few movies were discovered to have listed the first actor in the cast list incorrectly. I cross-referenced the cast list of said movies on IMDB and Wikipedia. The following movies were initially coded
      with the main character as female and later changed to male: <i>The Purge: Election Year</i>, <i>Haunted Mansion</i>, and <i>Circle</i>.
    </div>
    <div id="fn3" class="footnote-item">
    <sup><a href="#fnref3" class="footnote-backref">3</a></sup><u>Role Definition</u>: Role categories were determined by my own observations of what kinds of roles women usually play in horror movies.
    </div>
    <div id="fn4" class="footnote-item">
      <sup><a href="#fnref4" class="footnote-backref">4</a></sup><u>Role Analysis</u>: The plot and themes (if available) of each movie was collected from Wikipedia and given to OpenAI API, with ChatGPT 4.0 as the large language model version, to determine what role category each movie belonged to.
    Several prompts were tested until I felt like I received the best results.
    </div>
    <div id="fn5" class="footnote-item">
      <sup><a href="#fnref5" class="footnote-backref">5</a></sup><u>Monster Definition & Analysis</u>: Inspired by <a href="https://www.routledge.com/The-Monstrous-Feminine-Film-Feminism-Psychoanalysis/Creed/p/book/9780415052597"><i>The Monstrous Feminine: Film, Feminism, Psychoanalysis</i></a> by Barbara Creed,
    I gave the book blurb to ChatGPT 4.0 so it could categorize each female protagonist as a monster or not.
    </div>
    <div id="fn6" class="footnote-item">
      <sup><a href="#fnref6" class="footnote-backref">6</a></sup><u>Theme Definition</u>: Theme categories were determined using a a thesis titled <a href="https://digitalcommons.pace.edu/cgi/viewcontent.cgi?article=1216&context=honorscollege_theses"><i>Fears and the Female Circumstance: Women in 1970s Horror Films</i></a> by Lorian Gish,
    in which the author categorized the core female-centric themes in 1970s horror films into 3 buckets: 1) coming of age and sexual identity, 2) domesticity and suburbia, and 3) motherhood and pregnancy.
    However, I believe domesticity and motherhood to be closely linked enough to constitute one category.
    I added theme categories of: gender-based violence, beauty and perfection, and other. These themes were also chosen as they are aligned with changes to, monitoring of, and violations to the female body, which is a 
    prominent focus of horror movies.
    </div>
    <div id="fn7" class="footnote-item">
      <sup><a href="#fnref7" class="footnote-backref">7</a></sup><u>Theme Analysis</u>: I prompted ChatGPT 4.0 to read the plot and themes (if available) of each movie from Wikipedia to determine what theme category it belonged to the best.
    I prompted it to return two themes each movie could potentially be about, and if none of the theme categories fit, it would return 'Other' as an answer with an explanation.
    For movies where more than one theme could be applicable, I selected the theme that best underscores the plotline.
    Several prompts were tested until I was satisfied with the detailed explanations for why it answered the way it did. I manually checked a sample of movies and believed that I would have answered similarly.
    </div>
    <div id="fn8" class="footnote-item">
      <sup><a href="#fnref8" class="footnote-backref">8</a></sup><u>Victimization vs. Empowerment Definition & Analysis</u>: Female victimization versus female empowerment is perhaps the most subjective category. I used <a href="https://projects.fivethirtyeight.com/next-bechdel/"><i>The Next Bechdel Test</i></a> by Five Thirty Eight as a guide
    where director Kimberly Peirce came up with a category to see if a movie passes her test based on how well-developed the female protagonist is. I refined the prompt I gave
    to ChatGPT 4.0 until it gave me detailed explanations as to why it placed each movie in either category. I manually checked a sample of movies and believed that I would have answered similarly.
    There is full acknowledgement that not everyone will agree with the results.
    </div>
  </div>

  <div class="credit">
    <br>
    <br>
    <b>Svelte template for this site can be found <a href="https://svelte.dev/repl/ea206af4d7bc454f969ee6e31fb545b0?version=4.1.1">here</a>.
    Code for this data story, as well as readings referenced and tutorials for building a site with Svelte, can be found <a href="https://github.com/christinamyli/women-in-horror-prototype">here</a>.
    The data set with responses and explanations provided by OpenAI API / ChatGPT 4.0 can be found <a href="https://github.com/christinamyli/women-in-horror-prototype/blob/main/data/Letterboxd_Horror_Female_Wiki_OpenAI_final_2.xlsx">here</a>.</b>
    <br>
    <br>
    <b>Many thanks to <a href="https://yanansun0074.github.io/portfolio/">Yanan Sun</a>, <a href="https://thomastaoli.github.io/">Thomas Li</a>, <a href="https://manyunzou.github.io/">Manyun Zou</a>, and <a href="https://mhkeller.com/">Michael Keller</a> for their assistance on this project.</b>
    <br>
    <br>
    <br>
    <br>
  </div>
</section>


<link href="https://fonts.googleapis.com/css2?family=Source+Code+Pro&display=swap" rel="stylesheet">



<style>

h1, h2, p, .step-content {
  font-family: Georgia, 'Times New Roman', Times, serif;
  line-height: 1.5;
  color: #333;
}

.content {
  max-width: 640px;
  width: 90%; /* This allows some breathing room on smaller screens */
  margin: auto;
}

.hero {
  display: flex;
  flex-direction: column;
  justify-content: left;
  text-align: left;
  padding: 2rem 1rem;
}

h1 {
  font-size: 2.5rem; /* 40px at base font size */
  text-align: center;
  margin-bottom: 1rem;
}

h2 {
  font-size: 1.5625rem; /* 25px at base font size */
  text-align: center;
  margin-bottom: 2rem;
}

p {
  font-size: 1.25rem; /* 20px at base font size */
  text-align: left;
  margin-bottom: 1rem;
}

/* .footnote{
  font-size: 14px;
  text-align: left;
  max-width: 640px;
  width: 90%; /* This allows some breathing room on smaller screens */
  /* margin: auto;
} */

.footnote {
            font-size: 14px;
            text-align: left;
            max-width: 640px;
            width: 90%;
            margin: auto;
            padding-top: 20px;
        }

        /* Styles for linked footnotes */
        .footnote-ref, .footnote-backref {
            text-decoration: none;
            color: inherit;
        }

        .footnote-item {
            margin-bottom: 10px;
        }

        .footnote-number {
            vertical-align: super;
            font-size: 0.75em;
            line-height: 0;
        }

.credit {
  font-size: 16px;
  text-align: left;
  max-width: 640px;
  width: 90%; /* This allows some breathing room on smaller screens */
  margin: auto;
}

.code-font {
    font-family: 'Source Code Pro', monospace;
    font-size: 0.85rem;
}


.full-bleed-banner {
    position: relative;
    width: 100vw;
    margin-left: 50%;
    transform: translateX(-50%);
    overflow: hidden;
}

.full-bleed-banner img {
    width: 100%;
    height: auto;
    display: block;
}

.caption {
    font-style: italic; /* Italicize the caption */
    margin-top: 10px; /* Add some space between the image and the caption */
    font-size: 12px;
    text-align: right;
    padding-right: 20px; /* Add padding to the right */
}

.chart-container {
    width: 100%;
    max-width: 1000px; /* or whatever maximum width you want */
    margin: 0 auto;
}

  .chart-img {
    width: 100%;
    height: auto;
    display: block;
}
  
  a {
    color:#044892;
}
    
  .bg-tag {
    color: white;
    background-color: black;
    padding-right: 10px;
    padding-left: 10px;
    padding-top: 5px;
    padding-bottom: 5px;
    border-radius: 10px;
}

  .pink-bg{
    background-color: #C6458D;
}

  .yellow-bg{
    background-color: #D0842E;
}

  .blue-bg{
    background-color: #044892;
}

  .green-bg{
    background-color: #00502F;
}

  .gray-bg{
    background-color: #808080;
}

@import url('https://fonts.googleapis.com/css2?family=Barlow+Condensed:ital,wght@1,800&family=IBM+Plex+Serif&display=swap');
    :global(body) {
        overflow-x: hidden;
        width : 100%;
        margin: 0;
        background-color: #FEFAF1
        
}

  .section-container {
    margin-top: 1em;
    text-align: center;
    transition: background 100ms;
    display: flex;
}

  .step-content {
    font-size: 1rem;
    background-color: #FEFAF1;
    color: #ccc;
    border: 1px solid #FEFAF1;
    padding: .5rem 1rem;
    transition: background, border 500ms ease;
    text-align: left;
    width: 75%;
    margin: auto;
    max-width: 500px;
}

  .step {
    height: 100vh;
    display: flex;
    place-items: center;
    justify-content: center;
}

  .step.active .step-content {
      background-color: #FEFAF1;
      border: 1px solid black;
      color: black;
}
    
  .steps-container,
  .sticky {
      height: 100%;
}

  .steps-container {
      flex: 1 1 40%;
      z-index: 2;
}

  .sticky {
    position: sticky;
    top: 10%;
        flex: 1 1 60%;
    width: 80%;
}

/* Comment out the following line to always make it 'text-on-top' */
/* Responsive adjustments */
@media screen and (max-width: 768px) {

  .content {
    width: 95%;
  }

  h1 {
    font-size: 2rem;
  }

  h2 {
    font-size: 1.25rem;
  }

  p {
    font-size: 0.8rem;
  }

  .footnote {
    font-size: 0.8rem;
  }

  .credit {
    font-size: 0.8rem;
  }

  .code-font{
    font-size: 0.6rem;
  }

  .step-content {
    width: 85%;
    font-size: 0.9rem;
  }

  .step.active {
      opacity: 0.75;
}
  

    .section-container {
      flex-direction: column-reverse;
    }

    .sticky {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
  }
}
</style>
