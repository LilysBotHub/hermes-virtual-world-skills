# HTTP / JSON Web Bridges

Connecting in-world scripts to external web services via `llHTTPRequest` / `http_response` and parsing with the JSON functions.

## Core Flow

1. Fire `llHTTPRequest(url, method_list, body)` from an event (touch, timer, listen).
2. The matching `http_response(request_id, status, metadata, body)` fires when the request completes.
3. **Always match `request_id`** to the outstanding request (a slow request can return after a new one was fired).
4. Parse `body` with `llJsonGetValue` / `llJsonKeys`.

## Request

```lsl
llHTTPRequest("https://api.example.com/data",
    [HTTP_METHOD, "GET", HTTP_MIMETYPE, "application/json"],
    "");
```

POST with a body:

```lsl
llHTTPRequest("https://api.example.com/submit",
    [HTTP_METHOD, "POST", HTTP_MIMETYPE, "application/x-www-form-urlencoded"],
    "name=Alice&score=42");
```

## Response + JSON parse

```lsl
key gPending;

default
{
    state_entry()
    {
        gPending = "";
        llHTTPRequest("https://api.example.com/data",
            [HTTP_METHOD, "GET", HTTP_MIMETYPE, "application/json"], "");
        gPending = "data";
    }

    http_response(key request_id, integer status, list metadata, string body)
    {
        if (request_id != gPending)
            return;                              // stale response, ignore
        gPending = "";

        if (status != 200)
        {
            llOwnerSay("HTTP " + (string)status);
            return;
        }

        // body looks like: {"data":{"name":"Alice","score":42}}
        string name = llJsonGetValue(body, ["data", "name"]);
        integer score = llJsonGetValue(body, ["data", "score"]);
        llOwnerSay(name + " -> " + (string)score);

        // enumerate keys of an object:
        list keys = llJsonKeys(body, ["data"]);
    }
}
```

## JSON function reference

| Function | Purpose |
|---|---|
| `llJsonGetValue(json, [path])` | Read a value by list-path (e.g. `["data","name"]` or `["items",0,"id"]`) |
| `llJsonSetValue(json, [path], value)` | Set a value by list-path (creates objects/arrays as needed) |
| `llJsonRemove(json, [path])` | Remove a path |
| `llJsonKeys(json, [path])` | Keys of an object at `path` |

## Cross-Platform Caveats (Second Life vs OpenSim)

- **Availability:** `llJsonGetValue` / `llJsonSetValue` / `llJsonKeys` exist in modern Second Life (2023+) and in OpenSim, but **version matters** — verify the target supports them.
- **Path syntax:** both use a list path, but escaping and nested-index behavior can differ between XEngine and YEngine — **round-trip test** on the target grid.
- **`llHTTPRequest` parity:** request headers, MIME handling, and response timing differ subtly across engines — test the full cycle, don't assume identical behavior.
- **CORS / server side:** the remote endpoint must allow the request (CORS headers for browser-side calls; proper response format for in-world calls).
- **Security:** never place secrets in a client-side `llHTTPRequest` URL. Put the key on the server and call a server-side endpoint.

## Matching Stale Responses

Because responses are asynchronous, keep a set of outstanding request IDs and ignore late arrivals:

```lsl
list gPending;

fire(string what)
{
    key id = llHTTPRequest("https://api.example.com/" + what, 
        [HTTP_METHOD, "GET"], "");
    gPending += what + "/" + (string)id;
}

http_response(key request_id, integer status, list metadata, string body)
{
    string tag = (string)request_id;
    integer i;
    integer n = llGetListLength(gPending);
    boolean found = FALSE;
    for (i = 0; i < n; ++i)
    {
        if (llList2String(gPending, i) == tag)
        {
            found = TRUE;
            // handle response for `what`
            break;
        }
    }
    if (!found)
        return;                                  // stale — ignore
    gPending = llListDeleteElement(gPending, i);
}
```