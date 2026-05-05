---
title: "OpenPlural — Adoption guide"
nav_active: adopt
---

<section>
  <h1>Adoption guide</h1>
  <p class="sub"><span class="draft">draft v0.1</span>If you're considering an exporter or importer for your app, this is the page that argues it's worth your time.</p>
  <p>Two apps have committed: <b>Prism</b> and <b>Sheaf</b>. Below are full mapping tables for each app's existing export shape against OpenPlural v0.1, followed by shorter notes for Simply Plural, PluralKit, and Plural Star.</p>
</section>

<section id="legend">
  <div class="section-head">
    <h2>Status legend</h2>
    <p>The same four tags appear in the Status column of every Prism/Sheaf row below.</p>
  </div>
  <div class="panel">
    <p style="margin: 0 0 8px;"><span class="tag tag-direct">direct</span> The source field maps 1:1 to an OpenPlural field with the same shape.</p>
    <p style="margin: 0 0 8px;"><span class="tag tag-transform">transform</span> Lossless conversion (e.g. enum rename, string-to-array, decryption on export).</p>
    <p style="margin: 0 0 8px;"><span class="tag tag-normalize">normalize</span> Source inlines data that OpenPlural splits into a sibling array (e.g. inline <code>member_ids</code> → <code>group_memberships[]</code>).</p>
    <p style="margin: 0;"><span class="tag tag-extension">extensions.*</span> Source-specific data preserved under a namespaced key. Lossless but opaque to other apps.</p>
  </div>
</section>

