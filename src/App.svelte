<script lang="ts">
  let stream: MediaStream | undefined;
  let recorder: MediaRecorder | undefined;
  let elapsed = 0;
  let interval: any;
  let stamps: number[] = [];
  let stamped = false;
  let timeout: any;

  function count() {
    interval = setInterval(() => {
      elapsed += 0.1;
    }, 100);
  }

  async function prompt() {
    stream = await navigator.mediaDevices.getDisplayMedia({
      audio: true,
      video: true,
    });
  }

  async function record(stream: MediaStream) {
    stamps = [];
    elapsed = 0;

    const chunks: Blob[] = [];

    recorder = new MediaRecorder(stream);
    recorder.addEventListener("dataavailable", (e) => {
      if (e.data.size > 0) {
        chunks.push(e.data);
      }
    });
    recorder.addEventListener("stop", () => save(chunks));
    // For every 200ms the stream data will be stored in a separate chunk.
    recorder.start(200);
    count();
  }

  function save(chunks: Blob[]) {
    const type = "webm";
    const blob = new Blob(chunks, {
      type: `video/${type}`,
    });

    const downloadLink = document.createElement("a");
    downloadLink.href = URL.createObjectURL(blob);
    downloadLink.download = `qa.${type}`;

    document.body.appendChild(downloadLink);
    downloadLink.click();
    URL.revokeObjectURL(blob as any);
    document.body.removeChild(downloadLink);
  }

  function stamp() {
    stamps = [...stamps, Number(Math.max(0, elapsed - 5).toFixed(3))];
    stamped = true;
    if (timeout) clearTimeout(timeout);
    timeout = setTimeout(() => {
      stamped = false;
    }, 500);
  }

  function stop() {
    recorder.stop();
    recorder = undefined;
    clearInterval(interval);
  }

  $: {
    if (stream) {
      record(stream);
    }
  }
</script>

<main>
  <h1>Quality Assurance Recorder</h1>

  <button on:click={recorder ? stop : prompt}>
    {#if recorder}
      Stop
    {:else}
      Record
    {/if}
  </button>
  {#if elapsed > 0}
    {#if recorder}
      <button class="yellow" on:click={stamp}>Stamp {elapsed.toFixed(1)}</button>
    {:else}
      Elapsed time: {elapsed.toFixed(1)}
    {/if}
  {/if}
  {#if stamped}
    Stamped!
  {/if}
  {#if !recorder && stamps.length}
    <div>
      <h3>Timestamps:</h3>
      {#each stamps as stamp, i (i)}
        <p>{stamp}s</p>
      {/each}
    </div>
  {/if}
</main>
