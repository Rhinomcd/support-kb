<script lang="ts">
  import type { ProblemGroup } from '#lib/kb/split';

  let { groups, tickets }: { groups: ProblemGroup[]; tickets: Partial<Record<ProblemGroup['problem'], string>> } = $props();

  const WHAT: Record<ProblemGroup['problem'], string> = {
    parser: 'shared parser reads the date month first',
    hint: 'booking-v3 hint tells people to type month first',
    shift: 'date moves by one day',
    unplaced: 'no cause: tenant not on file or not day first',
  };
  const total = $derived(groups.reduce((n, g) => n + g.reports.length, 0));
  const ticketNo = (url: string) => `#${url.split('/').pop()}`;
</script>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each groups as g (g.problem)}
      {@const url = tickets[g.problem]}
      <tr class:open={!url}>
        <td><code>{g.problem}</code> {WHAT[g.problem]}</td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if url}<a href={url}>{ticketNo(url)}</a>{:else}none yet{/if}</td>
      </tr>
    {/each}
  </tbody>
  <tfoot>
    <tr><td>total</td><td>{total}</td><td colspan="3"></td></tr>
  </tfoot>
</table>

<style>
  tfoot td { font-weight: 600; border-bottom: none; }
  .open td { background: var(--warn-bg); }
  .open td:last-child { color: var(--warn); font-weight: 600; }
</style>
