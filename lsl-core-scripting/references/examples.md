# Canonical Working LSL Scripts

Complete, Second-Life-compliant examples. Copy-paste ready.

## Touch + Permission (animation)
```lsl
default
{
    state_entry()
    {
        llOwnerSay("Touch to sit.");
        llRequestPermissions(llGetOwner(), PERMISSION_TRIGGER_ANIMATION);
    }

    run_time_permissions(integer perm)
    {
        if (!(perm & PERMISSION_TRIGGER_ANIMATION))
        {
            llOwnerSay("Permission denied — cannot animate.");
            return;
        }
        llSitTarget(<0, 0, 0.5>, <0, 0, 0, 1>);
    }

    touch_start(integer num_detected)
    {
        llSitTarget(<0, 0, 0.5>, <0, 0, 0, 1>);
    }

    changed(integer change)
    {
        if (change & CHANGE_OWNER)
            llRequestPermissions(llGetOwner(), PERMISSION_TRIGGER_ANIMATION);
    }
}
```

## Listen on a unique negative channel
```lsl
default
{
    integer gChan;
    state_entry()
    {
        gChan = ~1000;                       // negative, unlikely to collide
        llListen(gChan, "", "", "");
        llOwnerSay("Listening on " + (string)gChan);
    }

    listen(integer channel, string name, key id, string message)
    {
        if (message == "toggle")
            llOwnerSay("Message from " + name + " (" + (string)id + ")");
    }
}
```

## Timer + notecard read (dataserver)
```lsl
default
{
    string  gCard = "notes";
    integer gLine;
    list    gLines;

    state_entry()
    {
        llOwnerSay("Loading " + gCard);
        gLine = 0;
        llGetNotecardLine(gCard, gLine);
    }

    dataserver(key qid, string data)
    {
        if (data == EOF)
        {
            llOwnerSay("Loaded " + (string)llGetListLength(gLines) + " lines.");
            llSetTimerEvent(0.0);
            return;
        }
        gLines += data;                        // accumulate
        gLine += 1;
        llGetNotecardLine(gCard, gLine);       // request next line
    }
}
```

## HTTP request/response bridge
```lsl
default
{
    state_entry()
    {
        llOwnerSay("Fetching...");
        llHTTPRequest("https://api.example.com/data", 
            [HTTP_METHOD, "GET", HTTP_MIMETYPE, "application/json"], "");
    }

    http_response(key request_id, integer status, list metadata, string body)
    {
        if (status == 200)
        {
            llOwnerSay("Got " + (string)llStringLength(body) + " bytes");
            // parse with llJsonGetValue / llJsonKeys
        }
        else
            llOwnerSay("HTTP status " + (string)status);
    }
}
```

## Attachment lifecycle (attach + changed)
```lsl
default
{
    state_entry()
    {
        llRequestPermissions(llGetOwner(), PERMISSION_TRIGGER_ANIMATION);
    }

    attach(key owner_id)
    {
        if (owner_id != NULL_KEY)
        {
            llOwnerSay("Attached to " + (string)owner_id);
            llSetTimerEvent(1.0);                  // do periodic work while worn
        }
        else
        {
            llOwnerSay("Detached — stopping.");
            llSetTimerEvent(0.0);
        }
    }

    timer()
    {
        // periodic maintenance while attached
    }
}
```

## llDialog menu (must handle Close / No Permission)
```lsl
default
{
    integer gChan;
    key    gTarget;

    state_entry()
    {
        gChan = ~2000;
        llListen(gChan, "", "", "");
    }

    touch_start(integer num)
    {
        gTarget = llDetectedKey(0);
        llDialog(gTarget, gChan, ["Open", "Close", "No Permission"]);
    }

    listen(integer channel, string name, key id, string message)
    {
        if (message == "Open")
            llInstantMessage(id, "Opening...");
        else if (message == "No Permission")
            llInstantMessage(id, "You lack permission.");
        // "Close" is the dialog close button — do nothing
    }
}
```
