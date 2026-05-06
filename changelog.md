---
title: "OpenPlural — Changelog"
nav_active: changelog
---

<section>
  <h1>Changelog</h1>
  <p class="sub"><span class="draft">draft v0.1</span>Editorial and schema-shape changes within the current draft. The displayed draft label stays at <code>draft v0.1</code> until the interchange-compatibility boundary changes materially.</p>
  <p>This page tracks meaningful spec revisions without implying a new wire-format version for every field rename or modeling correction.</p>
</section>

<section id="versioning-policy">
  <div class="section-head">
    <h2>Versioning policy</h2>
    <p>Short-term working rule while the spec is still a draft.</p>
  </div>
  <div class="panel">
    <p style="margin: 0 0 8px;"><code>openplural_version</code> is the interchange version. It should change when compatibility expectations or top-level format semantics change materially.</p>
    <p style="margin: 0 0 8px;">The site-wide label stays at <code>draft v0.1</code> while the current draft line is still being shaped.</p>
    <p style="margin: 0;">The changelog records design corrections, clarified constraints, and module-level decisions inside the current draft.</p>
  </div>
</section>

<section id="entries">
  <div class="section-head">
    <h2>Entries</h2>
    <p>Newest first.</p>
  </div>

  <article class="record">
    <h3>2026-05-06</h3>
    <p class="record-blurb">Conversation access in the chat module is now explicit instead of inferred from an empty participant list.</p>
    <ul>
      <li><code>Conversation.access</code> was added as a required discriminated union with <code>{ kind: "all_members" }</code> and <code>{ kind: "participants", member_ids: [...] }</code>.</li>
      <li><code>participant_member_ids</code> was removed from the core <code>Conversation</code> shape in favor of <code>access</code> as the sole authoritative membership/access signal.</li>
      <li><code>direct_message</code> conversations now explicitly require <code>access.kind: "participants"</code>.</li>
      <li>The old rule that <code>participant_member_ids: []</code> means a system-wide room was removed from the prose and examples.</li>
      <li>App-specific admin or moderator permission overrides are now documented as extension-space concerns rather than core shared-model semantics.</li>
    </ul>
  </article>
</section>
