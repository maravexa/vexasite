+++
title = "KISS the Ring: Building a Prometheus Exporter for Network Inventory"
date = 2026-04-03
description = "Why I built a single-binary Prometheus exporter for network inventory instead of deploying another IPAM server — and how constraint, Go, and an existing monitoring stack got me further than any purpose-built solution."
[taxonomies]
tags = ["observability", "golang", "homelab", "security"]
[extra]
hero = "signet-exporter-hero.webp"
hero_alt = "Signet ring pressing a wax seal onto a network topology diagram"
+++

# KISS the Ring: Building a Prometheus Exporter for Network Inventory

There were several reasons I created this exporter. Primarily I wanted a simple open-source IPAM solution but wasn't satisfied with anything currently available. I had used phpIPAM in the past but as a security professional I wanted a more modern and secure solution. That ruled out JavaScript, PHP, React, and a whole host of other languages. My preference for memory safety and enhanced security constrained me to Rust and Go, the memory-safe languages I was most familiar with.

I also didn't want to have to spin up and maintain yet another server in my homelab. A series of events led to a hardware failure on a vhost bringing my cluster down one. The price of RAM lately has discouraged upgrading the remaining node, and my compute demands on my other workstations have increased lately with my AI research so I didn't have as much breathing room. Necessity was the mother of invention, and constraint was the mother of flexibility here.

What resulted was natural for any observability pro. I'd just shoehorn an IPAM solution into my existing monitoring stack, and while I'm at it let's consolidate the functions of some other exporters that I have been frustrated with into this project, namely the blackbox exporter. Thus the signet-exporter was born.

The signet-exporter owns the unclaimed space of "what's actually alive on my network and has it changed." The name is a nod to signet rings — the wax-seal authentication artifacts that verified identity and provenance. It's also literally sig+net: security on the network. Historically resonant, security-coded, and nobody in the Prometheus ecosystem was using it.

## What Signet Actually Does

Signet is a Prometheus exporter for network inventory observability. It's a single Go binary that performs subnet scanning, ARP-based duplicate detection, MAC-IP binding stability tracking, DNS forward/reverse consistency checks, and rogue device detection. You point it at your subnets, it scans them on a schedule, and it exposes everything it finds as Prometheus metrics on `/metrics`. No database server, no web frontend, no runtime dependencies. Just a binary, a YAML config, and your existing Prometheus/Grafana stack.

The design target was air-gapped and compliance-heavy environments — government contractors, FedRAMP shops, anyone who cares about NIST 800-53. The binary runs as an unprivileged system user with nothing but `CAP_NET_RAW`. It refuses to start as root. The systemd unit ships with `ProtectSystem=strict`, `NoNewPrivileges`, and a locked-down `CapabilityBoundingSet`. If you're the kind of person who reads systemd hardening guides for fun, you'll appreciate the unit file.

On the build side, there's no CGo. Raw sockets come from `mdlayher/packet` instead of gopacket/libpcap, which means fully static builds and distroless containers are trivial. There's a FIPS variant built with `GOEXPERIMENT=boringcrypto` for environments that require it. Releases are signed with cosign keyless signing via Sigstore.

A key architectural decision is that scans run in the background on timers, not on scrape. When Prometheus hits `/metrics`, it reads cached state instantly. This prevents scrape timeouts and decouples scan timing from Prometheus scrape intervals — a lesson anyone who has fought with the blackbox exporter's on-scrape model will appreciate.

## The Evolution

### v0.1.0 — The Vertical Slice

The first release was about proving the concept end-to-end: an ARP scanner feeding a memory state store, a Prometheus collector with a custom registry (no default Go runtime metrics polluting the output), a scheduler running background scan loops, and an HTTP server serving `/metrics`, `/health`, and `/ready`. CI/CD was in place from day one with GitHub Actions, goreleaser, cosign signing, and the FIPS build variant.

The custom registry matters more than it sounds. With Signet, the absence of a metric means something. If `signet_unauthorized_device_detected` doesn't appear in the output, that means no rogue devices were found — not that the instrumentation is missing. Clean, auditable output was a first-class goal.

### v0.2.0 — Scanner Coverage

Phase 2 added ICMP scanning (unprivileged socket with `CAP_NET_RAW` fallback), DNS enrichment with forward/reverse consistency checks, and a TCP port scanner with a connect-based worker pool. This established the scanner execution pipeline that still holds: ARP → ICMP → DNS → Port.

The distinction between discovery scanners and enrichment scanners was formalized here. ARP and ICMP discover hosts and create records. DNS and Port read the state store for known hosts and add data to them. The ordering guarantee ensures host records exist before enrichment runs. Each scanner follows a nil-guard pattern on state updates — a DNS scan that doesn't know a host's MAC won't clobber the MAC that ARP already discovered. Sounds obvious, but getting this wrong is how you end up with state stores full of half-populated records.

### v0.3.0 — Intelligence Features

This is where Signet went from "network scanner with Prometheus output" to something with actual operational value. The DNS enrichment scanner gained hostname lookup, so a single scan cycle now resolves the full identity of every host on the network: IP, MAC, vendor OUI, and hostname. OUI vendor lookup from the IEEE database resolves MACs to manufacturers. Structured audit logging via slog emits JSON-line events for every meaningful state change. MAC-IP change detection tracks binding stability and fires audit events when something moves. Per-subnet MAC allowlists enable rogue device detection with a three-state authorization model (authorized, unauthorized, and unchecked — because in Go, `false` is also the zero value, and you need to distinguish "explicitly unauthorized" from "we haven't checked yet"). Duplicate IP detection catches ARP conflicts.

The three-state authorization model deserves a note. A naive boolean `Authorized` field can't distinguish "we checked the allowlist and this device isn't on it" from "we haven't loaded the allowlist yet." The solution is a companion `AuthorizationChecked` flag: if false, the authorization field is meaningless and shouldn't be touched. If true, the value of `Authorized` is an explicit decision. The same pattern handles duplicate detection. It's a small thing, but it prevents an entire class of bugs where enrichment scanners inadvertently reset fields to their zero values.

### v0.3.1 — Packaging and Polish

Distribution packages via goreleaser's nFPM: `.deb`, `.rpm`, `.pkg.tar.zst`. A postinstall script creates the `signet` user and sets capabilities on the binary. A preremove script cleans up the service. An OUI update helper script keeps the vendor database current. The systemd unit got its full hardening treatment. Makefile targets for `setcap`, `install`, and `install-data` round out the developer experience.

This release was about making Signet something you could hand to another operator and have them running in minutes, not hours.

### v0.4.0 — Hardening

The most recent release focused on making Signet production-grade. mTLS enforcement landed with full client certificate verification, cert rotation on SIGHUP, and a `--generate-certs` flag for dev quickstart. The state store moved from memory-only to bbolt, so network inventory now survives restarts. Audit logging got its CEF format option for direct SIEM ingestion. The FIPS build variant now has end-to-end BoringCrypto verification in CI with a `signet_exporter_fips_enabled` metric. And config hot-reload via SIGHUP means you can update subnet lists, scan intervals, ports, and allowlists without bouncing the service — only the listen address and TLS config require a restart.

## What's Next

With v0.4.0 shipped, the path to v1.0 is short. The remaining checklist items are SBOM generation automated in the release pipeline, a SECURITY.md with real contact info, and at least one external deployment beyond my own lab.

If any of this sounds useful, the project lives at [github.com/maravexa/signet-exporter](https://github.com/maravexa/signet-exporter).
