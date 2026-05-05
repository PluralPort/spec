---
title: "OpenPlural — Apps & feature matrix"
nav_active: apps
---

<section>
  <h1>Apps &amp; feature matrix</h1>
  <p class="sub">The data shapes of nine plurality apps, side by side. A translation reference, not a ranking. Each row is what an app actually stores; OpenPlural's job is to be a shape all of them can round-trip through.</p>
  <p>Prism and Sheaf have said they'd adopt this. The <a href="adopt.html">adoption guide</a> has their full mapping tables.</p>
</section>

<section id="matrix">
  <div class="section-head">
    <h2>Feature matrix</h2>
    <p>What each app stores, in OpenPlural's module vocabulary. Empty cells are real gaps, not unknowns.</p>
  </div>
  <div class="table-wrap">
    <table>
      <thead>
        <tr>
          <th>App</th>
          <th>Export shape</th>
          <th>Members</th>
          <th>Fronting</th>
          <th>Groups/tags</th>
          <th>Custom fields</th>
          <th>Notes</th>
          <th>Chat/messages</th>
          <th>Assets/privacy</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><b>Prism</b></td>
          <td>Encrypted <code>.prism</code> JSON envelope</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-member intervals plus sleep</td>
          <td>Hierarchical groups</td>
          <td>Definitions and values</td>
          <td>Notes</td>
          <td>Internal chat with media + member board posts</td>
          <td>Assets, friend/share metadata</td>
        </tr>
        <tr>
          <td><b>Sheaf</b></td>
          <td><code>/v1/export</code> JSON v2; async zip backup with images</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Co-front intervals</td>
          <td>Hierarchical groups + tags</td>
          <td>Definitions and values</td>
          <td>Journals + revision history</td>
          <td><span class="tag no">No</span></td>
          <td>Avatar URLs + uploaded-file inventory; privacy, safety, watch-token config</td>
        </tr>
        <tr>
          <td>PluralSpace</td>
          <td>ZIP with manifest + <code>data.json</code> v1.0 (GDPR export)</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-member intervals; co-fronting via overlapping rows</td>
          <td>Nested in app, flat in export (hierarchy dropped)</td>
          <td>Definitions; populated value shape unverified</td>
          <td>Journal entries with numeric visibility</td>
          <td>Channels with embedded messages (no member ID)</td>
          <td><code>media/</code> dir + avatar paths; <code>visibility</code> on system</td>
        </tr>
        <tr>
          <td>Ampersand</td>
          <td>Local JSON backup</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-member intervals with presence metadata</td>
          <td>Typed tags + nested systems</td>
          <td>Definitions, string values on members</td>
          <td>Journal posts</td>
          <td>Board messages</td>
          <td>Optional Data URI files/assets</td>
        </tr>
        <tr>
          <td>Lighthouse</td>
          <td>ZIP of CSVs + narrow token API</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Stubbed in inspected source</td>
          <td>Subsystems as systems</td>
          <td>Fixed rich alter fields</td>
          <td>Journals, communal journals, BDA plans, inner worlds, rules, wishlist</td>
          <td>Forum/thread posts</td>
          <td>Image URLs/blobs, token permissions</td>
        </tr>
        <tr>
          <td>Octocon</td>
          <td>Discord-bot slash export: PK datafile v2 or Octocon-native "full" JSON</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-alter intervals with short comment</td>
          <td>Hierarchical tags (used as groups)</td>
          <td>Definitions on user, string values on alters</td>
          <td>System + per-alter journals (in schema, not in export)</td>
          <td>Discord proxy fields on alter</td>
          <td>Avatar URLs; four-level security + friendships</td>
        </tr>
        <tr>
          <td>OpenSelves</td>
          <td>Sync log DTOs, no export found</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-member intervals</td>
          <td><span class="tag no">No</span></td>
          <td><span class="tag no">No</span></td>
          <td><span class="tag no">No</span></td>
          <td><span class="tag no">No</span></td>
          <td>Member image, account auth</td>
        </tr>
        <tr>
          <td>PluralKit</td>
          <td>API + datafile v2 JSON</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Switch events</td>
          <td>Flat groups</td>
          <td><span class="tag no">No</span></td>
          <td><span class="tag no">No</span></td>
          <td>Discord proxy records (out of core export)</td>
          <td>Avatar/banner URLs, rich privacy</td>
        </tr>
        <tr>
          <td>Plural Star</td>
          <td>Local backup JSON v1.2</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Tiered primary / co-front / co-conscious</td>
          <td>Flat groups</td>
          <td>Definitions and values</td>
          <td>Journal</td>
          <td>Channels/messages + noteboards</td>
          <td>Avatar/banner dictionaries</td>
        </tr>
        <tr>
          <td>Simply Plural</td>
          <td>Mongo collection export + token API</td>
          <td><span class="tag yes">Yes</span></td>
          <td>Per-member/status intervals</td>
          <td>Hierarchical groups</td>
          <td>Definitions, values on members</td>
          <td>Notes</td>
          <td>Chat + board messages</td>
          <td>Avatar URLs, privacy buckets, friends</td>
        </tr>
      </tbody>
    </table>
  </div>
