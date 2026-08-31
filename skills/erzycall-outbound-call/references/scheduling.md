# Scheduling

## Immediate vs scheduled

| Intent | `scheduledAt` | Resulting `status` |
|---|---|---|
| Call now | **omit the field entirely** | `new` → `processing` |
| Call later | ISO 8601 with offset | `scheduled` |

Passing `scheduledAt` with a near-now timestamp is not the same as omitting it, and passing
`null` is not valid. For "now", leave the field out.

## Timestamp format

The MCP validates `scheduledAt` strictly. It must carry an explicit UTC offset or `Z`:

```
2026-09-01T15:00:00+08:00     ✅
2026-09-01T07:00:00Z          ✅
2026-09-01T15:00              ❌ no offset, no seconds
2026-09-01 15:00:00+08:00     ❌ space instead of T
09/01/2026 3:00 PM            ❌
```

Seconds are optional; the offset is not.

## Resolving relative times

"Tomorrow at 3", "in an hour", "next Monday morning" all need an anchor. Get today's actual
date and time before computing one — do not assume the date from memory.

Then decide **whose** clock:

- The **contact's** local time is what matters. 3pm means 3pm where the phone rings.
- Infer the contact's timezone from their country dialing code, and say what you inferred.
- If the user and contact are in different zones, state both in the confirmation card:
  `Sep 1, 2026 at 3:00 PM MYT (8:00 AM your time)`.
- When you cannot infer it confidently, ask. A call at the wrong hour is worse than a question.

`scheduledAt` must be in the future. A past timestamp is an error, not an instant dial.

## Calling hours

The MCP will happily schedule 3am. Sanity-check the local hour at the destination and raise it
before confirming:

> That lands at 6:40 AM in Kota Kinabalu. Want to push it to 9?

Reasonable default window: 9:00–20:00 local, weekdays. Adjust to whatever the user says — many
businesses have their own rules — but always surface it when a time falls outside.

## Batches

When several calls are scheduled close together, space them out. Back-to-back scheduling of
many calls at the same minute can queue behind each other, and every call in the batch still
needs its own `idempotencyKey`.

The confirmation rule does not weaken for a batch: the user sees the full list — every
destination, every time — and approves before anything dispatches.

## Reschedule

There is no reschedule tool. To move a scheduled call:

1. `cancel_call(callId, confirmed: true, idempotencyKey)` — after confirming which call.
2. `create_call(...)` with the new `scheduledAt` and a **new** `idempotencyKey`.

Confirm both steps. Do not cancel until the user has agreed to the replacement, so a failure
in step 2 does not silently drop the call altogether.
