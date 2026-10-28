<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { uniqueById, type Report } from '#lib/kb/causes';

  // The reports the desk read as a one-day move, quoted as they came in.
  const rows = uniqueById(reports as Report[]).filter((r) => notes.oneDayMove.includes(r.id));
</script>

<ul>
  {#each rows as r (r.id)}
    <li><code>{r.id}</code> · {r.receivedAt.slice(0, 10)} · {r.tenant} · {r.channel}: “{r.body}”</li>
  {/each}
</ul>
