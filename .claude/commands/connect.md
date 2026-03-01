Set up Multi-User Editing (Concert) for remote collaboration over Tailscale.

## What to do

1. Ask the user for the other person's Tailscale IP (unless they provide it in the prompt)
2. Read the current `Config/DefaultEngine.ini` at `/mnt/d/unreal-projects/MyProject/Config/DefaultEngine.ini`
3. Ensure these sections exist (add if missing, don't duplicate):

```ini
[/Script/ConcertClient.ConcertClientConfig]
bInstallEditorToolbarButton=True

[/Script/UdpMessaging.UdpMessagingSettings]
EnabledByDefault=True
EnableTransport=True
UnicastEndpoint=0.0.0.0:6668
MulticastEndpoint=230.0.0.1:6666
+StaticEndpoints=[OTHER_PERSON_TAILSCALE_IP]:6668
```

4. Update the `+StaticEndpoints` line with the other person's IP and port 6668
5. Remind the user:
   - The other person needs YOUR Tailscale IP in THEIR StaticEndpoints (also port 6668)
   - Both restart UE after config changes
   - Host: toolbar button → Launch Server → Create Session → Join
   - Joiner: toolbar button → Browse → Join the session
   - WARNING: joining can sync blank state — leave session to revert

## Key gotchas
- UnicastEndpoint MUST be a different port than multicast (use 6668, NOT 6666) or the socket bind fails
- StaticEndpoints port must match the OTHER person's UnicastEndpoint port (6668)
- The toolbar button config section is `ConcertClient.ConcertClientConfig` (NOT `ConcertSyncClient`)
- `StaticEndpoints` is an array — MUST use `+` prefix in ini files
- Multicast doesn't work over Tailscale — static endpoints are required
- If settings don't persist from UE Project Settings UI, edit the ini file directly
- `Saved/Config/WindowsEditor/Engine.ini` can override `Config/DefaultEngine.ini` — check both