<section id="adopters">
  <div class="section-head">
    <h2>Headline adopters</h2>
    <p>About 30 fields each. Most rows are <span class="tag">direct</span>; the interesting cases are <span class="tag">normalize</span> and <span class="tag">extensions.*</span>.</p>
  </div>

  <div class="adopt-grid">

    <article class="adopt-card" id="prism">
      <h3>Prism <span class="who-status">Adopter</span></h3>
      <p class="adopt-sub">
        <code>.prism</code> encrypted JSON envelope (V3 container). Local-first Flutter app.
        Source of truth: <code>lib/features/data_management/models/export_models.dart</code>.
      </p>
      <p class="adopt-sub"><em>Mappings here are maintainer-provided pending a public sample export fixture.</em></p>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Prism field</th><th>OpenPlural target</th><th>Status</th></tr></thead>
          <tbody>
            <tr><td>formatVersion</td><td>(envelope) — replaces with <code>openplural_version</code></td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>appName</td><td>producer.app</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>version</td><td>producer.app_version</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>exportDate</td><td>exported_at</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>systemSettings.systemName</td><td>System.name</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>systemSettings.systemDescription</td><td>System.description</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>systemSettings.accentColor</td><td>System.color</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>systemSettings.avatarData</td><td>Asset (kind: avatar) + System.avatar_asset_id</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>systemSettings.feature toggles + theme</td><td>extensions.prism.settings</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>headmates[].id, name, displayName, pronouns</td><td>Member.id, name, display_name, pronouns</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>headmates[].age, birthday, notes</td><td>Member.age, birthday, description</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>headmates[].profilePhotoData</td><td>Asset (kind: avatar) + Member.avatar_asset_id</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>headmates[].emoji, customColor, isAdmin</td><td>extensions.prism.*</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>headmates[].pluralkitUuid, pluralkitId</td><td>SourceRef (app: "pluralkit")</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>headmates[].proxyTagsJson</td><td>Member.proxy_tags[]</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>headmates[].displayOrder</td><td>Member.sort_order</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>headmates[].parentSystemId</td><td>(via System.parent_system_id)</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>frontSessions[].startTime, endTime</td><td>FrontPeriod.started_at, ended_at</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>frontSessions[].headmateId</td><td>FrontPeriod.assignments[0].member_id</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>frontSessions[].notes, confidence, quality</td><td>FrontAssignment.note, confidence, mood</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>frontSessions[].sessionType (sleep)</td><td>FrontPeriod.status: "sleep" + extensions.prism.sessionType</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>frontSessions[] legacy co-fronter JSON</td><td>extensions.prism.legacyCoFronters</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>sleepSessions[]</td><td>FrontPeriod with status: "sleep"</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>frontComments (newer, time-anchored)</td><td>FrontComment</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>frontComments (legacy, sessionId-anchored)</td><td>FrontComment with front_period_id</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>memberGroups[] (id, name, color, emoji, parentGroupId)</td><td>Group</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>memberGroupEntries[]</td><td>GroupMembership</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>customFields[]</td><td>CustomFieldDefinition</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>customFieldValues[]</td><td>CustomFieldValue</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>notes[]</td><td>Note</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>conversations[]</td><td>chat.conversations[]</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>messages[]</td><td>chat.messages[]</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>mediaAttachments[]</td><td>Asset + chat.attachments[]</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>polls[], pollOptions[]</td><td>polls module (v0.2)</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>habits[], habitCompletions[]</td><td>habits module (v0.2)</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>reminders[]</td><td>reminders module (v0.2)</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>friends[]</td><td>sharing module (v0.2)</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
          </tbody>
        </table>
      </div>
    </article>

    <article class="adopt-card" id="sheaf">
      <h3>Sheaf <span class="who-status">Adopter</span></h3>
      <p class="adopt-sub">
        <code>/v1/export</code> JSON v2, plus an async zip export with image bytes. FastAPI/PostgreSQL
        with application-level encryption (decrypted on export). Source of truth:
        <code>docs/apps/sheaf.md</code>.
      </p>
      <div class="table-wrap">
        <table>
          <thead><tr><th>Sheaf field</th><th>OpenPlural target</th><th>Status</th></tr></thead>
          <tbody>
            <tr><td>version: "2"</td><td>(envelope) → openplural_version: "0.1"</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>system.id, name, description, tag</td><td>System.id, name, description, tag</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>system.avatar_url</td><td>Asset (uri-only) + System.avatar_asset_id</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>system.color</td><td>System.color</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>system.privacy</td><td>System.privacy.visibility</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>system.date_format, replace_fronts_default</td><td>extensions.sheaf.*</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>system.delete_confirmation, safety, retention</td><td>safety module (when v0.2 lands); otherwise <code>extensions.sheaf.*</code></td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>members[].id</td><td>Member.id</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].name (decrypted)</td><td>Member.name</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].display_name</td><td>Member.display_name</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].description (decrypted)</td><td>Member.description</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].pronouns</td><td>Member.pronouns</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].avatar_url</td><td>Asset + Member.avatar_asset_id</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>members[].color</td><td>Member.color</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].birthday</td><td>Member.birthday.value (Sheaf stores either <code>"MM-DD"</code> or <code>"YYYY-MM-DD"</code>; derive <code>precision</code> + <code>year_visible</code> accordingly)</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>members[].privacy</td><td>Member.privacy.visibility</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>members[].created_at</td><td>Member.created_at</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>fronts[].id, started_at, ended_at</td><td>FrontPeriod.id, started_at, ended_at</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>fronts[].member_ids</td><td>FrontPeriod.assignments[] (one per id, role: "member")</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>groups[].id, name, description, color</td><td>Group.id, name, description, color</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>groups[].parent_id</td><td>Group.parent_group_id</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>groups[].member_ids</td><td>GroupMembership[] (one per id)</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>tags[].id, name, color</td><td>TaxonomyTerm.id, name, color (kind: "tag")</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>tags[].member_ids</td><td>TaxonomyAssignment[] (subject_type: "member")</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>custom_fields[].id, name, field_type, options, order, privacy</td><td>CustomFieldDefinition (options accepts <code>string[] | Record&lt;string, unknown&gt; | null</code> since Sheaf stores it as JSONB <code>dict | None</code>)</td><td><span class="tag tag-transform">transform</span></td></tr>
            <tr><td>custom_fields[].values[] (member_id, value)</td><td>CustomFieldValue (subject_type: "member")</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>journals[].id, member_id, title, body, visibility, author_member_ids, created_at, updated_at</td><td>Note</td><td><span class="tag tag-direct">direct</span></td></tr>
            <tr><td>journals[].image_keys + async zip <code>images/&lt;key&gt;</code></td><td>Asset + Note.attachment_asset_ids</td><td><span class="tag tag-normalize">normalize</span></td></tr>
            <tr><td>sync <code>uploaded_files[]</code> inventory without bytes</td><td><code>extensions.sheaf.uploaded_files</code> unless paired with the async zip</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>revisions[] (journal/member-bio edit history)</td><td><code>extensions.sheaf.revisions</code> until OpenPlural grows a revision-history shape</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
            <tr><td>watch_tokens[] + channels[]</td><td><code>extensions.sheaf.watch_tokens</code> until OpenPlural grows a notifications/export module</td><td><span class="tag tag-extension">extensions.*</span></td></tr>
          </tbody>
        </table>
      </div>
      <div class="callout mt-14">
        <p>The sync JSON export is now good enough for systems, members, fronts, groups, tags, custom fields, and journals. For portable <code>assets[]</code>, prefer Sheaf's async zip export over bare <code>/v1/export</code>: the zip includes the actual <code>images/&lt;key&gt;</code> blobs, while the sync JSON only has avatar URLs and uploaded-file inventory metadata.</p>
      </div>
    </article>

  </div>
