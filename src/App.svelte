<script>
  import * as Tone from "tone";

  let bpm = 120;
  let beat = 0;
  let isPlaying = false;

  const kickSynth = new Tone.MembraneSynth({
    volume: 30
  }).toDestination();
  const pianoSynth = new Tone.NoiseSynth().toDestination();
  const clapSynth = new Tone.NoiseSynth({ noise: { type: "pink" } }).toDestination();
  const hatSynth = new Tone.MetalSynth({
    volume: 1
  }).toDestination();

  let rows = [
    Array.from({ length: 16 }, (_, i) => ({ sound: "kick", active: false })),
    Array.from({ length: 16 }, (_, i) => ({ sound: "piano", active: false })),
    Array.from({ length: 16 }, (_, i) => ({ sound: "clap", active: false })),
    Array.from({ length: 16 }, (_, i) => ({ sound: "hat", active: false })),
  ];

  let beatIndicators = Array.from({ length: 16 }, (_, i) => i);

  Tone.Transport.scheduleRepeat(time => {
    rows.forEach((row, index) => {
      let note = row[beat];
      if (note.active) {
        // Trigger different synthesizers based on the sound
        switch (note.sound) {
          case "kick":
            kickSynth.triggerAttackRelease("C1", "8n", time);
            break;
          case "piano":
            pianoSynth.triggerAttackRelease("8n", time);
            break;
          case "clap":
            clapSynth.triggerAttackRelease("8n", time);
            break;
          case "hat":
            hatSynth.triggerAttackRelease("8n", time);
            break;
        }
      }
    });
    beat = (beat + 1) % 16;
  }, "16n");

  const handleNoteClick = (rowIndex, noteIndex) => {
    rows[rowIndex][noteIndex].active = !rows[rowIndex][noteIndex].active;
  };

  const handlePlayClick = () => {
    if (!isPlaying) Tone.start();
    Tone.Transport.bpm.value = bpm;
    Tone.Transport.start();
    isPlaying = true;
  };

  const handleStopClick = () => {
    Tone.Transport.stop();
    isPlaying = false;
  };

  $: if (isPlaying) {
    Tone.Transport.bpm.value = bpm;
  }
</script>

<div>
  <h1 class="heading">
    <span class="green">_</span>
    <span class="white">.</span>
    <span class="green">M</span>
    <span class="white">U</span>
    <span class="green">S</span>
    <span class="white">I</span>
    <span class="green">C</span>
    <span class="white">_</span>
    <span class="green">S</span>
    <span class="white">t</span>
    <span class="green">e</span>
    <span class="white">p</span>
    <span class="green">_</span>
    <span class="white">S</span>
    <span class="green">e</span>
    <span class="white">q</span>
    <span class="green">u</span>
    <span class="white">e</span>
    <span class="green">n</span>
    <span class="white">c</span>
    <span class="green">e</span>
    <span class="white">r</span>
    <span>.</span>
    <span class="white">_</span>
  </h1>
</div>

<div class="bpm-controls">
  <label for="bpm">{bpm} BPM</label>
  <input class='slider' type="range" id="bpm" min="60" max="240" bind:value={bpm} />
  {#if isPlaying}
    <button class="play-stop-button" on:click={handleStopClick}>Stop</button>
  {:else}
    <button class="play-stop-button" on:click={handlePlayClick}>Play</button>
  {/if}
</div>

<div class="sequencer">
  {#each beatIndicators as beatIndicator, bi}
    <div class="beat-indicator {bi === beat - 1 ? 'live' : ''}"></div>
  {/each}
  {#each rows as row, i}
    {#each row as note, j}
      <button 
        on:click={() => handleNoteClick(i, j)}
        class="note {note.active ? 'active' : ''} {j % 4 === 0 ? 'first-beat-of-the-bar' : ''}"></button>
    {/each}
  {/each}
</div>

<style>
  .heading {
    font-size: 3rem;
    color: #05b08f;
    text-align: center;
    margin-bottom: 10px;
    text-transform: uppercase;
  }
  .green {
    color: #05b08f; /* Green color */
  }
  .white {
    color: #fff; /* White color */
  }
  .slider{
    margin-left: 20px
  }
  .play-stop-button {
    display: inline-block;
    padding: 10px 20px;
    background-color: #05b08f;
    color: #fff;
    border: none;
    margin: 30px;
    border-radius: 5px;
    font-size: 1rem;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }

  .play-stop-button:hover {
    background-color: #05f18f;
  }
  .sequencer {
    display: grid;
    grid-template-columns: repeat(16, 1fr);
    grid-gap: 5px;
    width: 100%;
  }
  .note {
    background: #ccc;
    font-size: 10px !important;
    width: 50px;
    height: 50px;
    border: 1px solid #ccc;
    border-radius: 7px;
    display: flex;
    justify-content: center;
    align-items: center;
    font-size: 2rem;
    padding: 0;
    box-sizing: border-box;
  }
  .active {
    background: #05b08f !important;
    border: 1px solid #600889;
  }
  .first-beat-of-the-bar {
    background: #98c9fa;
    border: 1px solid #98c9fa;
  }
  .bpm-controls {
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 30px;
    margin-top: 100px;
  }
  .bpm-controls label {
    color: #fff;
  }

  .beat-indicator {
    width: 10px;
    height: 10px;
    border-radius: 50%;
    background: #555;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    font-size: 1.5rem;
    margin: 0 auto;
  }
  .live {
    background: #05f18f;
  }
</style>