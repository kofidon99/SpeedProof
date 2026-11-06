# SpeedProof
Offline-first Android journey recorder for Ghana. Logs raw GPS speed into a signed, tamper-evident hash chain, with optional road photos and free public timestamps (OpenTimestamps, RFC 3161). Exports a PDF plus a ZIP bundle anyone can check with an independent verifier. Pilot: GPS speed is recorded, not certified.

The pilot APK is built and signed: SpeedProof-pilot-0.1.0.apk (87 KB). It runs on Android 8 and newer.

How to install: copy the APK to the phone, open it, and allow “install unknown apps” when Android asks.

What’s in it

Trip recording: you start and stop trips yourself, and can pause and resume. Recording runs as a foreground service with a visible notification, so you don’t need background location permission. It reads raw GPS (not Google’s fused location) once per second, or 5 times per second if you choose. Each reading stores satellite count, signal strength, the phone’s own accuracy estimates, mock-location flags, and the difference between the phone clock and GPS time. Motion sensors are summarised every second.

Tamper evidence: every event is linked into an encrypted hash chain on the phone. Every 10 seconds the chain is signed with a hardware-backed P-256 key (StrongBox if the phone has it), and the phone’s key attestation certificates are included when available.

Honest gaps: loss of GPS is recorded as a gap with start and end times. If the app is killed or the phone reboots, the next launch records the interruption and closes the trip. Nothing is filled in.
Optional road photos: off by default. When on, photos are taken every 15, 30 or 60 seconds, and each photo’s hash goes into the chain the moment it’s taken. Failed captures are recorded too.
Free independent timestamps: while online, the app sends only a hash of the chain to OpenTimestamps (anchored in Bitcoin) and FreeTSA (RFC 3161). Offline trips stay pending and are retried later. There’s no backend and no smart contract.
Disputes: you enter the alleged time and speed, and the app shows a ±2 minute chart and table. It then exports a PDF report plus a ZIP evidence bundle and opens the share sheet (WhatsApp, email and so on). If the alleged time isn’t covered by the recording, the report says “no matching evidence.”
Phone-specific help: a battery-optimisation prompt, plus tips for Tecno, Infinix and itel phones.

What is tested: the evidence engine and speedproof-verify.py, end to end. Trips verify cleanly. Editing one speed, deleting one event, or swapping a photo each make verification fail. I also checked that every newer-Android call is guarded on Android 8, that the timestamp request is valid, and that the .ots files parse with the official library.

First things to check post install:
Start a trip, turn the screen off, drive 10 minutes, and confirm the fixes kept coming. On Transsion phones this is the most likely failure.
Turn photos on and see how many succeed with the screen off. Many phones block the camera then, and the failures will be recorded.
Export a trip and run python3 speedproof-verify.py <zip> on a computer.
Force-stop the app mid-trip, reopen it, and check that the trip shows as interrupted.
