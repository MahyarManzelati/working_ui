<script lang="ts">
  import { db } from "./firebase";
  import { doc, onSnapshot } from "firebase/firestore";

  let jobId = "";
  let status: "processing" | "completed" | "failed" | "unknown" = "unknown";
  let destination = "";
  let durationDays: number | null = null;
  let itinerary: Array<any> = [];
  let errorMsg: string | null = null;
  let createdAt: string | null = null;
  let completedAt: string | null = null;

  let unsub: null | (() => void) = null;

  function reset() {
    status = "unknown";
    destination = "";
    durationDays = null;
    itinerary = [];
    errorMsg = null;
    createdAt = null;
    completedAt = null;
  }

  function formatTs(ts: any) {
    // Works with Firestore Timestamp or string/number fallback
    try {
      if (!ts) return null;
      if (typeof ts?.toDate === "function") return ts.toDate().toISOString();
      if (typeof ts === "string") return ts;
      if (typeof ts === "number") return new Date(ts).toISOString();
    } catch {}
    return null;
  }

  function subscribe() {
    if (!jobId.trim()) {
      alert("Please enter a jobId.");
      return;
    }
    if (unsub) {
      unsub();
      unsub = null;
    }
    reset();

    const ref = doc(db, "itineraries", jobId.trim());
    unsub = onSnapshot(
      ref,
      (snap) => {
        if (!snap.exists()) {
          status = "unknown";
          return;
        }
        const data = snap.data() as any;

        status =
          data.status === "processing" ||
          data.status === "completed" ||
          data.status === "failed"
            ? data.status
            : "unknown";

        destination = data.destination ?? "";
        durationDays = data.durationDays ?? null;
        itinerary = Array.isArray(data.itinerary) ? data.itinerary : [];
        errorMsg = data.error ?? null;

        createdAt = formatTs(data.createdAt);
        completedAt = formatTs(data.completedAt);
      },
      (err) => {
        console.error(err);
        errorMsg = err.message ?? String(err);
        status = "failed";
      }
    );
  }

  // Clean up when component unmounts
  // (Svelte 4/5 compatible)
  import { onDestroy } from "svelte";
  onDestroy(() => {
    if (unsub) unsub();
  });
</script>

<main class="page">
  <h1>Itinerary Status Checker</h1>

  <form class="controls" on:submit|preventDefault={subscribe}>
    <input
      placeholder="Enter jobId..."
      bind:value={jobId}
      autocomplete="off"
      spellcheck="false"
      class="input"
    />
    <button type="submit" class="btn">Check</button>
  </form>

  <section class="card">
    <h2>Status</h2>
    <div class="kv">
      <div>jobId</div>
      <div class="mono">{jobId || "—"}</div>
      <div>status</div>
      <div class="status {status}">{status}</div>
      <div>destination</div>
      <div>{destination || "—"}</div>
      <div>durationDays</div>
      <div>{durationDays ?? "—"}</div>
      <div>createdAt</div>
      <div>{createdAt ?? "—"}</div>
      <div>completedAt</div>
      <div>{completedAt ?? "—"}</div>
    </div>

    {#if errorMsg}
      <p class="error">Error: {errorMsg}</p>
    {/if}
  </section>

  {#if itinerary?.length}
    <section class="card">
      <h2>Itinerary</h2>
      {#each itinerary as day}
        <article class="day">
          <header>
            <h3>Day {day.day}</h3>
            {#if day.theme}<p class="theme">{day.theme}</p>{/if}
          </header>
          {#if Array.isArray(day.activities) && day.activities.length}
            <ul class="acts">
              {#each day.activities as act}
                <li>
                  <div class="time">{act.time}</div>
                  <div class="desc">{act.description}</div>
                  {#if act.location}<div class="loc">@ {act.location}</div>{/if}
                </li>
              {/each}
            </ul>
          {:else}
            <p class="muted">No activities listed.</p>
          {/if}
        </article>
      {/each}
    </section>
  {/if}
</main>

<style>
  :root {
    font-family:
      system-ui,
      -apple-system,
      Segoe UI,
      Roboto,
      Ubuntu,
      Cantarell,
      "Helvetica Neue",
      Arial,
      "Noto Sans",
      "Apple Color Emoji",
      "Segoe UI Emoji";
  }
  .page {
    max-width: 880px;
    margin: 2rem auto;
    padding: 0 1rem;
  }
  h1 {
    font-size: 1.75rem;
    margin-bottom: 1rem;
  }
  .controls {
    display: flex;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }
  .input {
    flex: 1;
    padding: 0.65rem 0.8rem;
    border: 1px solid #ddd;
    border-radius: 0.75rem;
  }
  .btn {
    padding: 0.6rem 1rem;
    border: 1px solid #111;
    background: #111;
    color: #fff;
    border-radius: 0.75rem;
    cursor: pointer;
  }
  .card {
    border: 1px solid #eee;
    border-radius: 1rem;
    padding: 1rem;
    margin-top: 1rem;
  }
  .kv {
    display: grid;
    grid-template-columns: 140px 1fr;
    gap: 0.35rem 0.75rem;
    margin-top: 0.5rem;
  }
  .mono {
    font-family: ui-monospace, SFMono-Regular, Menlo, Consolas,
      "Liberation Mono", monospace;
    overflow-wrap: anywhere;
  }
  .status {
    text-transform: uppercase;
    font-weight: 700;
    font-size: 0.9rem;
  }
  .status.processing {
    color: #b26a00;
  }
  .status.completed {
    color: #0a7a2d;
  }
  .status.failed {
    color: #b00020;
  }
  .error {
    color: #b00020;
    font-weight: 600;
  }
  .day {
    margin-top: 0.75rem;
    padding-top: 0.5rem;
    border-top: 1px dashed #e6e6e6;
  }
  .day header {
    display: flex;
    align-items: baseline;
    gap: 0.5rem;
  }
  .day h3 {
    margin: 0;
  }
  .theme {
    color: #555;
  }
  .acts {
    list-style: none;
    padding-left: 0;
    margin: 0.25rem 0 0;
    display: grid;
    gap: 0.5rem;
  }
  .acts li {
    border: 1px solid #f0f0f0;
    border-radius: 0.75rem;
    padding: 0.5rem 0.75rem;
  }
  .time {
    font-weight: 600;
  }
  .desc {
    margin-top: 0.15rem;
  }
  .loc {
    margin-top: 0.15rem;
    color: #555;
  }
  .muted {
    color: #777;
  }
</style>
