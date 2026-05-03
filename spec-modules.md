---
title: "OpenPlural Spec — Modules & contract"
nav_active: spec
---

<section>
  <h1>Optional modules &amp; importer contract</h1>
  <p class="sub"><span class="draft">draft v0.1</span>Chat and boards get full field specs because losing message history hurts most. The relationships module has a provisional shape — input wanted before it firms up. The other optional modules (polls, reminders, habits, proxy, sharing, safety) are sketched at the card level for v0.2.</p>

  <details class="toc">
    <summary>on this page</summary>
    <ol>
      <li><a href="#chat">Optional chat module</a>
        <ol>
          <li><a href="#conversation">Conversation</a></li>
          <li><a href="#chatmessage">ChatMessage</a></li>
          <li><a href="#attachment">Attachment</a></li>
          <li><a href="#reaction">Reaction</a></li>
        </ol>
      </li>
      <li><a href="#boards">Optional boards module</a>
        <ol>
          <li><a href="#boardpost">BoardPost</a></li>
        </ol>
      </li>
      <li><a href="#relationships">Optional relationships module <span class="draft">provisional</span></a>
        <ol>
          <li><a href="#relationshiptype">RelationshipType</a></li>
          <li><a href="#memberrelationship">MemberRelationship</a></li>
        </ol>
      </li>
      <li><a href="#other-modules">Other modules</a></li>
      <li><a href="#contract">Importer contract</a></li>
      <li><a href="#importresult">ImportResult</a></li>
      <li><a href="#validator">Validator plans</a></li>
    </ol>
  </details>
</section>

