<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  type Ticket = { url: string; outcome: string };
  let { ticket, tickets }: { ticket: number; tickets: { parser: Ticket; hint: Ticket } } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const WHAT: Record<string, string> = {
    parser: 'shared parser reads the date month first',
    hint: 'booking-v3 hint tells people to type month first',
    unplaced: 'no cause: tenant not on file or not day first',
  };
  const ticketNo = (url: string) => `#${url.split('/').pop()}`;
</script>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th><th>outcome</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const t = g.cause === 'unplaced' ? undefined : tickets[g.cause]}
      {@const moved = readAs(g, notes.oneDayMove)}
      <tr class:open={!t}>
        <td><code>{g.cause}</code> {WHAT[g.cause]}</td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if t}<a href={t.url}>{ticketNo(t.url)}</a>{:else}none yet{/if}</td>
        <td>{t?.outcome ?? ''}</td>
      </tr>
      {#if moved}
        <tr class="within">
          <td><code>shift</code> date moves by one day, filed under <code>{g.cause}</code> by tenant</td>
          <td>{moved}</td>
          <td colspan="4">of the {g.reports.length} above</td>
        </tr>
      {/if}
    {/each}
  </tbody>
  <tfoot>
    <tr><td>merged into #{ticket}</td><td>{merged.length}</td><td colspan="4"></td></tr>
  </tfoot>
</table>

<style>
  tfoot td { font-weight: 600; border-bottom: none; }
  .within td { color: var(--muted); font-size: 0.9em; }
  .within td:first-child { padding-left: 1.5rem; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
