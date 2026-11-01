<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const count = (cause: string) => groups.find((g) => g.cause === cause)?.reports.length ?? 0;
  // The desk's one-day moves are filed under `parser` by tenant, but they are not a flipped day and month.
  const oneDay = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)).length);
  const rows = $derived([
    { name: 'parser reads month first', n: count('parser') - oneDay },
    { name: 'booking-v3 form hints MM/DD/YYYY', n: count('hint') },
    { name: 'booking lands one day off', n: oneDay },
  ]);
</script>

<table>
  <thead><tr><th>problem</th><th>reports</th></tr></thead>
  <tbody>
    {#each rows as r (r.name)}<tr><td>{r.name}</td><td>{r.n}</td></tr>{/each}
  </tbody>
  <tfoot><tr><td>merged into #{ticket}</td><td>{merged.length}</td></tr></tfoot>
</table>
