---
title: "PluralPort Spec — Records"
nav_active: spec
---

<section>
  <h1>Core records</h1>
  <p class="sub"><span class="draft">draft v0.1</span>17 records, each with a field table. Types are TypeScript-flavored and JSON-compatible.</p>
  <p>Sister pages: <a href="spec-fronting.html">fronting</a>, <a href="spec-modules.html">optional modules</a>.</p>

  <details class="toc">
    <summary>records on this page</summary>
    <ol>
      <li><a href="#producer">Producer</a></li>
      <li><a href="#capabilities">Capabilities</a></li>
      <li><a href="#sourceref">SourceRef</a></li>
      <li><a href="#privacy">Privacy</a></li>
      <li><a href="#warning">Warning</a></li>
      <li><a href="#system">System</a></li>
      <li><a href="#member">Member</a></li>
      <li><a href="#birthday">Birthday</a></li>
      <li><a href="#proxytag">ProxyTag</a></li>
      <li><a href="#group">Group</a></li>
      <li><a href="#groupmembership">GroupMembership</a></li>
      <li><a href="#taxonomyterm">TaxonomyTerm</a></li>
      <li><a href="#taxonomyassignment">TaxonomyAssignment</a></li>
      <li><a href="#customfielddefinition">CustomFieldDefinition</a></li>
      <li><a href="#customfieldvalue">CustomFieldValue</a></li>
      <li><a href="#note">Note</a></li>
      <li><a href="#asset">Asset</a></li>
    </ol>
  </details>

  <div class="types-legend" aria-label="Type notation">
    <div><b>string</b> <span>JSON string</span></div>
    <div><b>number</b> <span>JSON number</span></div>
    <div><b>boolean</b> <span>JSON true/false</span></div>
    <div><b>null</b> <span>JSON null</span></div>
    <div><b>UUID</b> <span>file-local id (UUIDv4/v7/ULID)</span></div>
    <div><b>ISO8601</b> <span>UTC timestamp string</span></div>
    <div><b>Date</b> <span>YYYY-MM-DD</span></div>
    <div><b>HexColor</b> <span>#RRGGBB</span></div>
    <div><b>T | null</b> <span>nullable value</span></div>
    <div><b>T[]</b> <span>array of T</span></div>
    <div><b>Record&lt;K, V&gt;</b> <span>open object map</span></div>
    <div><b>"a" | "b"</b> <span>string enum (listed inline)</span></div>
  </div>
</section>

