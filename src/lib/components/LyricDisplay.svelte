<script>
  import { onMount } from 'svelte';
  import ResultsDisplay from './ResultsDisplay.svelte';
  export let lyrics;
  export let songTitle;
  export let artistName;
  export let imageUrl;
  let userInput = '';
  let startTime = null;
  let endTime = null;
  let testStarted = false;
  let cursorPosition = 0;
  let inputElement;
  let showResults = false;
  let wpm = 0;
  let accuracy = 0;
  let preloadedImage;
  console.log(songTitle, artistName, imageUrl);
  
  async function preloadImage(src) {
    const img = new Image();
    img.src = src;
    img.onload = () => {
      preloadedImage = img;
    };
  }

  function focusInput() {
    if(inputElement) inputElement.focus();
  }
  
  onMount(() => {
    focusInput();
  });

  function startTest() {
    if (!testStarted) {
      startTime = new Date();
      testStarted = true;
      preloadImage(imageUrl);
    }
  }

  // Function to end the test and calculate WPM and accuracy
  function endTest() {
    endTime = new Date();
    const durationInMinutes = (endTime - startTime) / 60000;
    const charactersTyped = userInput.length; // Use userInput length instead
    wpm = (charactersTyped / 5) / durationInMinutes;

    // Make sure to calculate accuracy after the entire userInput is processed
    const incorrectChars = formattedLyrics.reduce((acc, { class: charClass }) => acc + (charClass === 'incorrect' ? 1 : 0), 0);
    accuracy = ((charactersTyped - incorrectChars) / lyrics.length) * 100; // Ensure to divide by total lyrics length for accuracy
    showResults = true;

    console.log(`WPM: ${wpm.toFixed(2)}, Accuracy: ${accuracy.toFixed(2)}%`);
  }

  $: if (lyrics){
    showResults = false;
    userInput = '';
    focusInput();
    testStarted = false;
  };

  // Reactive statement to update formattedLyrics based on lyrics
  $: formattedLyrics = lyrics.split('').map((char, index) => {
    return { char, class: '' };
  });

  // Reactive statement to update the class of each character span based on user input
  $: {
    if (userInput && formattedLyrics.length > 0) {
      if (!testStarted) startTest();
      if (userInput.length === lyrics.length) endTest(); // Assuming completion is based on length match
      
      // Normalize userInput by collapsing multiple spaces and leaving single spaces intact.
      const normalizedUserInput = userInput.replace(/ +/g, ' ');

      // Split the lyrics into characters but keep the newlines and other whitespace intact.
      const lyricsChars = lyrics.split('');

      // Create an array of the same length as lyricsChars to track correct and incorrect input.
      const updatedFormattedLyrics = lyricsChars.map((char, index) => {
        if (index < normalizedUserInput.length) {
          let inputChar = normalizedUserInput[index];
          // If the character in lyrics is a whitespace (space, newline, etc.), treat any space typed by the user as correct.
          if (/\s/.test(char) && inputChar === ' ') {
            inputChar = char;
          }
          return {
            char,
            class: inputChar === char ? 'correct' : 'incorrect'
          };
        }
        // For the characters not yet reached by userInput, return them as neutral.
        return { char, class: '' };
      });

      formattedLyrics = updatedFormattedLyrics;
    } else if (userInput.length === 0) {
      // If userInput is empty, reset all classes to ''
      formattedLyrics = formattedLyrics.map(item => ({ ...item, class: '' }));
    }
  };

  $: cursorPosition = userInput.length;
</script>

{#if showResults && preloadedImage}
  <ResultsDisplay {wpm} {accuracy} {songTitle} {artistName} imageUrl={preloadedImage.src} />
{:else}
<div class="quote-display" role="button" tabindex="0" on:click={focusInput} on:keydown={focusInput}>
  {#each formattedLyrics as { char, class: spanClass }, i}
    {#if i === cursorPosition}
      <span class="blinking-cursor"></span>
    {/if}
    <span class={spanClass}>{char}</span>
  {/each}
  <!-- Handle the case where the cursor should be at the end of the text -->
  {#if cursorPosition === formattedLyrics.length}
    <span class="blinking-cursor"></span>
  {/if}
</div>
{/if}
<input bind:this={inputElement} class="quote-input" type="text" bind:value={userInput} />


<style>
  * {
    box-sizing: border-box;
  }
  .quote-display{
    white-space: pre-wrap;
    margin-top: 20px;
    padding: 10px;
    /* background-color: #f0f0f0; */
    border-radius: 8px;
    letter-spacing: -.18em;
    font-family: "Geneva", sans-serif;
    font-size: 1.6em;
    line-height: 180%;
    font-weight: 500;
  }

  .quote-input {
    position: absolute;
    opacity: 0;
    background-color: transparent;
  }

  .correct {
    color: rgb(54, 142, 94);
  }

  .incorrect {
    color: #bd5454;
  }

  .blinking-cursor {
    /* Ensure minimal layout impact */
    display: inline-block;
    width: 2px; /* Adjust based on the desired cursor thickness */
    height: 1.2em; /* Typically, matching the line height of the text */
    margin: 0; /* Ensure no additional space is added around the cursor */
    margin-right: -6px;
    margin-left: 4px;
    vertical-align: text-bottom; /* Aligns the cursor better with the baseline of the text */
    background-color: currentColor; /* Inherits the text color; adjust if a different cursor color is preferred */
    /* Blink animation */
    animation: blink-animation 1s steps(1) infinite;
  }
  .cursor-placeholder {
    /* Ensure this does not introduce additional spacing */
    width: 0;
    margin: 0;
    padding: 0;
    display: inline-block;
    letter-spacing: 0.1em;
  }

  @keyframes blink-animation {
    50% {
      opacity: 0;
    }
  }
</style>

