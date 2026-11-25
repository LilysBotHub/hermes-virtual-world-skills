# LSL Function Cheatsheet (Second Life)

Condensed reference of the most-used pure-LSL functions. Full signatures: https://wiki.secondlife.com/wiki/Category:LSL_Functions

## List / String / Math
| Function | Purpose |
|---|---|
| `llList2String(list, i)` / `llList2Integer` / `llList2Float` | Element access by index |
| `llList2Key(list, i)` | Cast element to `key` |
| `llListInsertList`, `llListReplaceList`, `llListDeleteElement` | Mutate lists |
| `llList2ListStrided(list, start, end, stride)` | Grab every Nth element (strided layout) |
| `llDumpList2String(list, sep)` / `llParseString2List(s, delims, splitters)` | List<->String |
| `llInsertString`, `llDeleteSubString`, `llGetSubString` | String ops |
| `llToLower`, `llToUpper`, `llStringTrim` | Case / trim |
| `llParseString2List`, `llCSV2List` | Split CSV/newlines |
| `llFloor`, `llCeil`, `llRound`, `llAbs`, `llFrand` | Integer/float math |

## Primitive / Link
| Function | Purpose |
|---|---|
| `llSetLinkPrimitiveParamsFast(linknum, params)` | Fast batched prim param change (LINK_SET, linknum, or LINK_ALL_CHILDREN) |
| `llGetLinkPrimitiveParams(linknum, [flags])` | Read params (slower, synchronous) |
| `llGetNumberOfPrims()` | Prim count of the **whole linkset** (includes seated avatars) |
| `llGetObjectPrimCount(key)` | Prim count of another object |
| `llGetLinkNumber()` | This prim's link number |
| `llGetLinkName(num)` / `llGetLinkKey(num)` | Name / key of a linked prim |
| `llCreateLink(key target, boolean attach_to_avatar)` | Link a prim (needs PERMISSION_CHANGE_LINKS) |
| `llBreakLink`, `llBreakAllLinks` | Unlink |
| `llMessageLinked(sender, num, str, id)` | Inter-prim message |

### Key link constants
`LINK_ROOT`, `LINK_THIS`, `LINK_ALL_PRIMS`, `LINK_ALL_CHILDREN`, `LINK_SET` (whole set), `LINK_NONE`.

## Color / Texture / Material
| Function | Purpose |
|---|---|
| `llSetColor(vector color, int face)` / `llSetTexture(key, face)` | Set color/texture on a face |
| `llSetMaterial(string)` | PhysX material (blinn/wood/metal/etc.) |
| `llSetLinkColor`, `llSetLinkTexture` | Per-link variants |

## Sensor / Detection
| Function | Purpose |
|---|---|
| `llSensor`, `llSensorRepeat`, `llSensorRemove` | Detect objects/avatars/landmarks |
| `llDetectGravity`, `llGetRadius` | Physical queries |

## Sound / Animation / Control
| Function | Purpose |
|---|---|
| `llPlaySound`, `llLoopSound`, `llTriggerSound` | Play sounds |
| `llStartAnimation`, `llStopAnimation` | Need PERMISSION_TRIGGER_ANIMATION |
| `llStartControl`, `llStopControl` | Need PERMISSION_CONTROL |
| `llSitTarget(vector offset, rotation localrot)` | Set sit position |
| `llAvatarOnSitTarget()` | Key of sitter (NULL_KEY if none) |

## Dataserver / URL
| Function | Purpose |
|---|---|
| `llGetNotecardLine(notecard, line)` | Async notecard read -> `dataserver` |
| `llLoadURL(owner, url, message)` | Open a web URL -> `dataserver` |

## Communication
| Function | Purpose |
|---|---|
| `llListen(int chan, string name, key id, string msg)` | Open a channel listener (store handle) |
| `llListenControl(handle, bool active)` | Pause/resume a listener |
| `llListenRemove(handle)` | Close a listener |
| `llSay`, `ll Whisper`, `llRegionSayTo`, `llRegionSay`, `llOwnerSay`, `llInstantMessage` | Chat |
| `llHTTPRequest(url, method, body)` / `http_response` | Web requests |
| `llHTTPRequest` methods: `[HTTP_METHOD, "GET"|"POST"]`, `HTTP_MIMETYPE` |

## JSON (LL 2023+)
| Function | Purpose |
|---|---|
| `llJsonGetValue(json, [path])` | Read a JSON value by path |
| `llJsonSetValue(json, [path], value)` | Set a JSON value by path |
| `llJsonRemove(json, [path])` | Remove a path |
| `llJsonKeys(json)` | Keys of an object |

## Misc / Utility
| Function | Purpose |
|---|---|
| `llOwnerSay` / `llInstantMessage` | Private feedback |
| `llSleep(float)` | **Block the script** — prefer a `timer` event |
| `llSetTimerEvent(float)` / `llSetTimerEvent(0)` | Enable/disable timer |
| `llRequestPermissions(key, perm)` / `run_time_permissions` | Permission request |
| `llGetOwner`, `llGetKey`, `llGetPos`, `llGetRotation` | Identity / transform |
| `llRegionPos`, `llGetRegionName` | Region info |
| `llCollisionFilter(skinned, geometry, distance)` | Collision filter |
| `llLinkParticleSystem(linknum, params)` | Particle systems (needs perms on some grids) |
