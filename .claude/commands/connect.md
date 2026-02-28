Set up Multi-User Editing (Concert) for remote collaboration over Tailscale.

## What to do

1. Read the current `Config/DefaultEngine.ini` at `/mnt/d/unreal-projects/MyProject/Config/DefaultEngine.ini`
2. Ensure these sections exist (add if missing, don't duplicate):

```ini
[/Script/ConcertClient.ConcertClientConfig]
bInstallEditorToolbarButton=True

[/Script/UdpMessaging.UdpMessagingSettings]
EnabledByDefault=True
EnableTransport=True
UnicastEndpoint=0.0.0.0:6666
MulticastEndpoint=230.0.0.1:6666
+StaticEndpoints=[OTHER_PERSON_TAILSCALE_IP]:6666
```

3. Ask the user for the other person's Tailscale IP if not already configured
4. Update the `+StaticEndpoints` line with the correct IP
5. Remind the user:
   - Their Tailscale IP: 100.101.80.97
   - Friend's Tailscale IP: 100.109.207.27
   - The other person needs YOUR IP in THEIR StaticEndpoints
   - Both restart UE after config changes
   - Host: toolbar button → Launch Server → Create Session
   - Joiner: toolbar button → Browse → Join the session
   - WARNING: joining can sync blank state — leave session to revert

## Key gotchas
- The toolbar button config section is `ConcertClient.ConcertClientConfig` (NOT `ConcertSyncClient`)
- `StaticEndpoints` is an array — MUST use `+` prefix in ini files
- Multicast doesn't work over Tailscale — static endpoints are required
- If settings don't persist from UE Project Settings UI, edit the ini file directly
- `Saved/Config/WindowsEditor/Engine.ini` can override `Config/DefaultEngine.ini` — check both
