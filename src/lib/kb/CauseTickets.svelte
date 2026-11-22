<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import { groupByCause, readAs, uniqueById, type Cause, type Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';

  // One row per cause found in a merged ticket, each with the ticket that now owns it.
  let { ticket, tickets }: { ticket: number; tickets: Partial<Record<Cause, { href: string; label: string }>> } = $props();

  const merged = $derived(uniqueById(reports as Report[]).filter((r) => r.mergedInto === ticket));
  const groups = $derived(groupByCause(merged, tenantOf));
  const rows = (reports as Report[]).length;
</script>

<p class="sum">
  The export holds <strong>{rows}</strong> rows; <strong>{merged.length}</strong> unique reports were merged into
  #{ticket}. They split into <strong>{groups.length}</strong> {groups.length === 1 ? 'problem' : 'problems'}.
</p>

<table>
  <thead><tr><th>problem</th><th>reports</th><th>tenants</th><th>seen</th><th>ticket</th></tr></thead>
  <tbody>
    {#each groups as g (g.cause)}
      {@const t = g.cause === 'unplaced' ? undefined : tickets[g.cause]}
      <tr>
        <td><code>{g.cause}</code></td>
        <td>{g.reports.length}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>{#if t}<a href={t.href}>{t.label}</a>{:else}none yet{/if}</td>
      </tr>
    {/each}
  </tbody>
</table>

{#each groups as g (g.cause)}
  {@const moved = readAs(g, notes.oneDayMove)}
  {#if moved}
    <p class="note">{moved} of the <code>{g.cause}</code> reports read as a one-day move (desk notes). They stay counted under <code>{g.cause}</code>, by tenant.</p>
  {/if}
{/each}

<style>
  .sum { font-size: 1.05rem; }
  .note { color: var(--muted); font-size: 0.85em; }
</style>
