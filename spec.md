---
title: "OpenPlural Draft API Spec"
nav_active: spec
---

<section>
  <h1>Spec hub</h1>
  <p class="sub"><span class="draft">draft v0.1</span>What lives on this page: data conventions, the top-level envelope, and the shared fragments (<code>source_refs</code>, <code>privacy</code>, <code>warnings</code>) that show up on most records.</p>

  <p>Field-by-field detail lives on three subpages:</p>
  <ul>
    <li><a href="spec-records.html">Records</a> — 17 core records with field tables.</li>
    <li><a href="spec-fronting.html">Fronting</a> — periods, events, comments, assignments.</li>
    <li><a href="spec-modules.html">Modules &amp; contract</a> — chat, boards, relationships, optional modules, importer contract.</li>
  </ul>
</section>

<section id="conventions">
  <div class="section-head">
    <h2>Data conventions</h2>
    <p>Six rules. Skim once; they're assumed everywhere else.</p>
  </div>
  <div class="module-grid">
    <article class="module-card">
      <h3>JSON canonical</h3>
      <p>The interchange format is JSON. Not doing protobuf or CBOR in v0.1 — every researched app already speaks JSON, nothing else does.</p>
    </article>
    <article class="module-card">
      <h3>UTC timestamps</h3>
      <p>Timestamps are ISO-8601 in UTC. Apps with source timezones can preserve them in <code>extensions</code>.</p>
    </article>
    <article class="module-card">
      <h3>File-local IDs</h3>
      <p>Every record has a file-local <code>id</code>. Prefer UUIDv7 or ULID; importers must not assume an algorithm.</p>
    </article>
    <article class="module-card">
      <h3>Source preservation</h3>
      <p>Original IDs go in <code>source_refs</code>. App-specific fields go in namespaced <code>extensions</code>.</p>
    </article>
    <article class="module-card">
      <h3>Modules are optional</h3>
      <p>Importers should accept any subset of modules. The <code>capabilities.modules</code> array declares what a file intends to populate, but present data still counts.</p>
    </article>
    <article class="module-card">
      <h3>Loss is reported</h3>
      <p>Skipped or degraded data must surface in <code>warnings</code> at export time and in the import result at import time.</p>
    </article>
  </div>
</section>

<section id="bundle">
  <div class="section-head">
    <h2>Bundle convention</h2>
    <p>How a self-contained OpenPlural export travels when it needs binary files.</p>
  </div>
  <div class="panel">
    <p style="margin: 0 0 10px;">
      OpenPlural may be delivered as a bare JSON document, or as a ZIP bundle with the canonical
      extension <code>.pluralport.zip</code>. Importers must also accept the legacy
      <code>.openplural.zip</code> extension. They may accept <code>.openplural</code> as an additional
      compatibility alias, but must identify and validate the content rather than trust the filename.
      New exporters should emit <code>.pluralport.zip</code>; existing exporters may continue to emit
      <code>.openplural.zip</code> while applications transition. Both extensions carry the same v0.1
      bundle and do not rename its wire identifiers.
      A bundle must put the envelope at the ZIP root as <code>openplural.json</code>. Binary files
      referenced by <a href="spec-records.html#asset">Asset</a> records should live under
      <code>assets/</code>, with each record's <code>bundle_path</code> pointing at the exact ZIP entry.
    </p>
    <p style="margin: 0 0 10px;">
      A <code>README.txt</code> at the ZIP root is recommended so users can identify the file without
      tooling. A bundle may also include an empty <code>openplural-version-0.1</code> marker as a cheap
      detection hint, but importers must still validate <code>openplural_version</code> in the envelope.
      Extra root files such as an app-specific <code>manifest.json</code> are allowed, but v0.1 does not
      standardize a manifest schema: <code>openplural.json</code> remains the source of truth.
    </p>
    <p style="margin: 0;">
      Exporters should write <code>assets/</code>, not <code>media/</code>. Importers may resolve a safe,
      explicitly referenced <code>media/</code> path from an older or provisional bundle, but must not infer
      assets merely because a <code>media/</code> directory exists. Earlier bundle discussion used
      <code>export.json</code> and <code>Asset.path</code>; v0.1 standardizes <code>openplural.json</code> and
      <code>Asset.bundle_path</code> to match shipped envelopes and distinguish an archive entry from a URI.
      Until Sheaf migrates to the core field, importers should also recognize its current
      <code>extensions.sheaf.bundle_path</code> using the same path-safety rules.
      The v0.1 ZIP is plaintext and does not define an encrypted wrapper; producers must say so clearly when
      handling sensitive exports.
    </p>
  </div>
</section>

