+++
title = "Open Brain Surgery: A Guide to API Shimming in Production"
date = 2026-02-27
description ="How a corrupted artifact server, a broken backup, an incompatible API, and a ticking clock forced an emergency production migration — and why a 400-line FastAPI shim was the right tool for the job."
+++

There's a particular flavor of dread that comes at 2 AM when a production system starts failing. It's worse when you already know the cause, worse still when you've known about it for weeks, and the fix was "scheduled for next sprint." This is the story of how a dying artifact server, a hard API incompatibility, and a one-week window forced me to perform surgery on a running system — without stopping the patient's heart.

## The Patient: A Very Tired Artifact Server

Every CI/CD pipeline has one: the workhorse that nobody thinks about until it stops working. In our case, it was an artifact server — the system responsible for storing, versioning, and serving compiled components to production. Build something, push the artifact, production pulls it when a component needs to be deployed or rebuilt. Simple, boring, reliable.

Until it wasn't.

The server had been running on an end-of-life version for longer than anyone was comfortable admitting. No security patches, no bug fixes, just the same binary chugging along on the same VM, doing its job. Then one day it didn't. Filesystem corruption — the kind that doesn't announce itself cleanly. Artifacts started returning malformed data. Checksums failed. Component rebuilds started erroring out in ways that pointed squarely at the server, not the components themselves.

Here's the saving grace, and also the complicating factor: **corruption only affected component rebuilds**. If a component was already deployed and running, it stayed running. Production was stable — for now. But the moment any component needed to be rebuilt (failed health check, crash loop, memory leak restart), it would fail to pull a clean artifact. That meant a failed component in the middle of the night required a manual intervention to get back up. Every. Single. Time.

We were on borrowed time. The clock was ticking.

## The False Start: When the Backup Is Also Broken

Before we got to that point, we tried the obvious thing first: restore from a known-good snapshot.

We had one. Taken before the filesystem corruption, verified at the time, sitting in cold storage exactly for situations like this. We spun it up, pointed a test environment at it, and ran a rebuild.

It failed.

Not with a filesystem error — the snapshot's storage was clean. Something else was wrong. After some digging, the culprit appeared to be in the artifact server binary itself, not the data it was serving. Our best theory, and we never fully confirmed it, was a date or time library hitting an edge case. The software was old enough that this was plausible: a dependency that worked fine when the snapshot was taken, but whose behavior had drifted into broken territory somewhere in the interim — a year-2038-style overflow in a 32-bit timestamp field, or a daylight saving time boundary that the library mishandled, or simply a relative date calculation that assumed "now" would always fall within a range that it no longer did.

Whatever the cause, the snapshot was useless. The filesystem was fine. The binary was not. And since this was an EOL artifact server, there was no patch coming and no supported upgrade path — the vendor had long since moved on.

This was the moment "migrate to the new server" stopped being a plan for next sprint and became the only option on the table.

## The Complication: A New Server That Speaks a Different Language

Standing up a new artifact server was the easy part. We had a modern, supported replacement ready to go. We migrated all the artifacts over, verified integrity, and breathed a small sigh of relief.

Then we looked at the API documentation.

The old server used a RESTful API that had grown organically over the years — quirky, inconsistent, but thoroughly understood by the production system that talked to it. Request an artifact? `GET /artifacts/{name}?version={ver}&format=bin`. Get a manifest? A deeply nested JSON structure with its own idiosyncratic conventions that the production codebase had been built around for years.

The new server? Clean, modern, well-designed. And completely incompatible. Different endpoint paths, different authentication patterns, different versioning semantics, different response envelope structures. Both spoke JSON, but they might as well have been different languages. Not a single API call from the production system would work against it without changes.

And those changes? **Scheduled for next week's release cycle.** There was no emergency path to push production code changes faster — change freeze, compliance requirements, the usual corporate immune response to "we need to move fast."

So we had our problem neatly framed: a corrupted server we couldn't use, a working replacement we couldn't talk to, and a production system that might need to rebuild a component at any moment.

## The Diagnosis: We Need a Translator

The solution, when it crystallized, was almost elegant in its simplicity: don't change what the production system sends. Don't change what the new artifact server receives. Put something in the middle that speaks both languages.

A **shim** — specifically, an API translation layer that would:

1. Sit on the old server's DNS endpoint (so production would never know anything changed)
2. Accept requests in the old API format
3. Translate them into the new API format and forward them to the new server
4. Receive the new server's responses
5. Repack them into the old API format
6. Return them to production as if nothing had changed

The production system would think it was still talking to the old server. The new server would see clean, well-formed requests. Nobody would be the wiser until next week's release made the whole thing unnecessary.

## The Operation: FastAPI as a Surgical Instrument

Python's [FastAPI](https://fastapi.tiangolo.com/) is, it turns out, an excellent tool for this kind of work. It's fast to write, fast to run, and its route-matching and request/response modeling make it easy to reason about what's being transformed at each step.

