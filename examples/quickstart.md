# Quickstart: a full session in Saved Messages

A copy-paste smoke test against your own Saved Messages (`me`) - the always-allowed, zero-risk target. Every step shows the expected JSON shape (ids will differ). Prerequisites: [README](../README.md#install--setup) setup done, session authorized.

## 0. Sanity: offline dry-run (nothing is sent, guaranteed)

```console
$ python3 telethon_checklist.py create --title "Quickstart" --task "step one" --dry-run
{"ok": true, "kind": "checklist-create", "dry_run": true,
 "would_send": {"chat": "me", "thread": null, "title": "Quickstart",
                "tasks": ["step one"], "others_can_append": false, "others_can_complete": false},
 "source_map": [], "warnings": []}
```

## 1. Write a plan and validate it offline

```console
$ cat > /tmp/plan.json <<'EOF'
{
  "target": {"chat": "me"},
  "title": "Quickstart",
  "collected_from_chat": false,
  "tasks": [{"text": "step one"}, {"text": "step two"}]
}
EOF

$ python3 telethon_checklist.py plan --file /tmp/plan.json
{"ok": true, "kind": "plan", "dry_run": true,
 "would_send": {"chat": "me", "thread": null, "title": "Quickstart",
                "tasks": ["step one", "step two"], ...}, "warnings": []}
```

## 2. Create (the first real write)

```console
$ python3 telethon_checklist.py create --from-plan /tmp/plan.json
{"ok": true, "kind": "checklist-create", "chat": "me", "thread": null, "message_id": 12345,
 "title": "Quickstart", "tasks_created": 2,
 "verified": {"title": "Quickstart", "total": 2, "done": 0,
              "others_can_append": false, "others_can_complete": false,
              "tasks": [{"id": 1, "text": "step one", "done": false},
                        {"id": 2, "text": "step two", "done": false}]},
 "warnings": []}
```

An interactive checklist appears in your Saved Messages. Note the `message_id` - every later command needs it.

## 3. Read it back

```console
$ python3 telethon_checklist.py get --chat me --message-id 12345
{"ok": true, "kind": "checklist-get", "chat": "me", "message_id": 12345, "title": "Quickstart",
 "done": 0, "total": 2,
 "tasks": [{"id": 1, "text": "step one", "done": false},
           {"id": 2, "text": "step two", "done": false}],
 "others_can_append": false, "others_can_complete": false, "warnings": []}
```

## 4. Tick a task

```console
$ python3 telethon_checklist.py toggle --chat me --message-id 12345 --done 2
{"ok": true, "kind": "checklist-toggle", "chat": "me", "message_id": 12345,
 "marked_done": [2], "marked_undone": [],
 "verified": {"title": "Quickstart", "total": 2, "done": 1, ...,
              "tasks": [..., {"id": 2, "text": "step two", "done": true}]},
 "warnings": []}
```

The checkbox ticks in the app and the progress counter reads 1/2.

## 5. Append a task

```console
$ python3 telethon_checklist.py append --chat me --message-id 12345 --task "step three"
{"ok": true, "kind": "checklist-append", "chat": "me", "message_id": 12345,
 "appended": 1, "total": 3,
 "verified": {"title": "Quickstart", "total": 3, "done": 1, ...}, "warnings": []}
```

## 6. Final read

```console
$ python3 telethon_checklist.py get --chat me --message-id 12345
{"ok": true, ..., "done": 1, "total": 3,
 "tasks": [{"id": 1, "text": "step one", "done": false},
           {"id": 2, "text": "step two", "done": true},
           {"id": 3, "text": "step three", "done": false}], "warnings": []}
```

## 7. See clean refusals (nothing is written)

```console
$ python3 telethon_checklist.py create --title "x" --task "y" --chat -100999999999
{"ok": false, "error": "target -100999999999 not allowed - only Saved ('me') and TELETHON_CHECKLIST_CHATS entries; current allowlist: {'me': 'any'}"}

$ python3 telethon_checklist.py toggle --chat me --message-id 12345 --done 99
{"ok": false, "error": "unknown task ids [99]; valid: [1, 2, 3]"}
```

## 8. Clean up

Delete the "Quickstart" checklist from Saved Messages by hand (the script has no delete on purpose) and remove `/tmp/plan.json`.
