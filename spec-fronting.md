---
title: "OpenPlural Spec — Fronting"
nav_active: spec
---

<section>
  <h1>Fronting model</h1>
  <p class="sub"><span class="draft">draft v0.1</span>The hardest cross-app translation. We support both intervals (periods) and point-in-time switches (events) so apps that store fronting differently can round-trip without lying about what changed.</p>
  <p>See <a href="spec-records.html">core records</a> for member shapes referenced here.</p>

  <details class="toc">
    <summary>on this page</summary>
    <ol>
      <li><a href="#patterns">How apps store fronting</a></li>
      <li><a href="#frontperiod">FrontPeriod</a></li>
      <li><a href="#frontassignment">FrontAssignment</a></li>
      <li><a href="#frontevent">FrontEvent</a></li>
      <li><a href="#frontcomment">FrontComment</a></li>
      <li><a href="#conversion">Conversion notes</a></li>
    </ol>
  </details>
</section>

<div class="spec-page mt-24">
  <div>
    <section id="patterns">
      <div class="section-head">
        <h2>How apps store fronting</h2>
        <p>Every app we've looked at falls into one of these four.</p>
      </div>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Source pattern</th><th>Examples</th><th>Best OpenPlural target</th></tr></thead>
          <tbody>
            <tr><td>Switch events</td><td>PluralKit</td><td><code>front_events[]</code> directly. Periods can be derived from adjacent events if needed.</td></tr>
            <tr><td>Per-member intervals</td><td>Prism, Simply Plural, OpenSelves, Ampersand, PluralSpace</td><td>One <code>FrontPeriod</code> per row with a single <code>FrontAssignment</code>. PluralSpace stores co-fronting as multiple rows sharing identical <code>started_at</code>/<code>ended_at</code> — group by timestamps to reconstruct the period.</td></tr>
            <tr><td>Grouped intervals (co-fronting)</td><td>Sheaf</td><td>One <code>FrontPeriod</code> per row, one <code>FrontAssignment</code> per <code>member_id</code>, all with <code>front_role: "member"</code>.</td></tr>
            <tr><td>Tiered (primary / co-front / co-conscious)</td><td>Plural Star</td><td>One <code>FrontPeriod</code> with assignments using the matching <code>front_role</code> per tier.</td></tr>
          </tbody>
        </table>
      </div>
    </section>

    <section id="records">
      <div class="section-head">
        <h2>Records</h2>
      </div>

      <article class="record" id="frontperiod">
        <h3>FrontPeriod</h3>
        <p class="record-blurb">An interval during which one or more members were fronting. Most apps' native shape.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>started_at</td><td>ISO8601</td><td class="req-yes">yes</td><td>Sheaf: <code>fronts[].started_at</code>. Prism: <code>frontSessions[].startTime</code>.</td></tr>
              <tr><td>ended_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Null = currently fronting. Sheaf: <code>fronts[].ended_at</code> (nullable).</td></tr>
              <tr><td>assignments</td><td>FrontAssignment[]</td><td class="req-yes">yes</td><td>At least one. See <a href="#frontassignment">FrontAssignment</a>.</td></tr>
              <tr><td>status</td><td>string | null</td><td class="req-no">no</td><td>Optional period-level status — e.g. <code>"sleep"</code>, <code>"masking"</code>. Apps without this leave null.</td></tr>
              <tr><td>note</td><td>string | null</td><td class="req-no">no</td><td>Whole-period note — distinct from per-assignment <code>note</code>.</td></tr>
              <tr><td>source_kind</td><td>"interval" | "event_pair" | "tiered" | "grouped" | "unknown"</td><td class="req-no">no</td><td>How this period was derived. Helps importers reconstruct source semantics.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "id": "front_01HV4Z...",
  "system_id": "sys_01HV4Z...",
  "started_at": "2026-04-29T12:00:00Z",
  "ended_at": "2026-04-29T15:00:00Z",
  "assignments": [
    { "member_id": "mem_01HV4Z...", "front_role": "primary",  "note": null },
    { "member_id": "mem_01HV5A...", "front_role": "co_front", "note": null }
  ],
  "status": null,
  "note": null,
  "source_kind": "tiered",
  "source_refs": [{ "app": "plural_star", "collection": "frontPeriods", "id": "p_..." }],
  "extensions": {}
}</code></pre>
      </article>

      <article class="record" id="frontassignment">
        <h3>FrontAssignment</h3>
        <p class="record-blurb">A single member's role within a period. Sub-record on <a href="#frontperiod">FrontPeriod</a> and <a href="#frontevent">FrontEvent</a>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>member_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>members[].id</code>.</td></tr>
              <tr><td>front_role</td><td>string</td><td class="req-yes">yes</td><td>See enum below. Apps with flat co-fronting use <code>"member"</code>; Sheaf <code>fronts[].member_ids</code> all become <code>"member"</code>.</td></tr>
              <tr><td>confidence</td><td>number | null</td><td class="req-no">no</td><td>0–1. Prism: <code>frontSessions[].confidence</code>.</td></tr>
              <tr><td>presence</td><td>"present" | "background" | "muted" | "asleep" | string | null</td><td class="req-no">no</td><td>Ampersand-style presence metadata.</td></tr>
              <tr><td>mood</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>energy</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>location</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>note</td><td>string | null</td><td class="req-no">no</td><td>Per-assignment note. Prism: <code>frontSessions[].notes</code>.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="enum-sidebar">
          <b>Recommended <code>front_role</code> values</b>
          <code>"primary" | "co_front" | "co_conscious" | "influencing" | "member" | "custom_status" | "unknown"</code>
          <p style="margin: 6px 0 0; color: var(--muted); font-size: 0.82rem;">"front_role" describes a fronting tier within this specific period. It's not a member profile role — those belong in <a href="spec-records.html#taxonomyterm">TaxonomyTerm</a>.</p>
        </div>
      </article>

      <article class="record" id="frontevent">
        <h3>FrontEvent</h3>
        <p class="record-blurb">A point-in-time switch. PluralKit's switch log primary target. Periods can be derived from sequential events.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>at</td><td>ISO8601</td><td class="req-yes">yes</td><td>The instant of the switch.</td></tr>
              <tr><td>assignments</td><td>FrontAssignment[]</td><td class="req-yes">yes</td><td>The members fronting <em>after</em> this switch. PluralKit: <code>switches[].members</code>. May be empty for switch-out / no-front events (PluralKit pattern).</td></tr>
              <tr><td>note</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "id": "event_01HV4Z...",
  "system_id": "sys_01HV4Z...",
  "at": "2026-04-29T12:00:00Z",
  "assignments": [{ "member_id": "mem_01HV4Z...", "front_role": "member" }],
  "note": null,
  "source_refs": [{ "app": "pluralkit", "collection": "switches", "id": "abcde" }],
  "extensions": {}
}</code></pre>
      </article>

      <article class="record" id="frontcomment">
        <h3>FrontComment</h3>
        <p class="record-blurb">A comment anchored to a moment in time, optionally linked to a period. Time anchoring lets Prism's newer comments and Simply Plural's front-history comments coexist.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>front_period_id</td><td>UUID | null</td><td class="req-no">no</td><td>Optional link to a specific period.</td></tr>
              <tr><td>front_event_id</td><td>UUID | null</td><td class="req-no">no</td><td>Optional link to a specific event.</td></tr>
              <tr><td>target_time</td><td>ISO8601</td><td class="req-yes">yes</td><td>The instant the comment is "about". Prism: <code>frontComments.targetTime</code>.</td></tr>
              <tr><td>author_member_id</td><td>UUID | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>body</td><td>string</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>created_at</td><td>ISO8601</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>edited_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>
    </section>

    <section id="conversion">
      <div class="section-head">
        <h2>Conversion notes</h2>
        <p>Per-app instructions. If you're writing an exporter, your app is probably one of these.</p>
      </div>
      <ul class="proposal-list">
        <li><b>Sheaf</b> exporters: emit <code>front_periods[]</code> only; one assignment per <code>member_id</code> with <code>front_role: "member"</code>; <code>source_kind: "grouped"</code>.</li>
        <li><b>Prism</b> exporters: emit <code>front_periods[]</code> with single-member assignments. Overlapping periods are valid for co-fronting. Sleep is exported as a separate period with <code>status: "sleep"</code> or via <code>extensions.prism.sessionType</code>.</li>
        <li><b>PluralKit</b> exporters: emit <code>front_events[]</code> from the switch log. Optionally derive <code>front_periods[]</code> for importers that need durations.</li>
        <li><b>Plural Star</b> exporters: emit <code>front_periods[]</code> with explicit <code>front_role</code> per tier (<code>primary</code>, <code>co_front</code>, <code>co_conscious</code>); <code>source_kind: "tiered"</code>.</li>
        <li><b>Simply Plural / PluralBridge importer note</b>: Joshua Reid, developer of Lighthouse-DID Hub, reported importer edge cases to PluralBridge while testing Simply Plural / PluralBridge-shaped data. One important case is current fronting with multiple open entries. This matches observed Simply Plural behavior: more than one member may be fronting at the same time, and current fronters should be reconstructed from all front-history records with a start time and no end time. Importers should preserve all such open entries rather than collapsing them to a single current fronter.</li>
        <li>Importers without per-period roles should display the <code>primary</code> assignment first if present, otherwise the first assignment.</li>
        <li>Importers without comments should preserve them in archive form and emit a <code>warning</code> with code <code>"front_comments_archived"</code>.</li>
      </ul>
    </section>

    <div class="callout mt-24">
      <p>Continue to <a href="spec-modules.html">optional modules</a> for chat, polls, and the importer contract — or back to <a href="spec-records.html">core records</a>.</p>
    </div>
  </div>

</div>