</section>

<section id="apps">
  <div class="section-head">
    <h2>Per-app summaries</h2>
    <p>Prism and Sheaf first because they've committed. PluralSpace next because it has by far the largest install base of the apps still in active use. The rest alphabetical. Each block has a mapping snippet and links to the upstream repo and our research notes.</p>
  </div>

  <article class="app-block panel" id="prism">
    <h3>Prism <span class="who-status">Adopter</span></h3>
    <p class="app-sub">
      Local-first Flutter app with structured database tables and an encrypted <code>.prism</code> export
      envelope (currently V3). Broad in scope: members, fronting, groups, custom fields, chat, member
      board posts, polls, habits, reminders, media, and local sharing.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Prism shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td><code>headmates[]</code></td><td><a href="spec-records.html#member">Member</a></td></tr>
          <tr><td><code>frontSessions[]</code> per-member intervals + <code>sleepSessions[]</code></td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> with single-member assignments; sleep via <code>status: "sleep"</code></td></tr>
          <tr><td><code>memberGroups[]</code> with <code>parentGroupId</code></td><td><a href="spec-records.html#group">Group</a> + <a href="spec-records.html#groupmembership">GroupMembership</a></td></tr>
          <tr><td><code>customFields[]</code> + <code>customFieldValues[]</code></td><td><a href="spec-records.html#customfielddefinition">CustomFieldDefinition</a> + <a href="spec-records.html#customfieldvalue">CustomFieldValue</a></td></tr>
          <tr><td><code>conversations[]</code> + <code>messages[]</code> + <code>mediaAttachments[]</code></td><td><a href="spec-modules.html#conversation">Conversation</a> + <a href="spec-modules.html#chatmessage">ChatMessage</a> + <a href="spec-modules.html#attachment">Attachment</a></td></tr>
          <tr><td><code>member_board_posts</code> (public/private audience, optional <code>targetMemberId</code>); sync-only, not in <code>.prism</code> export envelope</td><td><a href="spec-modules.html#boardpost">BoardPost</a> in the <a href="spec-modules.html#boards">boards module</a></td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="adopt.html#prism">Full Prism mapping</a>
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/prism.md">Research doc</a>
    </div>
  </article>

  <article class="app-block panel" id="sheaf">
    <h3>Sheaf <span class="who-status">Adopter</span></h3>
    <p class="app-sub">
      FastAPI/PostgreSQL app with application-level encryption. Already exposes <code>/v1/export</code>
      returning JSON v2 with system, members, fronts, groups, tags, custom fields, journals,
      revision history, watch-token notification config, and uploaded-file inventory. A separate
      async export job also packages image bytes into a zip.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Sheaf shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td><code>system</code> object (<code>name</code>, <code>tag</code>, <code>privacy</code>, etc.)</td><td><a href="spec-records.html#system">System</a></td></tr>
          <tr><td><code>members[]</code> (decrypted on export)</td><td><a href="spec-records.html#member">Member</a></td></tr>
          <tr><td><code>fronts[]</code> with inline <code>member_ids</code></td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> + one <a href="spec-fronting.html#frontassignment">FrontAssignment</a> per <code>member_id</code></td></tr>
          <tr><td><code>groups[]</code> with <code>parent_id</code> + inline <code>member_ids</code></td><td><a href="spec-records.html#group">Group</a> + <a href="spec-records.html#groupmembership">GroupMembership</a></td></tr>
          <tr><td><code>tags[]</code> with inline <code>member_ids</code></td><td><a href="spec-records.html#taxonomyterm">TaxonomyTerm</a> (<code>kind: "tag"</code>) + <a href="spec-records.html#taxonomyassignment">TaxonomyAssignment</a></td></tr>
          <tr><td><code>custom_fields[]</code> with nested <code>values</code></td><td><a href="spec-records.html#customfielddefinition">CustomFieldDefinition</a> + <a href="spec-records.html#customfieldvalue">CustomFieldValue</a></td></tr>
          <tr><td><code>journals[]</code></td><td><a href="spec-records.html#note">Note</a></td></tr>
          <tr><td><code>revisions[]</code>, <code>watch_tokens[]</code>, sync <code>uploaded_files[]</code></td><td>Best preserved in <code>extensions</code> today; the async zip is the better source for portable <a href="spec-records.html#asset">Asset</a> records</td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="adopt.html#sheaf">Full Sheaf mapping</a>
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/sheaf.md">Research doc</a>
      <a href="https://github.com/sheaf-project/sheaf">Repository</a>
    </div>
  </article>

  <article class="app-block panel" id="pluralspace">
    <h3>PluralSpace</h3>
    <p class="app-sub">
      Web app at <a href="https://pluralspace.app/">pluralspace.app</a>. Separate product from
      Plural Star despite the shared naming history. Current portable surface is a GDPR-style ZIP
      (<code>manifest.json</code> + <code>data.json</code> v1.0); the shape documented here is
      reconstructed from inspected sample exports, not a published schema. The public
      <a href="https://pluralspace.app/developers">developers page</a> lists a REST API as
      "Coming Soon" with a preview, but it isn't a usable surface yet — too early to call it the
      long-term converter target. Per-member front rows (co-fronting via overlapping intervals),
      system-level polls, embedded chat messages, journal entries with a numeric
      <code>visibility_level</code>. Groups nest in the app but appear flat in the GDPR export.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>PluralSpace shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td><code>fronts[]</code>: one row per member, co-fronts share <code>started_at</code>/<code>ended_at</code></td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> built by grouping rows on identical timestamps; one <a href="spec-fronting.html#frontassignment">FrontAssignment</a> per row</td></tr>
          <tr><td><code>members[].role</code> as free-text string array</td><td><a href="spec-records.html#taxonomyterm">TaxonomyTerm</a> (<code>kind: "role"</code>) + <a href="spec-records.html#taxonomyassignment">TaxonomyAssignment</a> per entry</td></tr>
          <tr><td><code>member_groups[]</code> flat (no <code>parent_id</code>); membership denormalized via <code>members[].groups</code> as <em>group names</em> and <code>group.members[]</code> with member IDs</td><td><a href="spec-records.html#group">Group</a> + <a href="spec-records.html#groupmembership">GroupMembership</a>, prefer the ID-side; emit <code>warnings[]</code> with <code>code: "hierarchy_dropped"</code> &mdash; upstream export change needed to recover nesting</td></tr>
          <tr><td><code>journal_entries[]</code> with <code>visibility_level</code> (numeric)</td><td><a href="spec-records.html#note">Note</a>; raw scale preserved in <code>extensions.pluralspace.visibility_level</code></td></tr>
          <tr><td><code>chat_channels[]</code> with embedded messages keyed by <code>member_name</code></td><td>Conversation + ChatMessage; original name preserved in <code>extensions.pluralspace.author_name</code></td></tr>
          <tr><td><code>polls[]</code> system-scoped with embedded <code>options[].votes</code></td><td><code>polls</code> module (post-v0.1)</td></tr>
          <tr><td><code>custom_fields[]</code> definitions; values shape unverified (no populated values seen)</td><td><a href="spec-records.html#customfielddefinition">CustomFieldDefinition</a> + <a href="spec-records.html#customfieldvalue">CustomFieldValue</a> &mdash; needs a sample export with populated values</td></tr>
          <tr><td><code>members[].is_custom_front</code> flag (no instances observed in inspected exports)</td><td>Member with <code>is_custom_front: true</code>; full shape unverified</td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/pluralspace.md">Research doc</a>
      <a href="https://pluralspace.app/">Website</a>
    </div>
  </article>

  <article class="app-block panel" id="ampersand">
    <h3>Ampersand</h3>
    <p class="app-sub">
      Offline-first Tauri/Vue app (alpha). Local backup JSON via <code>exportDatabaseToJSON</code>.
      Per-member intervals with presence metadata; typed tags for members/journals/assets;
      nested systems; optional Data URI–embedded files.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Ampersand shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td>Front entries with presence metadata</td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> with <code>presence</code>/<code>mood</code>/<code>energy</code> on assignments</td></tr>
          <tr><td>Typed tags (members/journals/assets)</td><td><a href="spec-records.html#taxonomyterm">TaxonomyTerm</a> + scoped <a href="spec-records.html#taxonomyassignment">TaxonomyAssignment</a></td></tr>
          <tr><td>Nested systems</td><td><a href="spec-records.html#system">System</a> with <code>parent_system_id</code></td></tr>
          <tr><td>Board messages + polls</td><td><a href="spec-modules.html#boardpost">BoardPost</a> + <code>polls</code> module (poll attached to a board post lives in polls and references the post via <code>extensions</code> until v0.2 formalizes attachment)</td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/ampersand.md">Research doc</a>
      <a href="https://github.com/NyaomiDEV/Ampersand">Repository</a>
    </div>
  </article>

  <article class="app-block panel" id="lighthouse">
    <h3>Lighthouse</h3>
    <p class="app-sub">
      Server-backed Node/Express + PostgreSQL app. Exports a ZIP of CSV files per user; also has a
      narrow token API. Uses subsystems as separate system records. Fixed rich alter fields rather
      than free-form custom fields. The export covers more than alters and journals: <code>bdaPlan</code>,
      <code>innerWorlds</code>, <code>rules</code>, <code>wishlist</code>, communal journals, and the
      forum surface (<code>categories</code>, <code>threads</code>, <code>threadPosts</code>) ship in
      the same ZIP. Most land in <code>extensions</code> for a converter built today.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Lighthouse shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td>Subsystems as separate systems</td><td>Multiple <a href="spec-records.html#system">System</a> records with <code>parent_system_id</code></td></tr>
          <tr><td>Fixed alter fields (source/type/relationship)</td><td><a href="spec-records.html#customfielddefinition">CustomFieldDefinition</a> with <code>kind: "select"</code> or taxonomy</td></tr>
          <tr><td><code>relationships</code> column on alters (denormalized text per alter)</td><td><a href="spec-modules.html#memberrelationship">MemberRelationship</a> + <a href="spec-modules.html#relationshiptype">RelationshipType</a> via heuristic parse; emit <code>warning</code> code <code>"relationships_parsed_from_text"</code> (module is provisional)</td></tr>
          <tr><td>Forums + threads + communal journals</td><td><a href="spec-modules.html#conversation">Conversation</a> with <code>kind: "forum"</code>/<code>"thread"</code></td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/lighthouse.md">Research doc</a>
      <a href="https://github.com/team-crystalline/Lighthouse">Repository</a>
    </div>
  </article>

  <article class="app-block panel" id="octocon">
    <h3>Octocon</h3>
    <p class="app-sub">
      Elixir/Phoenix/ScyllaDB monolith plus Discord bot for DID/OSDD system management. Backend
      (<code>OctoconDev/octocon</code>) and app (<code>OctoconDev/app</code>) repos are now public.
      Alters carry name, pronouns, color, avatar, four-level security, <code>untracked</code>/
      <code>archived</code>/<code>pinned</code> flags, Discord proxy strings, and embedded custom-field
      values. Fronts are per-alter intervals with a short comment. Tags are hierarchical and play the
      role of groups. Custom-field definitions live on the user; values live inline on alters with
      <code>text</code>/<code>number</code>/<code>boolean</code> types. System and per-alter journals
      exist in the schema. Friendships are bidirectional with <code>friend</code>/<code>trusted_friend</code>
      levels that pair with alter/tag <code>security_level</code>.
    </p>
    <p class="app-sub">
      The current user-facing export is a Discord slash command (<code>/export</code>) with two formats:
      <em>PluralKit datafile v2</em> (one-way migration to PK; switch history dropped) or
      <em>Octocon "full" JSON</em> (alters, fronts, tags, polls, user profile). The full export
      currently omits journals, friendships, Discord settings, and several alter flags.
    </p>
    <div class="callout">
      <p>Octocon announced its own discontinuation in 2026, shortly after Simply Plural's — see the <a href="https://blog.octocon.app/post/810568179808157696/regarding-octocon-and-simply-plural">team's statement</a>. The realistic audience for any Octocon exporter is people getting their data out before shutdown.</p>
    </div>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Octocon shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td><code>Alters.Alter</code> with <code>untracked</code>/<code>archived</code>/<code>pinned</code></td><td><a href="spec-records.html#member">Member</a>; <code>untracked</code> mirrors SP custom-front via <code>extensions.octocon.untracked</code></td></tr>
          <tr><td><code>Fronts.Front</code> rows (per-alter, with optional <code>comment</code>)</td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> + one <a href="spec-fronting.html#frontassignment">FrontAssignment</a> per row; carry <code>comment</code> on the assignment</td></tr>
          <tr><td><code>Tags.Tag</code> with <code>parent_tag_id</code></td><td><a href="spec-records.html#group">Group</a> + <a href="spec-records.html#groupmembership">GroupMembership</a> (Octocon's PK exporter already calls these groups)</td></tr>
          <tr><td><code>Accounts.Field</code> defs + <code>Alters.Field</code> values</td><td><a href="spec-records.html#customfielddefinition">CustomFieldDefinition</a> + <a href="spec-records.html#customfieldvalue">CustomFieldValue</a>; values stringly typed</td></tr>
          <tr><td><code>discord_proxies</code> array (<code>prefixtextsuffix</code>) + <code>proxy_name</code></td><td><a href="spec-records.html#proxytag">ProxyTag</a> array; preserve <code>proxy_name</code> via display name</td></tr>
          <tr><td><code>security_level</code> on alters/tags + <code>Friendships.Friendship.level</code></td><td>Privacy fragment; raw enums via <code>extensions.octocon.security_level</code> / <code>friendship_level</code></td></tr>
          <tr><td>Journals (<code>GlobalJournalEntry</code>, <code>AlterJournalEntry</code>) — present in schema, not in export</td><td><a href="spec-records.html#note">Note</a> only when read directly from the API; the Discord export drops them</td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta mt-14">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/octocon.md">Research doc</a>
      <a href="https://github.com/OctoconDev/octocon">Backend repo</a>
      <a href="https://github.com/OctoconDev/app">App repo</a>
      <a href="https://octocon.app/docs">Public docs</a>
    </div>
  </article>

  <article class="app-block panel" id="openselves">
    <h3>OpenSelves</h3>
    <p class="app-sub">
      Smaller offline-first app with browser IndexedDB + a Drizzle/PostgreSQL sync schema. No general
      export file found in source — portable shape is inferred from sync DTOs and database schema.
      Per-member front intervals; no groups, custom fields, notes, or chat.
    </p>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/openselves.md">Research doc</a>
      <a href="https://github.com/FreckleQueens/OpenSelves">Repository</a>
    </div>
  </article>

  <article class="app-block panel" id="pluralkit">
    <h3>PluralKit</h3>
    <p class="app-sub">
      Discord bot with public API and a JSON datafile (currently <code>version: 2</code>) produced by
      <code>DataFileService.ExportSystem</code>. Uses point-in-time switch events rather than intervals.
      Flat groups. Rich privacy on members. Discord proxy records sit outside the core export.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>PluralKit shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td><code>switches[]</code> with <code>members</code> array</td><td><a href="spec-fronting.html#frontevent">FrontEvent[]</a> with assignments using <code>front_role: "member"</code></td></tr>
          <tr><td><code>members[]</code> with <code>proxy_tags</code></td><td><a href="spec-records.html#member">Member</a> with <a href="spec-records.html#proxytag">ProxyTag</a> array</td></tr>
          <tr><td>Flat <code>groups[]</code></td><td><a href="spec-records.html#group">Group</a> with null <code>parent_group_id</code></td></tr>
          <tr><td>Per-record privacy</td><td>Privacy fragment with <code>source.pluralkit</code> raw detail</td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/pluralkit.md">Research doc</a>
      <a href="https://pluralkit.me/api/">API docs</a>
    </div>
  </article>

  <article class="app-block panel" id="plural-star">
    <h3>Plural Star</h3>
    <p class="app-sub">
      React Native app with <code>AsyncStorage</code> + filesystem backups. Local backup JSON v1.2.
      Tiered fronting (primary / co-front / co-conscious), flat groups, definitions + values for custom
      fields, channels + messages, noteboards, journals. Formerly "Plural Space" (with a space) before
      its rebrand &mdash; not the same product as PluralSpace below.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Plural Star shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td>Tiered front periods</td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a> with explicit <code>front_role</code> per tier; <code>source_kind: "tiered"</code></td></tr>
          <tr><td><code>chatChannels</code> + <code>ps:chat:&lt;id&gt;</code> messages</td><td><a href="spec-modules.html#conversation">Conversation</a> + <a href="spec-modules.html#chatmessage">ChatMessage</a></td></tr>
          <tr><td>Noteboards (member-targeted posts)</td><td><a href="spec-modules.html#boardpost">BoardPost</a> with <code>target_member_id</code> + <code>pinned</code></td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/plural-star.md">Research doc</a>
      <a href="https://github.com/TheHanyou/Plural-Star">Repository</a>
    </div>
  </article>

  <article class="app-block panel" id="simply-plural">
    <h3>Simply Plural</h3>
    <p class="app-sub">
      Announced March 7, 2026 — servers stay up until at least June 1, 2026.
      Public token API + a more raw Mongo-collection export. Many migration paths flow through it,
      so the export shape still matters.
    </p>
    <div class="table-wrap">
      <table>
        <thead><tr><th>Simply Plural shape</th><th>OpenPlural target</th></tr></thead>
        <tbody>
          <tr><td>Per-member/status intervals</td><td><a href="spec-fronting.html#frontperiod">FrontPeriod</a></td></tr>
          <tr><td>Hierarchical groups</td><td><a href="spec-records.html#group">Group</a> with <code>parent_group_id</code></td></tr>
          <tr><td>Custom fronts</td><td><a href="spec-records.html#member">Member</a> with <code>is_custom_front: true</code></td></tr>
          <tr><td>Privacy buckets</td><td>Privacy fragment with <code>source.simply_plural.privacyBuckets</code></td></tr>
          <tr><td>Channels + chat messages</td><td><a href="spec-modules.html#conversation">Conversation</a> + <a href="spec-modules.html#chatmessage">ChatMessage</a></td></tr>
          <tr><td><code>boardMessages</code> (member-addressed with title/body, written by/for)</td><td><a href="spec-modules.html#boardpost">BoardPost</a></td></tr>
        </tbody>
      </table>
    </div>
    <div class="app-meta">
      <a href="https://github.com/skylartaylor/openplural/blob/main/docs/apps/simply-plural.md">Research doc</a>
      <a href="https://github.com/ApparyllisOrg/SimplyPluralApi">API source</a>
      <a href="https://apparyllis.com/simply-plural-will-be-discontinued/">Discontinuation notice</a>
    </div>
  </article>
</section>

<section id="messaging">
  <div class="section-head">
    <h2>Chat &amp; messaging across apps</h2>
    <p>
      Messaging is where apps disagree most. The optional <code>chat</code> module covers what
      they share; per-app oddities go in <code>extensions</code>.
    </p>
  </div>
  <div class="message-grid">
    <article class="message-card">
      <h3>Prism</h3>
      <p>Closest to a modern internal chat schema.</p>
      <ul>
        <li>Conversations with participants, categories, read timestamps, archive/mute state.</li>
        <li>Messages with author, content, edits, replies, reactions, system-message flag.</li>
        <li>Media attachments with hashes, MIME data, dimensions, blurhash, waveform.</li>
      </ul>
    </article>
    <article class="message-card">
      <h3>Simply Plural</h3>
      <p>Two surfaces: chat and board messages.</p>
      <ul>
        <li><code>channels</code> + <code>chatMessages</code> form Discord-like chat.</li>
        <li><code>boardMessages</code> are member-addressed posts with title, body, target.</li>
        <li>Chat text is decrypted on export.</li>
      </ul>
    </article>
    <article class="message-card">
      <h3>Plural Star</h3>
      <p>Local channels plus noteboards.</p>
      <ul>
        <li><code>chatChannels</code> + per-channel <code>ps:chat:&lt;id&gt;</code> message arrays.</li>
        <li>Messages: text, image, file, reply, or reaction.</li>
        <li>Noteboard entries are member-targeted with author and pinned state.</li>
      </ul>
    </article>
    <article class="message-card">
      <h3>Ampersand</h3>
      <p>Board-style, not channel chat.</p>
      <ul>
        <li><code>boardMessages</code> have member, title, body, date, pinned/archive flags.</li>
        <li>Board messages can include polls with choices and votes.</li>
        <li>No channel/thread chat in the inspected export.</li>
      </ul>
    </article>
    <article class="message-card">
      <h3>PluralKit / Octocon</h3>
      <p>Discord proxying, not internal chat.</p>
      <ul>
        <li>PluralKit: proxy tags, autoproxy config, proxied messages outside core export.</li>
        <li>Octocon: <code>discord_proxies</code> array on the alter (<code>prefixtextsuffix</code>) + <code>proxy_name</code>; no internal chat.</li>
        <li>Portable data should preserve proxy settings separately from message archives.</li>
      </ul>
    </article>
    <article class="message-card">
      <h3>Lighthouse / Sheaf / OpenSelves</h3>
      <p>No direct internal chat in current portable shapes.</p>
      <ul>
        <li>Lighthouse: forums, threads, thread posts, communal journals.</li>
        <li>Sheaf: no chat in <code>/v1/export</code>.</li>
        <li>OpenSelves: no chat model in inspected schema.</li>
      </ul>
    </article>
  </div>
</section>