<div class="spec-page mt-24">
  <div>

    <!-- ENVELOPE HELPERS -->
    <section id="envelope-helpers">
      <div class="section-head">
        <h2>Envelope helpers</h2>
        <p>Producer metadata, capabilities declaration, source references, privacy, and warnings — these can appear at the file level or on individual records.</p>
      </div>

      <article class="record" id="producer">
        <h3>Producer</h3>
        <p class="record-blurb">Identifies the app that wrote the export. Sits at the top of the envelope.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>app</td><td>string</td><td class="req-yes">yes</td><td>Display name of the producing app, e.g. <code>"Prism"</code>, <code>"Sheaf"</code>.</td></tr>
              <tr><td>app_version</td><td>string</td><td class="req-no">no</td><td>Producing app's release version.</td></tr>
              <tr><td>exporter_version</td><td>string</td><td class="req-no">no</td><td>Version of the PluralPort exporter implementation, if separate from the app.</td></tr>
              <tr><td>app_id</td><td>string</td><td class="req-no">no</td><td>Short canonical app ID — see registered IDs in <a href="spec.html#extension-ids">spec hub</a>.</td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "app": "Prism",
  "app_version": "3.4.0",
  "exporter_version": "0.1.0",
  "app_id": "prism"
}</code></pre>
      </article>

      <article class="record" id="capabilities">
        <h3>Capabilities</h3>
        <p class="record-blurb">Declares which PluralPort modules this file populates. Lets importers know what to expect without scanning every array.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>modules</td><td>string[]</td><td class="req-yes">yes</td><td>Subset of: <code>"systems"</code>, <code>"members"</code>, <code>"groups"</code>, <code>"taxonomy"</code>, <code>"custom_fields"</code>, <code>"front_periods"</code>, <code>"front_events"</code>, <code>"front_comments"</code>, <code>"notes"</code>, <code>"assets"</code>, <code>"chat"</code>, <code>"boards"</code>, <code>"relationships"</code>, <code>"polls"</code>, <code>"reminders"</code>, <code>"habits"</code>, <code>"proxy"</code>, <code>"sharing"</code>, <code>"safety"</code>.</td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="sourceref">
        <h3>SourceRef</h3>
        <p class="record-blurb">Records the original app's identifier for a converted record. Multiple refs allowed when a record has been round-tripped between apps.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>app</td><td>string</td><td class="req-yes">yes</td><td>Registered app ID, e.g. <code>"prism"</code>, <code>"sheaf"</code>, <code>"simply_plural"</code>, <code>"pluralkit"</code>.</td></tr>
              <tr><td>collection</td><td>string</td><td class="req-yes">yes</td><td>Source-app collection/table name, e.g. <code>"members"</code>, <code>"fronts"</code>.</td></tr>
              <tr><td>id</td><td>string</td><td class="req-yes">yes</td><td>Source-app primary key value (string-coerced).</td></tr>
              <tr><td>uuid</td><td>string | null</td><td class="req-no">no</td><td>Some apps (e.g. PluralKit) expose a separate stable UUID alongside their public ID.</td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>"source_refs": [
  { "app": "sheaf", "collection": "members", "id": "0193..." },
  { "app": "pluralkit", "collection": "members", "id": "abcde", "uuid": "5b2..." }
]</code></pre>
      </article>

      <article class="record" id="privacy">
        <h3>Privacy</h3>
        <p class="record-blurb">A conservative common privacy descriptor plus the original source detail preserved for round-trips.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>visibility</td><td>"public" | "friends" | "private" | "trusted" | "unknown"</td><td class="req-no">no</td><td>Conservative bucket. Importers without matching levels must round to the next-strictest. Sheaf maps from <code>system.privacy</code> directly.</td></tr>
              <tr><td>source</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td>Raw source-app privacy data — buckets, per-relation rules, scopes. Lossless preservation.</td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="warning">
        <h3>Warning</h3>
        <p class="record-blurb">Documents loss, degradation, or skipped records. Used both in the envelope's <code>warnings</code> array and in importer results.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>level</td><td>"info" | "warning" | "error"</td><td class="req-yes">yes</td><td>Severity. <code>"error"</code> means data was lost; <code>"warning"</code> means data was degraded or partially preserved.</td></tr>
              <tr><td>code</td><td>string</td><td class="req-yes">yes</td><td>Machine-readable code, e.g. <code>"module_not_supported"</code>, <code>"field_truncated"</code>, <code>"asset_uri_only"</code>.</td></tr>
              <tr><td>record_type</td><td>string | null</td><td class="req-no">no</td><td>The PluralPort record type or path the warning relates to.</td></tr>
              <tr><td>record_id</td><td>UUID | null</td><td class="req-no">no</td><td>Specific record's <code>id</code> if applicable.</td></tr>
              <tr><td>message</td><td>string</td><td class="req-yes">yes</td><td>Human-readable explanation.</td></tr>
              <tr><td>count</td><td>number | null</td><td class="req-no">no</td><td>If the warning aggregates multiple records.</td></tr>
            </tbody>
          </table>
        </div>
      </article>
    </section>

    <!-- SYSTEMS & MEMBERS -->
    <section id="systems-members">
      <div class="section-head">
        <h2>Systems &amp; members</h2>
        <p>The two records every app has. Member fields overlap heavily across apps; system-level fields vary more.</p>
      </div>

      <article class="record" id="system">
        <h3>System</h3>
        <p class="record-blurb">A plurality system's profile and nesting metadata. Multiple systems may appear in one file (Lighthouse, Ampersand, and some user setups support nested systems).</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td>File-local. References target for <code>system_id</code> on other records.</td></tr>
              <tr><td>name</td><td>string</td><td class="req-yes">yes</td><td>System name. Sheaf: <code>system.name</code>. Prism: from <code>systemSettings.systemName</code>.</td></tr>
              <tr><td>display_name</td><td>string | null</td><td class="req-no">no</td><td>Optional alternate display form.</td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td>Markdown allowed.</td></tr>
              <tr><td>tag</td><td>string | null</td><td class="req-no">no</td><td>System tag/suffix. Sheaf: <code>system.tag</code>. PluralKit: <code>system.tag</code>.</td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td>Accent color.</td></tr>
              <tr><td>avatar_asset_id</td><td>UUID | null</td><td class="req-no">no</td><td>References an entry in <code>assets[]</code>.</td></tr>
              <tr><td>banner_asset_id</td><td>UUID | null</td><td class="req-no">no</td><td>References an entry in <code>assets[]</code>.</td></tr>
              <tr><td>parent_system_id</td><td>UUID | null</td><td class="req-no">no</td><td>For nested systems (Lighthouse subsystems, Ampersand nested systems).</td></tr>
              <tr><td>archived</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>.</td></tr>
              <tr><td>privacy</td><td>Privacy</td><td class="req-no">no</td><td>See <a href="#privacy">Privacy</a>.</td></tr>
              <tr><td>settings</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td>App-agnostic system settings; app-specific settings belong in <code>extensions</code>.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td>See <a href="#sourceref">SourceRef</a>.</td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td>App-namespaced raw data preservation.</td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "id": "sys_01HV4Z...",
  "name": "Example System",
  "display_name": null,
  "description": "Markdown or plain text",
  "tag": "| ex",
  "color": "#66ccff",
  "avatar_asset_id": "asset_avatar",
  "banner_asset_id": null,
  "parent_system_id": null,
  "archived": false,
  "privacy": { "visibility": "private", "source": {} },
  "settings": {},
  "source_refs": [{ "app": "sheaf", "collection": "system", "id": "u_..." }],
  "extensions": {
    "sheaf": { "date_format": "ymd", "replace_fronts_default": true }
  }
}</code></pre>
      </article>

      <article class="record" id="member">
        <h3>Member</h3>
        <p class="record-blurb">A member, alter, headmate, custom front, or equivalent profile record. Role-like data lives in <a href="#taxonomyterm">taxonomy</a>, not on this record.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td>File-local.</td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>systems[].id</code>.</td></tr>
              <tr><td>name</td><td>string | null</td><td class="req-no">no</td><td>Sheaf: <code>members[].name</code> (decrypted in export). Optional because some apps (Octocon) allow nameless alters and synthesize a display label. Importers without name handling should derive a display string from <code>display_name</code>, <code>pronouns</code>, or a fallback.</td></tr>
              <tr><td>display_name</td><td>string | null</td><td class="req-no">no</td><td>Sheaf: <code>members[].display_name</code>. Prism: <code>headmates[].displayName</code>.</td></tr>
              <tr><td>pronouns</td><td>string | null</td><td class="req-no">no</td><td>Free text — no enum because every app uses free text.</td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td>Bio. Markdown allowed if <code>extensions.pluralport.markdown_fields</code> says so.</td></tr>
              <tr><td>age</td><td>string | null</td><td class="req-no">no</td><td>Free text (some apps store ranges, "ageless", etc.).</td></tr>
              <tr><td>birthday</td><td>Birthday | null</td><td class="req-no">no</td><td>See <a href="#birthday">Birthday</a>.</td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>avatar_asset_id</td><td>UUID | null</td><td class="req-no">no</td><td>References <code>assets[]</code>. Prism: <code>headmates[].profilePhotoData</code> becomes an Asset.</td></tr>
              <tr><td>banner_asset_id</td><td>UUID | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>proxy_tags</td><td>ProxyTag[]</td><td class="req-no">no</td><td>See <a href="#proxytag">ProxyTag</a>. PluralKit primary source.</td></tr>
              <tr><td>is_custom_front</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>. Used for Simply Plural and Ampersand custom fronts.</td></tr>
              <tr><td>archived</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>.</td></tr>
              <tr><td>created_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Sheaf: <code>members[].created_at</code>. Prism: <code>headmates[].createdAt</code>.</td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td>Lower = earlier in the list. Prism: <code>displayOrder</code>.</td></tr>
              <tr><td>privacy</td><td>Privacy</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "id": "mem_01HV4Z...",
  "system_id": "sys_01HV4Z...",
  "name": "Alex",
  "display_name": "Alex",
  "pronouns": "they/them",
  "description": "Bio",
  "age": null,
  "birthday": { "value": "2001-04-29", "precision": "day", "year_visible": true },
  "color": "#88ccaa",
  "avatar_asset_id": "asset_avatar",
  "banner_asset_id": null,
  "proxy_tags": [{ "prefix": "a:", "suffix": null }],
  "is_custom_front": false,
  "archived": false,
  "created_at": "2026-04-29T18:00:00Z",
  "sort_order": 0,
  "privacy": { "visibility": "private", "source": {} },
  "source_refs": [{ "app": "sheaf", "collection": "members", "id": "0193..." }],
  "extensions": {}
}</code></pre>
      </article>

      <article class="record" id="birthday">
        <h3>Birthday</h3>
        <p class="record-blurb">Birthdays vary by precision: full date, month-day only, year-only, or with year hidden. Sub-record on <a href="#member">Member</a>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>value</td><td>string</td><td class="req-yes">yes</td><td>Date in <code>YYYY-MM-DD</code>, <code>--MM-DD</code>, <code>YYYY</code>, or <code>YYYY-MM</code> form depending on <code>precision</code>.</td></tr>
              <tr><td>precision</td><td>"day" | "month" | "year" | "month_day"</td><td class="req-yes">yes</td><td>Granularity actually known.</td></tr>
              <tr><td>year_visible</td><td>boolean</td><td class="req-no">no</td><td>If <code>false</code>, importers should hide the year in UI even though it's stored. Defaults to <code>true</code>.</td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="proxytag">
        <h3>ProxyTag</h3>
        <p class="record-blurb">PluralKit-style proxy tag. Sub-record on <a href="#member">Member</a>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>prefix</td><td>string | null</td><td class="req-no">no</td><td>Match prefix, e.g. <code>"a:"</code>.</td></tr>
              <tr><td>suffix</td><td>string | null</td><td class="req-no">no</td><td>Match suffix.</td></tr>
            </tbody>
          </table>
        </div>
      </article>
    </section>

    <!-- ORGANIZATION -->
    <section id="organization">
      <div class="section-head">
        <h2>Organization</h2>
        <p>Groups when you want hierarchy or folders. Taxonomy when you want flat labels — roles, tags, sources, relationship categories.</p>
      </div>

      <article class="record" id="group">
        <h3>Group</h3>
        <p class="record-blurb">Hierarchical member organization. Sheaf groups, Prism member groups, Simply Plural groups, PluralKit (flat) groups all map here.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>name</td><td>string</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>emoji</td><td>string | null</td><td class="req-no">no</td><td>Single emoji. Prism: <code>memberGroups.emoji</code>.</td></tr>
              <tr><td>parent_group_id</td><td>UUID | null</td><td class="req-no">no</td><td>Sheaf: <code>groups[].parent_id</code>. Prism: <code>memberGroups.parentGroupId</code>. PluralKit groups are flat — leave null.</td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="callout mt-14"><p><b>Sheaf mapping:</b> Sheaf inlines <code>member_ids</code> on each group. Map those into <a href="#groupmembership">GroupMembership</a> records — one per <code>member_id</code> — instead of duplicating onto the group.</p></div>
      </article>

      <article class="record" id="groupmembership">
        <h3>GroupMembership</h3>
        <p class="record-blurb">Normalized many-to-many edge between groups and members.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>group_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>groups[].id</code>.</td></tr>
              <tr><td>member_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>members[].id</code>.</td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td>Position within the group.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="taxonomyterm">
        <h3>TaxonomyTerm</h3>
        <p class="record-blurb">Reusable label of a given <code>kind</code> — a role, tag, source, relationship, etc. Created once, attached many times via <a href="#taxonomyassignment">assignments</a>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>kind</td><td>string</td><td class="req-yes">yes</td><td>See enum below.</td></tr>
              <tr><td>name</td><td>string</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>parent_term_id</td><td>UUID | null</td><td class="req-no">no</td><td>For nested taxonomies (e.g. tag categories).</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="enum-sidebar">
          <b>Recommended <code>kind</code> values</b>
          <code>"role" | "tag" | "source" | "relationship" | "identity" | "topic" | "status" | "custom" | "unknown"</code>
        </div>
        <div class="callout mt-14"><p><b>Sheaf mapping:</b> Sheaf <code>tags[]</code> become <code>kind: "tag"</code> terms. Sheaf inlines <code>member_ids</code> per tag — those become <a href="#taxonomyassignment">TaxonomyAssignment</a> records with <code>subject_type: "member"</code>.</p></div>
      </article>

      <article class="record" id="taxonomyassignment">
        <h3>TaxonomyAssignment</h3>
        <p class="record-blurb">Attaches a term to a subject record (member, note, asset, etc.).</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>term_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>taxonomy_terms[].id</code>.</td></tr>
              <tr><td>subject_type</td><td>"member" | "note" | "asset" | "front_period" | "custom"</td><td class="req-yes">yes</td><td>Importers must skip assignments whose <code>subject_type</code> they don't support and emit a warning.</td></tr>
              <tr><td>subject_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>scope</td><td>"profile" | "appearance" | "session" | "custom" | null</td><td class="req-no">no</td><td>Optional context — e.g. a role applies to a member's profile, vs. just a particular front.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>
    </section>

    <!-- CUSTOM FIELDS -->
    <section id="custom-fields-sec">
      <div class="section-head">
        <h2>Custom fields</h2>
        <p>Splitting definitions from values lets importers type-check before assigning, and lets one definition cover any number of subjects.</p>
      </div>

      <article class="record" id="customfielddefinition">
        <h3>CustomFieldDefinition</h3>
        <p class="record-blurb">A custom field's name, type, and metadata. Sheaf, Prism, Simply Plural, Ampersand all expose this concept.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>name</td><td>string</td><td class="req-yes">yes</td><td>Display name.</td></tr>
              <tr><td>field_type</td><td>string</td><td class="req-yes">yes</td><td>See enum below.</td></tr>
              <tr><td>options</td><td>string[] | Record&lt;string, unknown&gt; | null</td><td class="req-no">no</td><td>For <code>select</code>/<code>multiselect</code>. Sheaf stores <code>options</code> as JSONB <code>dict | None</code>, so PluralPort accepts either a string array or an object/record verbatim.</td></tr>
              <tr><td>supports_markdown</td><td>boolean</td><td class="req-no">no</td><td>Hint to importers for <code>text</code>/<code>markdown</code> rendering.</td></tr>
              <tr><td>date_precision</td><td>"day" | "month" | "year" | "month_day" | null</td><td class="req-no">no</td><td>For <code>date</code>/<code>date_range</code> types. Prism: <code>customFields.datePrecision</code>.</td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td>Sheaf: <code>custom_fields[].order</code>.</td></tr>
              <tr><td>privacy</td><td>Privacy</td><td class="req-no">no</td><td>Sheaf: <code>custom_fields[].privacy</code>.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="enum-sidebar">
          <b>Recommended <code>field_type</code> values</b>
          <code>"text" | "markdown" | "number" | "boolean" | "color" | "date" | "date_range" | "timestamp" | "month" | "year" | "month_year" | "month_day" | "select" | "multiselect" | "json"</code>
        </div>
      </article>

      <article class="record" id="customfieldvalue">
        <h3>CustomFieldValue</h3>
        <p class="record-blurb">A single value for one definition + one subject. Sheaf nests these inside the definition; PluralPort keeps them in a sibling array for normalization.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>field_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>custom_fields[].id</code>.</td></tr>
              <tr><td>subject_type</td><td>"member" | "system"</td><td class="req-yes">yes</td><td>For v0.1, only members and the system itself. Future expansion possible.</td></tr>
              <tr><td>subject_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>value</td><td>unknown</td><td class="req-yes">yes</td><td>Type matches the field's <code>field_type</code>. For <code>multiselect</code> use a string array; for <code>date_range</code> use <code>{ start, end }</code>; for <code>json</code> use the original payload.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>
    </section>

    <!-- CONTENT -->
    <section id="content">
      <div class="section-head">
        <h2>Content</h2>
        <p>Two records do all the work for free-form text and media: <a href="#note">Note</a> for journals and member notes, <a href="#asset">Asset</a> for anything binary.</p>
      </div>

      <article class="record" id="note">
        <h3>Note</h3>
        <p class="record-blurb">Member notes, journal entries, communal journal entries, and dated entries. Maps cleanly from Prism notes, Simply Plural notes, Plural Star journals, Lighthouse journals, Sheaf journals, Ampersand journal posts.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>member_id</td><td>UUID | null</td><td class="req-no">no</td><td>Primary subject member for per-member entries. Null for system-wide entries; communal or multi-member journals may need a future explicit subject-members field.</td></tr>
              <tr><td>title</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>body</td><td>string</td><td class="req-yes">yes</td><td>Markdown unless flagged otherwise.</td></tr>
              <tr><td>created_at</td><td>ISO8601</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>updated_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>entry_date</td><td>Date | null</td><td class="req-no">no</td><td>For journal-style day entries (Plural Star, Lighthouse).</td></tr>
              <tr><td>author_member_ids</td><td>UUID[]</td><td class="req-no">no</td><td>Co-authored entries (Sheaf journal frozen-author snapshots).</td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td>Prism: <code>notes.color</code>.</td></tr>
              <tr><td>visibility</td><td>"private" | "system" | "friends" | "trusted" | "public" | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>pinned</td><td>boolean</td><td class="req-no">no</td><td></td></tr>
              <tr><td>content_warning</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>attachment_asset_ids</td><td>UUID[]</td><td class="req-no">no</td><td>Inline images/files via <a href="#asset">Asset</a>.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="callout mt-14"><p><b>Open design question:</b> <code>Note</code> currently collapses member notes, per-member journals, and communal journals into one record, with <code>member_id</code> meaning the primary subject member and <code>author_member_ids</code> meaning who wrote it. That's enough for Sheaf's current journal export, but apps like Lighthouse and Octocon suggest a possible future additive field such as <code>subject_member_ids</code> if multi-member journal scoping needs to become first-class. Splitting journals into a wholly separate core record looks less attractive than making note subject-scoping more explicit.</p></div>
      </article>

      <article class="record" id="asset">
        <h3>Asset</h3>
        <p class="record-blurb">An image, file, or media blob. Records reference assets by ID instead of embedding bytes inline. Either inline <code>data_base64</code>/<code>data_uri</code> or external <code>uri</code>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>kind</td><td>"avatar" | "banner" | "image" | "audio" | "video" | "file" | "thumbnail" | "unknown"</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>mime_type</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>file_name</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>uri</td><td>string | null</td><td class="req-no">no</td><td>External URL. Importers should treat as fragile.</td></tr>
              <tr><td>data_base64</td><td>string | null</td><td class="req-no">no</td><td>Base64-encoded payload (no data URI prefix).</td></tr>
              <tr><td>data_uri</td><td>string | null</td><td class="req-no">no</td><td>Full <code>data:&lt;mime&gt;;base64,&lt;...&gt;</code>.</td></tr>
              <tr><td>size_bytes</td><td>number | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>sha256</td><td>string | null</td><td class="req-no">no</td><td>Hex digest. Recommended for dedupe.</td></tr>
              <tr><td>width</td><td>number | null</td><td class="req-no">no</td><td>Pixels.</td></tr>
              <tr><td>height</td><td>number | null</td><td class="req-no">no</td><td>Pixels.</td></tr>
              <tr><td>duration_ms</td><td>number | null</td><td class="req-no">no</td><td>For audio/video.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td>E.g. Prism's <code>blurhash</code>, <code>waveform</code>, encryption metadata.</td></tr>
            </tbody>
          </table>
        </div>
        <div class="callout mt-14"><p>An asset must populate at least one of <code>uri</code>, <code>data_base64</code>, or <code>data_uri</code>. Files with only <code>uri</code> should emit a <code>"warning"</code> with code <code>"asset_uri_only"</code> on export so importers know the asset isn't self-contained.</p></div>
      </article>
    </section>

    <div class="callout mt-24">
      <p>Continue to <a href="spec-fronting.html">fronting</a> for front periods, events, comments, and assignments — or <a href="spec-modules.html">optional modules</a> for chat, polls, and the importer contract.</p>
    </div>
  </div>

</div>
