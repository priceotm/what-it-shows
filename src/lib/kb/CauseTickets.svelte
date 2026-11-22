<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, uniqueById, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per problem found in a merged ticket, each with the ticket that now owns it.
  // causes.ts sorts reports by tenant. Reports the desk read as a one-day move (desk-notes.json)
  // describe a different bug from their tenant's cause, so they get a row of their own here.
  type Link = { href: string; label: string };
  let { ticket, tickets }: { ticket: number; tickets: Record<string, Link> } = $props();

  const ONE_DAY = 'one-day';
  const moved = new Set(notes.oneDayMove);
  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const rows = $derived.by(() => {
    const byCause = groupByCause(merged.filter((r) => !moved.has(r.id)), tenantOf).map((g) => ({
      problem: String(g.cause),
      reports: g.reports,
    }));
    const oneDay = merged.filter((r) => moved.has(r.id));
    return [...byCause, ...(oneDay.length ? [{ problem: ONE_DAY, reports: oneDay }] : [])].map((g) => {
      const days = g.reports.map((r) => r.receivedAt.slice(0, 10)).sort();
      return { ...g, tenants: new Set(g.reports.map((r) => r.tenant)).size, first: days[0], last: days[days.length - 1] };
    });
  });
  const total = (reports as Report[]).length;
</script>

<p class="sum">
  The export holds <strong>{total}</strong> rows; <strong>{merged.length}</strong> unique reports were merged into
  #{ticket}. They split into <strong>{rows.length}</strong> {rows.length === 1 ? 'problem' : 'problems'}.
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each rows as g (g.problem)}
      {@const t = tickets[g.problem]}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.first} → {g.last}</td>
        <td>{#if t}<a href={t.href}>{t.label}</a>{:else}none yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

<style>
  .sum { font-size: 1.05rem; }
</style>
