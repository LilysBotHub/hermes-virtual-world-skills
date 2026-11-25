# LSL Optimization Patterns

Goal: minimize per-second **script time** (LPS — script execution units) so the simulator stays responsive.

## 1. Batch prim changes with llSetLinkPrimitiveParamsFast

**Bad** — N calls in a loop:
```lsl
integer i;
for (i = 1; i <= llGetNumberOfPrims(); ++i)
    llSetLinkPrimitiveParamsFast(i, [PRIM_COLOR, ALL_SIDES, <1,1,1>, 1]);
```
**Good** — one call for the whole set:
```lsl
llSetLinkPrimitiveParamsFast(LINK_SET, [PRIM_COLOR, ALL_SIDES, <1,1,1>, 1]);
```

## 2. Strided lists (one list = many records)
Keep parallel fields interleaved so a batch update is a single call.
```lsl
// Stride 4: name, x, y, z
list avatars = ["Alice", 1.0, 2.0, 3.0, "Bob", 4.0, 5.0, 6.0];
// Every 4th element starting at 0:
integer n = llGetListLength(avatars);
integer i;
for (i = 0; i < n; i += 4)
    llOwnerSay(llList2String(avatars, i));
```
Use `llList2ListStrided(list, start, end, stride)` to slice.

## 3. Manage listeners tightly
```lsl
integer gChan;
state_entry()
{
    gChan = ~llGenerateKey();          // unique negative channel
    llListen(gChan, "", "", "");        // empty filters = accept all on this channel
}
// when idle:
llListenControl(gChan, FALSE);
// when done:
llListenRemove(gChan);
```
Never leave an open `llListen` on channel `0`. Use **negative** channels so they never collide with public chat.

## 4. Timers: on only when needed
```lsl
// Poll only during active work:
llSetTimerEvent(0.25);   // start
...
llSetTimerEvent(0.0);    // stop when idle
```
Lower frequency = less LPS. `0.1` is usually plenty; `1.0` for heartbeat.

## 5. Never llSleep in an event handler
```lsl
// Bad — freezes the script for the whole duration:
llSleep(2.0);

// Good — yield via timer:
llSetTimerEvent(2.0);
timer() { /* do work now */ }
```

## 6. Cache immutable values
```lsl
key gOwner;
state_entry()
{
    gOwner = llGetOwner();   // owner rarely changes; cache it
}
```

## 7. Prefer region/private chat over channel 0
`llSay(0, ...)` broadcasts to every viewer on the region. Use `llOwnerSay` or `llRegionSayTo(target, msg)`.

## 8. Coalesce HTTP requests
Fire one `llHTTPRequest` and cache the `http_response`; don't fire one per user action.

## 9. Avoid llGetLinkName lookups
Use integer link numbers (`llGetLinkNumber`, `llDetectedLinkNumber`) for compares and param calls.

## 10. Watch the 64 KB Mono limit
Keep lists small; push large datasets to notecards or remote storage. A script that grows unbounded will hit `SCRIPT_MEMORY_EXCEEDED` and halt.
