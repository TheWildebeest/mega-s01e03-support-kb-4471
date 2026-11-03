<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  let { ticket }: { ticket: number } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const moved = $derived(merged.filter((r) => notes.oneDayMove.includes(r.id)));
  const rest = $derived(merged.filter((r) => !notes.oneDayMove.includes(r.id)));
  const groups = $derived(groupByCause(rest, tenantOf));
  const count = (c: string) => groups.find((g) => g.cause === c)?.reports.length ?? 0;
</script>

<p><strong>{merged.length}</strong> unique reports in #{ticket}, <strong>{groups.length + (moved.length ? 1 : 0)}</strong> problems.</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>belongs in #{ticket}?</th></tr></thead>
  <tbody>
    <tr><td><code>parser</code> reads dates month first</td><td>{count('parser')}</td><td>yes</td></tr>
    <tr class="wrong"><td><code>hint</code> booking-v3 prints MM/DD/YYYY</td><td>{count('hint')}</td><td>no, needs its own ticket</td></tr>
    <tr class="wrong"><td>one-day move (desk notes)</td><td>{moved.length}</td><td>no, needs its own ticket</td></tr>
  </tbody>
</table>

<style>
  .wrong td { background: var(--warn-bg); }
</style>