<div class="spec-page mt-24">
  <div>
    <section id="chat">
      <div class="section-head">
        <h2>Optional chat module</h2>
        <p>One shape that fits internal chat, forums, threaded discussion, and proxy logs. Member-addressed posts (boards, noteboards) live in the separate <a href="#boards">boards module</a> because the unit is a post with a target member, not a message in a thread. Apps without chat keep the whole object as archive data and report it in <code>warnings</code>.</p>
      </div>

      <article class="record" id="conversation">
        <h3>Conversation</h3>
        <p class="record-blurb">A container for messages. <code>kind</code> distinguishes channel chat from forum-style and thread-style surfaces.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>kind</td><td>string</td><td class="req-yes">yes</td><td>See enum below.</td></tr>
              <tr><td>title</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>category</td><td>string | null</td><td class="req-no">no</td><td>Optional grouping. Prism: <code>conversations.category</code>.</td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>emoji</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>participant_member_ids</td><td>UUID[]</td><td class="req-no">no</td><td>Empty for system-wide channels.</td></tr>
              <tr><td>creator_member_id</td><td>UUID | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>archived</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>.</td></tr>
              <tr><td>muted</td><td>boolean</td><td class="req-no">no</td><td></td></tr>
              <tr><td>direct_message</td><td>boolean</td><td class="req-no">no</td><td>Prism: <code>conversations.isDirectMessage</code>.</td></tr>
              <tr><td>created_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <div class="enum-sidebar">
          <b>Recommended <code>kind</code> values</b>
          <code>"internal_chat" | "forum" | "thread" | "direct_message" | "proxy_log" | "unknown"</code>
        </div>
      </article>

      <article class="record" id="chatmessage">
        <h3>ChatMessage</h3>
        <p class="record-blurb">A single message. Supports channel posts, system messages, replies, and edits.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>conversation_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>chat.conversations[].id</code>.</td></tr>
              <tr><td>author_member_id</td><td>UUID | null</td><td class="req-no">no</td><td>Null for system messages.</td></tr>
              <tr><td>kind</td><td>"text" | "image" | "file" | "system" | "reply" | "reaction" | "unknown"</td><td class="req-no">no</td><td>Defaults to <code>"text"</code>.</td></tr>
              <tr><td>body</td><td>string</td><td class="req-yes">yes</td><td>Markdown if the source app uses it.</td></tr>
              <tr><td>created_at</td><td>ISO8601</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>edited_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>reply_to_message_id</td><td>UUID | null</td><td class="req-no">no</td><td>For threaded replies.</td></tr>
              <tr><td>attachment_asset_ids</td><td>UUID[]</td><td class="req-no">no</td><td>References <code>assets[]</code>. See <a href="#attachment">Attachment</a> for richer per-attachment metadata.</td></tr>
              <tr><td>system_message</td><td>boolean</td><td class="req-no">no</td><td></td></tr>
              <tr><td>pinned</td><td>boolean</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="attachment">
        <h3>Attachment</h3>
        <p class="record-blurb">Optional richer metadata for a message attachment. Lightweight references can use <code>chat.messages[].attachment_asset_ids</code> directly.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>message_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>asset_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>assets[]</code>.</td></tr>
              <tr><td>role</td><td>"primary" | "thumbnail" | "preview" | "inline" | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>caption</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="reaction">
        <h3>Reaction</h3>
        <p class="record-blurb">A reaction left by a member on a message.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>message_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>member_id</td><td>UUID | null</td><td class="req-no">no</td><td>Null for anonymous reactions.</td></tr>
              <tr><td>value</td><td>string</td><td class="req-yes">yes</td><td>Unicode emoji, <code>:shortcode:</code>, or app-specific reaction ID.</td></tr>
              <tr><td>created_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <pre class="mt-14"><code>{
  "chat": {
    "conversations": [
{
  "id": "conv_01HV4Z...",
  "system_id": "sys_01HV4Z...",
  "kind": "internal_chat",
  "title": "General",
  "participant_member_ids": [],
  "archived": false,
  "source_refs": [{ "app": "prism", "collection": "conversations", "id": "c_..." }],
  "extensions": {}
}
    ],
    "messages": [
{
  "id": "msg_01HV4Z...",
  "conversation_id": "conv_01HV4Z...",
  "author_member_id": "mem_01HV4Z...",
  "kind": "text",
  "body": "Hi.",
  "created_at": "2026-04-29T18:00:00Z",
  "edited_at": null,
  "reply_to_message_id": null,
  "attachment_asset_ids": [],
  "source_refs": [],
  "extensions": {}
}
    ],
    "attachments": [],
    "reactions": []
  }
}</code></pre>

      <div class="callout mt-14">
        <p>An app without chat should still preserve the entire <code>chat</code> object as import archive data and emit a <code>warning</code> with code <code>"module_archived_only"</code>. Users need to know their messages weren't dropped.</p>
      </div>
    </section>

    <section id="boards">
      <div class="section-head">
        <h2>Optional boards module</h2>
        <p>Member-addressed posts: profile walls, noteboards, board messages. The unit is a post with a target member (or system-wide timeline), not a message in a thread. No replies, threading, or per-message reactions in the v0.1 shape — apps that have those should record them in <code>extensions</code> until v0.2.</p>
      </div>

      <article class="record" id="boardpost">
        <h3>BoardPost</h3>
        <p class="record-blurb">A post addressed to a member's board, or to the system-wide timeline when <code>target_member_id</code> is null.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>target_member_id</td><td>UUID | null</td><td class="req-no">no</td><td>Null = system-wide timeline (Prism public board with no target).</td></tr>
              <tr><td>author_member_id</td><td>UUID | null</td><td class="req-no">no</td><td>Null = legacy/imported post with no surviving author (Prism preserves this for SP imports).</td></tr>
              <tr><td>title</td><td>string | null</td><td class="req-no">no</td><td>Prism, Simply Plural. Plural Star and Ampersand have no title.</td></tr>
              <tr><td>body</td><td>string</td><td class="req-yes">yes</td><td>Markdown if the source app uses it. Required and non-empty.</td></tr>
              <tr><td>audience</td><td>"public" | "private" | "unknown"</td><td class="req-no">no</td><td>Defaults to <code>"public"</code>. Prism's public/private distinction. Apps without an audience concept use <code>"public"</code>.</td></tr>
              <tr><td>pinned</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>. Plural Star, Simply Plural.</td></tr>
              <tr><td>created_at</td><td>ISO8601</td><td class="req-yes">yes</td><td>When the post entered the source system.</td></tr>
              <tr><td>written_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>User-facing authored time. Differs from <code>created_at</code> on imports — Prism preserves SP's <code>writtenAt</code> here.</td></tr>
              <tr><td>edited_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>deleted_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Soft-delete tombstone. Prism uses <code>isDeleted</code>; importers should set <code>deleted_at</code> to a timestamp when known, or to a sentinel time when not.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <pre class="mt-14"><code>{
  "boards": {
    "posts": [
{
  "id": "bp_01HV4Z...",
  "system_id": "sys_01HV4Z...",
  "target_member_id": "mem_01HV4Z...",
  "author_member_id": "mem_01HV4Z...",
  "title": null,
  "body": "Welcome back. Hope today is gentle.",
  "audience": "public",
  "pinned": false,
  "created_at": "2026-04-29T18:00:00Z",
  "written_at": "2026-04-29T18:00:00Z",
  "edited_at": null,
  "deleted_at": null,
  "source_refs": [{ "app": "prism", "collection": "member_board_posts", "id": "..." }],
  "extensions": {}
}
    ]
  }
}</code></pre>

      <div class="callout mt-14">
        <p>Boards is a separate module from chat so that apps that have one but not the other don't pull in unused infrastructure. An app without boards should preserve the entire <code>boards</code> object as archive data and emit <code>warning</code> code <code>"module_archived_only"</code>.</p>
      </div>
    </section>

    <section id="relationships">
      <div class="section-head">
        <h2>Optional relationships module <span class="draft">provisional</span></h2>
        <p>Member-to-member edges for apps with relationship maps — partner, parent_of, metamour, etc. Two records: a type vocabulary, and the edges themselves.</p>
      </div>

      <div class="callout">
        <p><b>Heads up:</b> this module is more provisional than the rest of v0.1. Only one inspected app (Lighthouse) currently exports any relationship data, and it's an opaque text field. PluralSpace's <a href="https://pluralspace.app/developers">developer page</a> previews relationship endpoints, but the API itself isn't released yet. The shape below is forward-looking and likely to shift once more apps implement it. Treat it as a sketch we'd like input on, not a settled design.</p>
      </div>

      <article class="record" id="relationshiptype">
        <h3>RelationshipType</h3>
        <p class="record-blurb">A reusable type for member-to-member edges. Symmetric ("partner") or asymmetric ("parent_of" with inverse "child_of").</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>name</td><td>string</td><td class="req-yes">yes</td><td>Canonical name, e.g. <code>"partner"</code>, <code>"parent_of"</code>, <code>"metamour"</code>. Free-form — no enum.</td></tr>
              <tr><td>inverse_name</td><td>string | null</td><td class="req-no">no</td><td>For asymmetric types: the opposite-direction name. <code>"parent_of"</code> → <code>"child_of"</code>. Null for symmetric types.</td></tr>
              <tr><td>symmetric</td><td>boolean</td><td class="req-no">no</td><td>Defaults to <code>false</code>. <code>true</code> means one stored edge represents the relationship in both directions.</td></tr>
              <tr><td>description</td><td>string | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>color</td><td>HexColor | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>sort_order</td><td>number | null</td><td class="req-no">no</td><td></td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <article class="record" id="memberrelationship">
        <h3>MemberRelationship</h3>
        <p class="record-blurb">An edge between two members of a given <a href="#relationshiptype">RelationshipType</a>.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>system_id</td><td>UUID</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>from_member_id</td><td>UUID</td><td class="req-yes">yes</td><td>For asymmetric types, the subject (e.g. the parent in "parent_of"). For symmetric types, just storage order.</td></tr>
              <tr><td>to_member_id</td><td>UUID</td><td class="req-yes">yes</td><td>For asymmetric types, the object.</td></tr>
              <tr><td>type_id</td><td>UUID</td><td class="req-yes">yes</td><td>References <code>relationships.types[].id</code>.</td></tr>
              <tr><td>note</td><td>string | null</td><td class="req-no">no</td><td>Free-text annotation on this specific edge.</td></tr>
              <tr><td>started_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Real-world start of the relationship, distinct from when the record was written.</td></tr>
              <tr><td>ended_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Null = ongoing.</td></tr>
              <tr><td>created_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>When the record was written.</td></tr>
              <tr><td>deleted_at</td><td>ISO8601 | null</td><td class="req-no">no</td><td>Soft-delete tombstone — preserves CRDT-sync semantics for apps that delete-by-flag.</td></tr>
              <tr><td>source_refs</td><td>SourceRef[]</td><td class="req-no">no</td><td></td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
      </article>

      <pre class="mt-14"><code>{
  "relationships": {
    "types": [
      {
        "id": "rtype_01HV...",
        "system_id": "sys_01HV...",
        "name": "parent_of",
        "inverse_name": "child_of",
        "symmetric": false,
        "description": null,
        "color": null,
        "sort_order": 0,
        "source_refs": [],
        "extensions": {}
      }
    ],
    "edges": [
      {
        "id": "rel_01HV...",
        "system_id": "sys_01HV...",
        "from_member_id": "mem_01HV...",
        "to_member_id": "mem_01HV...",
        "type_id": "rtype_01HV...",
        "note": null,
        "started_at": null,
        "ended_at": null,
        "created_at": "2026-04-29T18:00:00Z",
        "deleted_at": null,
        "source_refs": [],
        "extensions": {}
      }
    ]
  }
}</code></pre>

      <div class="callout mt-14">
        <p>Apps without relationships should preserve the whole <code>relationships</code> object as archive data and emit a <code>warning</code> with code <code>"module_archived_only"</code>. Lighthouse's denormalized text field can be parsed heuristically into multiple edges — that's lossy by definition; flag the parse with <code>warning</code> code <code>"relationships_parsed_from_text"</code>.</p>
      </div>
    </section>

    <section id="other-modules">
      <div class="section-head">
        <h2>Other optional modules</h2>
        <p>Sketched at module level.</p>
      </div>
      <div class="module-grid">
        <article class="module-card">
          <h3>polls</h3>
          <p>Polls, options, votes, vote reasons, close state, expiry. Maps Prism polls and Ampersand board polls.</p>
        </article>
        <article class="module-card">
          <h3>reminders</h3>
          <p>One-off and recurring reminders, intervals, assigned members, notification preferences.</p>
        </article>
        <article class="module-card">
          <h3>habits</h3>
          <p>Habit definitions, recurrence rules, completions, streaks, assigned members.</p>
        </article>
        <article class="module-card">
          <h3>proxy</h3>
          <p>Discord proxy tags, autoproxy settings, account links. Distinct from core <a href="spec-records.html#proxytag">ProxyTag</a> on members.</p>
        </article>
        <article class="module-card">
          <h3>sharing</h3>
          <p>Friends, peer keys, granted scopes, verification state, per-record visibility.</p>
        </article>
        <article class="module-card">
          <h3>safety</h3>
          <p>Safety plans, retention overrides, lock states, destructive-action safeguards, app security flags.</p>
        </article>
      </div>
    </section>

    <section id="contract">
      <div class="section-head">
        <h2>Importer contract</h2>
        <p>The file shape is half the story. The other half is what an importer reports back: what moved, what changed, what was kept as archive only.</p>
      </div>

      <div class="panel">
        <h3>Expected importer behavior</h3>
        <ul class="contract-list">
          <li>Validate the envelope and resolve every cross-record reference before mutating app data.</li>
          <li>Emit per-module counts: imported, skipped, degraded, preserved-only, failed.</li>
          <li>Keep raw unsupported module data when an archive area exists.</li>
          <li>Preserve <code>source_refs</code> on imported records when the target app has a place for them.</li>
          <li>Use conservative privacy defaults when the target app can't express the source privacy model.</li>
          <li>Reject the file with an error if <code>openplural_version</code> is unknown — don't silently ignore unsupported versions.</li>
        </ul>
      </div>

      <article class="record mt-24" id="importresult">
        <h3>ImportResult</h3>
        <p class="record-blurb">The shape importers should return after a run. Useful as both a UI summary and a logged audit record.</p>
        <div class="table-wrap">
          <table class="field-table">
            <thead><tr><th>Field</th><th>Type</th><th>Required</th><th>Notes</th></tr></thead>
            <tbody>
              <tr><td>started_at</td><td>ISO8601</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>finished_at</td><td>ISO8601</td><td class="req-yes">yes</td><td></td></tr>
              <tr><td>source</td><td>{ openplural_version: string, producer_app: string, producer_app_version: string | null }</td><td class="req-yes">yes</td><td>Echoed from the imported envelope's <code>producer</code>.</td></tr>
              <tr><td>counts</td><td>{ imported: number, skipped: number, degraded: number, preserved_only: number, failed: number }</td><td class="req-yes">yes</td><td>Aggregate across all modules.</td></tr>
              <tr><td>per_module</td><td>Record&lt;string, ModuleCounts&gt;</td><td class="req-no">no</td><td>Optional per-module breakdown using the same shape as <code>counts</code>.</td></tr>
              <tr><td>warnings</td><td>Warning[]</td><td class="req-no">no</td><td>See <a href="spec-records.html#warning">Warning</a>.</td></tr>
              <tr><td>extensions</td><td>Record&lt;string, unknown&gt;</td><td class="req-no">no</td><td></td></tr>
            </tbody>
          </table>
        </div>
        <pre class="mt-14"><code>{
  "import_result": {
    "started_at": "2026-04-29T18:05:00Z",
    "finished_at": "2026-04-29T18:05:14Z",
    "source": {
"openplural_version": "0.1",
"producer_app": "Sheaf",
"producer_app_version": "1.4.2"
    },
    "counts": {
"imported": 284,
"skipped": 3,
"degraded": 12,
"preserved_only": 158,
"failed": 0
    },
    "per_module": {
"members": { "imported": 24, "skipped": 0, "degraded": 0, "preserved_only": 0, "failed": 0 },
"chat":    { "imported":   0, "skipped": 0, "degraded": 0, "preserved_only": 158, "failed": 0 }
    },
    "warnings": [
{
  "level": "warning",
  "code": "module_archived_only",
  "record_type": "chat.messages",
  "message": "Target app does not display chat. Messages retained as archive data.",
  "count": 158
}
    ]
  }
}</code></pre>
      </article>
    </section>

    <section id="validator">
      <div class="section-head">
        <h2>Validator plans</h2>
        <p>What we're planning to build once v0.1 fields are settled.</p>
      </div>
      <ul class="proposal-list">
        <li><b>JSON Schema</b> for <code>openplural_version: "0.1"</code> covering every record on this site.</li>
        <li><b>Reference fixtures</b>: a hand-written export per researched app under <code>fixtures/&lt;app&gt;/</code>.</li>
        <li><b>Validator CLI</b>: validates a file against the schema and prints an <a href="#importresult">ImportResult</a>-shaped report.</li>
        <li><b>Reference converters</b>: Prism, Sheaf, Simply Plural, PluralKit, Plural Star — each as a small standalone script that reads a real export and emits an OpenPlural file.</li>
        <li><b>Conformance test</b>: round-trip exporters → importers → exporters and diff the result. Lossy fields must be flagged in <code>warnings</code>.</li>
      </ul>
    </section>

    <div class="callout mt-24">
      <p>Back to <a href="spec-records.html">core records</a> or <a href="spec-fronting.html">fronting</a>. Maintainers: see the <a href="adopt.html">adoption guide</a>.</p>
    </div>
  </div>

</div>
