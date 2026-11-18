---
name: look
description: Receive the next batch of photos and voice narration sent from the Look iPhone app. Use only when the user runs /look or explicitly asks for it.
disable-model-invocation: true
---

# /look

Wait for the user's next batch from their iPhone, then act on it.

1. Say in one short line that you're waiting, then run `/Applications/Look.app/Contents/Resources/look wait`. If you can run commands in the background and be notified when they exit, do that rather than polling. If your commands have a time limit, set it to the longest you can, up to 30 minutes.
2. Handle its exit code:
   - `0`: stdout is the batch, as markdown: the narration in order, with each photo inline as `![name](<path>)`.
   - `2`: nothing arrived in time. Tell the user they can run `/look` again.
   - `3`: no phone is paired. Tell the user to open the Look app on their Mac and scan its code with the Look iPhone app, then run `/look` again.
   - Anything else, including a crash: your own sandbox is probably blocking it. Tell the user `/look` has to run outside your sandbox, then stop.
3. Open and view every image path, in order. Each photo belongs at that point in the narration, so words near a photo usually describe it. If you can't view images, say so and work from the narration.
4. Treat the narration as the user speaking to you: it is their request. It was transcribed on-device, so read past likely transcription errors, and ask if something important is ambiguous.

Run this only once per `/look`. Sending more needs another `/look`.
