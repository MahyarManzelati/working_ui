<script lang="ts">
  import { onDestroy } from "svelte";
  import { doc, onSnapshot } from "firebase/firestore";
  import { db } from "./firebase";

  let jobId = "";
  let status = "";
  let itinerary: any = null;
  let error = "";
  let unsubscribe: (() => void) | null = null;

  function checkStatus() {
    if (unsubscribe) {
      unsubscribe();
      unsubscribe = null;
    }

    status = "";
    itinerary = null;
    error = "";

    const id = jobId.trim();
    if (!id) {
      error = "Please enter a Job ID";
      return;
    }

    // ← point here at the 'itineraries' collection
    const ref = doc(db, "itineraries", id);

    unsubscribe = onSnapshot(
      ref,
      (snap) => {
        if (!snap.exists()) {
          error = "Job not found";
          return;
        }
        const data = snap.data();
        status = data.status;
        if (status === "completed") {
          itinerary = data.itinerary;
        } else if (status === "failed") {
          error = data.error || "Unknown error";
        }
      },
      (err) => {
        error = `Realtime listener error: ${err.message}`;
      }
    );
  }

  onDestroy(() => {
    if (unsubscribe) unsubscribe();
  });
</script>

<main class="p-6 max-w-lg mx-auto">
  <h1 class="text-3xl font-bold mb-6 text-center">Itinerary Status Checker</h1>

  <div class="flex gap-2 mb-4">
    <input
      class="flex-1 border rounded p-2 focus:outline-none focus:ring"
      bind:value={jobId}
      placeholder="Enter your Job ID"
      on:keydown={(e) => e.key === "Enter" && checkStatus()}
    />
    <button
      class="bg-blue-600 text-white rounded px-4 py-2 hover:bg-blue-700 disabled:opacity-50"
      on:click={checkStatus}
      disabled={!jobId.trim()}
    >
      Check
    </button>
  </div>

  {#if error}
    <p class="text-red-600 mb-4"><strong>Error:</strong> {error}</p>
  {:else if status}
    <div class="space-y-2">
      <p><strong>Status:</strong> {status}</p>
      {#if status === "completed" && itinerary}
        <pre class="bg-gray-100 rounded p-4 overflow-auto">
{JSON.stringify(itinerary, null, 2)}
        </pre>
      {/if}
    </div>
  {/if}
</main>