<section id="envelope">
  <div class="section-head">
    <h2>Top-level envelope</h2>
    <p>What goes at the top of every file. Producer info, a <code>capabilities</code> hint for quick previews, and the slots that hold each module's records.</p>
  </div>
  <div class="code-grid">
    <div class="schema-map" aria-label="Envelope module map">
      <div class="schema-nodes">
        <div class="schema-node core">
          <b>Core arrays</b>
          <span><code>systems</code>, <code>members</code>, <code>groups</code>, <code>group_memberships</code>, <code>taxonomy_terms</code>, <code>taxonomy_assignments</code>, <code>custom_fields</code>, <code>custom_field_values</code>, <code>notes</code>, <code>assets</code>.</span>
        </div>
        <div class="schema-node front">
          <b>Fronting arrays</b>
          <span><code>front_periods</code>, <code>front_events</code>, <code>front_comments</code>.</span>
        </div>
        <div class="schema-node optional">
          <b>Optional modules</b>
          <span><code>chat</code>, <code>boards</code>, <code>relationships</code>, <code>polls</code>, <code>reminders</code>, <code>habits</code>, <code>proxy</code>, <code>sharing</code>, <code>safety</code>.</span>
        </div>
        <div class="schema-node extension">
          <b>Preservation</b>
          <span><code>source_refs</code>, <code>extensions</code>, <code>warnings</code>, raw app IDs.</span>
        </div>
      </div>
    </div>
    <pre><code>{
  "openplural_version": "0.1",
  "exported_at": "2026-04-29T18:00:00Z",
  "producer": {
    "app": "Sheaf",
    "app_version": "1.4.2",
    "exporter_version": "0.1.0",
    "app_id": "sheaf"
  },
  "capabilities": {
    "modules": [
"systems", "members", "groups", "taxonomy",
"custom_fields", "front_periods", "notes", "assets"
    ]
  },
  "systems": [],
  "members": [],
  "groups": [],
  "group_memberships": [],
  "taxonomy_terms": [],
  "taxonomy_assignments": [],
  "custom_fields": [],
  "custom_field_values": [],
  "front_periods": [],
  "front_events": [],
  "front_comments": [],
  "notes": [],
  "assets": [],
  "chat": null,
  "extensions": {},
  "warnings": []
}</code></pre>
  </div>
</section>

<section id="shared-records">
  <div class="section-head">
    <h2>Shared record fragments</h2>
    <p>These can appear on any record. They're the main mechanism for safe conversion and future recovery. Full field tables in <a href="spec-records.html#envelope-helpers">records</a>.</p>
  </div>
  <div class="table-wrap">
    <table>
      <thead><tr><th>Fragment</th><th>Shape</th><th>Purpose</th></tr></thead>
      <tbody>
        <tr><td><code>source_refs</code></td><td><code>SourceRef[]</code></td><td>Retains original app IDs for reconciliation, round-trips, dedupe, and import auditing.</td></tr>
        <tr><td><code>extensions</code></td><td><code>Record&lt;string, unknown&gt;</code></td><td>Preserves source-specific fields without forcing every app to understand them. App-specific data should sit under a namespace key, e.g. <code>extensions.sheaf</code> or <code>extensions.prism</code>.</td></tr>
        <tr><td><code>privacy</code></td><td><code>Privacy</code></td><td>A conservative common privacy level plus the original source detail.</td></tr>
        <tr><td><code>warnings</code></td><td><code>Warning[]</code></td><td>Documents skipped, degraded, preserved-only, or repaired data.</td></tr>
      </tbody>
    </table>
  </div>
</section>

<section id="extension-ids">
  <div class="section-head">
    <h2>Registered app IDs</h2>
    <p>Use these short IDs in <code>SourceRef.app</code> and <code>extensions</code> namespaces.</p>
  </div>
  <div class="panel">
    <p style="margin: 0 0 8px;">
      <code>prism</code>, <code>sheaf</code>, <code>simply_plural</code>, <code>pluralkit</code>,
      <code>octocon</code>, <code>plural_star</code>, <code>lighthouse</code>, <code>openselves</code>,
      <code>ampersand</code>, <code>pluralspace</code>, <code>tupperbox</code>.
    </p>
    <p style="margin: 0;">
      New IDs are registered by PR to the OpenPlural repo — maintainers keep the canonical list. Apps
      that want a private namespace without registration can use reverse-DNS keys (e.g.
      <code>com.example.app</code>) inside <code>extensions</code> instead. The <code>openplural</code>
      namespace is reserved for spec-level extensions. Extension values should be JSON objects; use
      <code>extensions.pluralspace.moods</code> rather than a compound key like
      <code>"pluralspace:moods"</code>.
    </p>
  </div>
</section>
