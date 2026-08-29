# Open decisions

Things that are not yet settled. Do not silently assume an answer to any of these while building — flag it instead.

## Aging thresholds

What are the actual target processing times per document type, and at what percentage of the target should a "warning" notification fire vs an "overdue" one? The earlier prototype used a flat 4-hour fallback and a 75% warning ratio — these were placeholder numbers, not confirmed with anyone.

## Business hours vs raw elapsed time

Should aging/turnaround calculations count only working hours (e.g. 8am-5pm, weekdays, excluding holidays), or raw elapsed clock time? Raw elapsed time is simpler to build but will make every document look far more "overdue" than it really is over a weekend. This affects every timing calculation in the system, so it needs an answer before the notification feature is built, not after.

## Multiple signatories in sequence

When a document needs signatures from more than one BAC member in order, does Secretariat manually hand it to each one in turn (current plan), or does the system need to know the required order in advance? Manual hand-off is simpler and matches the current plan, but worth confirming this matches how BAC actually operates.

## Escalation recipients

When a document is overdue, who exactly gets notified beyond the current assignee? All of Management, or a specific person?

## Document edits after registration

Can a document's details (title, type) be edited after it's registered but before it's released? Not currently planned, but worth confirming this is acceptable.
