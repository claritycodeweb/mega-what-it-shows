<script lang="ts">
  import reports from '#lib/data/reports-4471.json';
  import notes from '#lib/data/desk-notes.json';
  import type { Report } from '#lib/kb/causes';
  import { tenantOf } from '#lib/kb/locales';
  import { splitTicket, type Problem } from '#lib/kb/problems';

  // issues: where each problem is tracked now, e.g. { parser: 'https://…/issues/1' }.
  let { ticket, issues = {} }: { ticket: number; issues?: Partial<Record<Problem, string>> } = $props();

  const split = $derived(splitTicket(reports as Report[], ticket, tenantOf, notes.oneDayMove));
  const filed = $derived(split.groups.filter((g) => g.problem !== 'one-day').reduce((n, g) => n + g.reports.length, 0));
  const oneDay = $derived(split.groups.find((g) => g.problem === 'one-day')?.reports.length ?? 0);
  const channels = (c: Record<string, number>) =>
    Object.entries(c)
      .sort((a, b) => b[1] - a[1])
      .map(([ch, n]) => `${ch} ×${n}`)
      .join(' · ');
  const issueNo = (url: string) => '#' + url.split('/').pop();
</script>

<p class="sum">
  <strong>{split.merged.length}</strong> reports merged into #{ticket} ·
  <strong>{split.groups.length}</strong>
  {split.groups.length === 1 ? 'problem' : 'problems'}
</p>

<table>
  <thead>
    <tr><th>problem</th><th>reports</th><th>channels</th><th>tenants</th><th>seen</th><th>ticket</th></tr>
  </thead>
  <tbody>
    {#each split.groups as g (g.problem)}
      <tr>
        <td><code>{g.problem}</code></td>
        <td>{g.reports.length}</td>
        <td class="muted">{channels(g.channels)}</td>
        <td>{g.tenants}</td>
        <td>{g.firstSeen} → {g.lastSeen}</td>
        <td>
          {#if issues[g.problem]}<a href={issues[g.problem]}>{issueNo(issues[g.problem]!)}</a>{:else}<span class="warn">no ticket</span>{/if}
        </td>
      </tr>
    {/each}
  </tbody>
</table>

<p class="muted">
  {filed === split.merged.length ? `Every report is filed under one cause: ${filed} of ${split.merged.length}.` : `Filed ${filed} of ${split.merged.length}: check the split.`}
  {#if oneDay}The {oneDay} <code>one-day</code> reports are also counted under the cause they are filed under, as the desk counts them.{/if}
</p>

<style>
  .sum { font-size: 1.1rem; }
  .muted { color: var(--muted); font-size: 0.85em; }
  .warn { color: var(--warn); font-weight: 600; }
</style>