</section>

<section id="other-paths">
  <div class="section-head">
    <h2>Adoption notes for other apps</h2>
    <p>Shorter pointers — full mapping tables come once each app commits to OpenPlural support.</p>
  </div>
  <div class="module-grid">
    <article class="module-card">
      <h3>Simply Plural</h3>
      <p>Pre-discontinuation export. Custom fronts → <code>Member.is_custom_front: true</code>. Privacy buckets preserved under <code>privacy.source.simply_plural</code>. Chat decrypted on export. <code>channels</code>+<code>chatMessages</code> map into the chat module; <code>boardMessages</code> map into the boards module as <code>BoardPost</code>.</p>
    </article>
    <article class="module-card">
      <h3>PluralKit</h3>
      <p>Switch logs → <code>front_events[]</code>; periods optionally derived. Flat groups; rich per-record privacy preserved on the privacy fragment. Proxy tags map to <code>Member.proxy_tags[]</code>; autoproxy/account links wait for the <code>proxy</code> module in v0.2.</p>
    </article>
    <article class="module-card">
      <h3>Plural Star</h3>
      <p>Tiered fronting → <code>FrontPeriod</code> with <code>front_role</code> per tier and <code>source_kind: "tiered"</code>. Channels map to the chat module; noteboards map to the boards module as <code>BoardPost</code> with <code>pinned</code>. Filesystem backup format v1.2 maps cleanly to v0.1 core.</p>
    </article>
  </div>
</section>

<section id="guidance">
  <div class="section-head">
    <h2>Maintainer guidance</h2>
    <p>Four things worth getting right when you build an exporter or importer.</p>
  </div>
  <div class="maintainer-grid">
    <div class="maintainer-card">
      <b>One exporter beats N converters</b>
      <p>If you map your internal records to OpenPlural's core, you're done — every other app's importer handles the rest. Pairwise converters are how you end up maintaining nine of them.</p>
    </div>
    <div class="maintainer-card">
      <b>Partial imports are still wins</b>
      <p>If your app doesn't have chat, polls, custom fields, or groups, importing the rest and reporting the skip is much better than refusing the file. The user's other data still moves.</p>
    </div>
    <div class="maintainer-card">
      <b>Provenance survives migrations</b>
      <p>Original app IDs in <code>source_refs</code> are what lets users (and future sync tools) reconcile records after a round-trip. Without them, every import looks like a fresh dataset.</p>
    </div>
    <div class="maintainer-card">
      <b>Extensions are cheap insurance</b>
      <p>Source-specific fields under namespaced <code>extensions</code> cost almost nothing to write but preserve data the user might want later — including data another app may eventually understand.</p>
    </div>
  </div>
  <div class="panel mt-24">
    <div class="next-step">
      <div class="step-number">01</div>
      <div>
        <b>Adoption path</b>
        <p class="muted-copy">
          Start with exporter support for systems, members, front history, groups, custom fields, notes, and assets.
          Add taxonomy when your app has role-like labels, tags, or reusable profile categories.
          Treat chat, polls, reminders, proxy settings, and friend sharing as optional modules —
          you can preserve them even when the target app won't show them.
        </p>
      </div>
    </div>
  </div>
  <div class="panel mt-14">
    <div class="next-step">
      <div class="step-number">02</div>
      <div>
        <b>Reporting</b>
        <p class="muted-copy">
          Always emit an <a href="spec-modules.html#importresult">ImportResult</a> after a run.
          Compare "247 imported, 12 degraded, 158 retained as archive data, 0 failed" to "import complete."
          The first one tells the user what actually happened.
        </p>
      </div>
    </div>
  </div>
</section>
