# HUD & Dialog Menu System

Building interactive HUDs, dialog menus (`llDialog`), and multi-page option lists for Second Life and OpenSim.

## HUD Architecture

- A HUD is an **attachment**; pin it to a fixed `ATTACHPOINT` and keep the script lean (low LPS) — the HUD is worn and its script runs continuously while attached.
- **One unique negative channel per HUD.** Never use channel 0.
- Open the `llListen` in `state_entry`; close it in `state_exit` / on detach.
- Keep UI state in globals; use `http_response` / `dataserver` queues for async work instead of blocking.

## Dialog Fundamentals

`llDialog(agent, channel, list buttons)` requires an open `llListen` on the **same channel**. The avatar's choice arrives via the `listen` event as `message`. Two buttons are special and always present in some viewers:

- `"Close"` — the dialog's close button (do nothing on receive).
- `"No Permission"` — shown when the target lacks permission (handle gracefully).

## Channel + Listen Setup

```lsl
integer gChan;
default
{
    state_entry()
    {
        gChan = ~3000;                 // unique negative channel
        llListen(gChan, "", "", "");
    }

    state_exit()
    {
        llListenRemove(gChan);         // avoid listener leaks
    }
}
```

## Basic Dialog

```lsl
integer gChan;
default
{
    state_entry()
    {
        gChan = ~3000;
        llListen(gChan, "", "", "");
    }

    touch_start(integer num)
    {
        key who = llDetectedKey(0);
        llDialog(who, gChan, ["Menu", "Info", "No Permission"]);
    }

    listen(integer channel, string name, key id, string message)
    {
        if (message == "Menu")
            llDialog(id, gChan, ["Back", "Option A", "Option B"]);
        else if (message == "Info")
            llInstantMessage(id, "Info here.");
        else if (message == "No Permission")
            llInstantMessage(id, "You lack permission.");
        // "Back" / "Close" are handled by whatever logic you define; "Close" does nothing
    }
}
```

## Multi-Page Menus (paging)

`llDialog` is capped at **10 buttons**. For longer lists, page through them and store the current page in a global keyed by the caller's key.

```lsl
integer gChan;
list    gPages;     // list of pages, each a list of buttons
integer gPageOf;    // caller key -> current page (strided with gTargetOf)
key     gTargetOf;

list BuildPage(integer page, integer perPage)
{
    // slice a master list `gItems` into `perPage` buttons + a Next/Back row
    list row = [];
    integer start = page * perPage;
    integer i;
    for (i = start; i < start + perPage && i < llGetListLength(gItems); ++i)
        row += llList2String(gItems, i);
    if (page > 0)            row += "< Back";
    if (start + perPage < llGetListLength(gItems))
        row += "Next >";
    return row;
}

default
{
    state_entry()
    {
        gChan = ~3000;
        llListen(gChan, "", "", "");
        gItems = ["Item 1", "Item 2", "Item 3", "Item 4", "Item 5",
                  "Item 6", "Item 7", "Item 8", "Item 9", "Item 10",
                  "Item 11", "Item 12"];
    }

    touch_start(integer num)
    {
        key who = llDetectedKey(0);
        gTargetOf += who;
        gPageOf += 0;                    // reset this caller's page
        llDialog(who, gChan, BuildPage(0, 8));
    }

    listen(integer channel, string name, key id, string message)
    {
        integer idx = llListFindList(gTargetOf, [id]);
        if (idx == -1) return;
        integer page = llList2Integer(gPageOf, idx);

        if (message == "Next >")
            page += 1;
        else if (message == "< Back")
            page -= 1;
        else
        {
            llInstantMessage(id, "Selected: " + message);
            return;
        }

        gPageOf = llListReplaceList(gPageOf, [(string)page], idx, idx);
        llDialog(id, gChan, BuildPage(page, 8));
    }
}
```

## Checklist

- [ ] Unique negative channel, opened in `state_entry`, removed on detach/`state_exit`.
- [ ] `Close` and `No Permission` handled.
- [ ] Lists capped at 10 buttons with paging.
- [ ] Caller key tracked so multiple users don't clobber each other's page state.
- [ ] Script kept lean (HUD runs continuously while worn).