The basic structure looked like this:

```python
from fastapi import FastAPI, Request, Response
import httpx
import json

app = FastAPI()
NEW_SERVER = "https://new-artifact-server.internal"

@app.get("/artifacts/{name}")
async def get_artifact(name: str, version: str, format: str, request: Request):
    # Translate old query params to new API path structure
    new_url = f"{NEW_SERVER}/v2/components/{name}/versions/{version}/download"
    new_params = {"encoding": format}  # old "format" -> new "encoding"
    
    async with httpx.AsyncClient() as client:
        upstream = await client.get(new_url, params=new_params, headers=build_new_headers(request))
    
    # New server returns a different JSON envelope; old server had its own format
    # Repack: extract the binary payload, return it with old-style headers
    payload = extract_binary_from_response(upstream)
    return Response(
        content=payload,
        headers=build_legacy_headers(upstream),
        media_type="application/octet-stream"
    )
```

The manifest endpoint was messier. The old server's JSON format had some genuinely unusual structural decisions — deep nesting, non-standard conventions, the kind of thing that grows organically in a codebase over years and becomes load-bearing before anyone realizes it. I won't get into the specifics since that format is proprietary, but suffice to say the translation logic was the most painstaking part of the whole exercise. Empty arrays that needed to be null in one direction, required fields that the old system expected even when absent, subtle differences in how version identifiers were represented.

Each of these got its own translation function, tested against captured request/response pairs from the old server's logs before we'd lost confidence in it.

The whole shim ended up being around 400 lines of Python — not large, but dense with the specific knowledge of both APIs' quirks.

## Deployment: The Tricky Part

The shim itself was straightforward. Deploying it transparently was trickier.

The plan:

1. Deploy the FastAPI app on the same host as the old artifact server (which was still running, just unreliable — we needed its DNS record)
2. Change the listening port of the old server (to get it out of the way while keeping it around for reference)
3. Bind the FastAPI shim to port 80/443 on that host
4. The DNS record never changes. Production never changes. The shim just... appears.

We kept the old server running in its degraded state on a non-standard port as a fallback reference. If the shim's translation was wrong for some edge case, we could compare against what the old server actually returned (on the few requests it could still handle correctly).

The first live test was terrifying in the way only production deployments can be. We triggered a manual component rebuild. The request hit the shim, got translated, went to the new server, came back, got repacked. The component deployed cleanly.

We did it a few more times with different components. Each one worked.

Then we went to sleep — for the first time in a few nights without the pager anxiety that a 3 AM rebuild failure was coming.

## The Recovery: One Week Later

When the scheduled production code update went out the following week, the changes were straightforward: update the artifact server URL and port in the configuration, swap out the API client for one that spoke the new format, remove the legacy translation logic that had been accumulating in the production codebase for years.

Once that deployed, we turned off the shim. It had done its job and was no longer needed. Total operational life: nine days.

## What This Taught Me

**Shims are a legitimate engineering tool, not a hack.** There's a tendency to feel a little dirty about translation layers — like you're admitting defeat, papering over the real problem. But the real problem here *was* being solved (new artifact server, modern API, updated production client). The shim just made sure the solving happened on a schedule that didn't compromise production stability.

**Put the translation at the seam, not in the callers.** The power of this approach is that it's completely transparent to everything except the shim itself. Production code didn't change. The new server didn't know about the translation. All the complexity was contained in one small, testable, replaceable service sitting at the boundary between two systems.

**FastAPI is surprisingly good for this.** The async HTTP client (`httpx`), the clean route definitions, and the ability to inspect and manipulate requests and responses at every level made it fast to write and easy to reason about. For a shim that needed to be written, tested, and deployed under pressure, that matters.

**Capture everything while you can.** The most valuable thing we did in the early hours was pull logs from the old server — real request/response pairs that let us build test cases for the translation logic before we ever pointed it at production. By the time we deployed, we had confidence that the shim handled every call pattern the production system actually used.

**EOL software doesn't just stop getting fixes — it accumulates invisible risk.** The snapshot failure was the most unsettling part of this whole incident. The filesystem was clean. The data was intact. But the binary itself had quietly become unreliable, probably due to some date or time library that had silently crossed into broken territory. That's the insidious thing about end-of-life software: it doesn't always fail loudly at the moment it goes EOL. It fails later, in ways you didn't anticipate, when you need it most. The snapshot that should have been our safety net had the same rot in it as the system it was meant to replace.

**Know your blast radius.** This only worked because the failure mode was narrow: component rebuilds, not running components. Before writing a single line of the shim, we spent time understanding exactly what would break and when. That clarity made the solution possible.

---

There's something clarifying about a genuine emergency. The ideal solution — update the production client, migrate cleanly, test thoroughly — wasn't available. What was available was a week, a functional new server, and Python. Sometimes that's enough.

The system never knew it was on the table.

---

*Have you had to do emergency API shimming in production? I'd love to hear how you approached it — especially if your constraints were different.*
