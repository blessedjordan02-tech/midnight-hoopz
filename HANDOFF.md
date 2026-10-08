# MIDNIGHT HOOPZ — handoff

State of the Roblox Studio work as of the end of the "UI connection to Roblox Studios"
session (Sep 6 – Sep 14, 2026). That work was done live in Roblox Studio through the
Studio MCP on the owner's PC; the place files are the source of truth, not this repo.

## Places (universe 9925603818)

| Place | PlaceId | Notes |
|---|---|---|
| Park | 111970808421179 | `ServerType="Park"`, 6 courts (1v1 ×2, 3v3 ×2, 2v2 ×2), event + plates |
| 1v1 Matchmaking | 112709498543636 | `ServerType="Gym"` |
| 2v2 Matchmaking | 122356163279849 | `ServerType="Gym"` |
| MY COURT | 118411994582073 | `ServerType="Gym"` |
| Main Menu | 111624265962938 | Lobby only — no gameplay, **no admin scripts needed** |

Do not edit: Rim Rush (117976871251709, different game), `PRAC Park_*.rbxlx` (local copy),
`Place1 (Version 8)` (reference only).

## Publish checklist (nothing was ever confirmed published)

- [ ] Publish **Park** (everything at once so the v3 reset lands with the economy fixes)
- [ ] Publish **1v1**, **2v2**, **MY COURT**
- [ ] Publish **Main Menu** (JoinPlayerHandler edit)
- [ ] Park `workspace.HeadLogo.ParkTakeover.Image` + `ParkEventConfig.EventLogoImageId` — both empty
- [ ] Park `workspace.HeadLogo.VIP.Image` — empty (other places use `rbxassetid://103680902663440`)
- [ ] Park Workspace: `StreamingMinRadius = 512`, `StreamingTargetRadius = 1536` (not scriptable)
- [ ] Asset permissions: sounds 76491257654685, 126499891956718, 134672250801211,
      102918164522524, 132694932798232, 14295107949; texture 102905143812530;
      sound 9118772490 is the wrong asset type
- [ ] Trail templates for Electric / Spark / Spark2 green effects

### Check before publishing: event clock

The Park event window was stamped during a Studio playtest: start 2026-09-14 20:02 UTC,
end 2026-09-18 20:02 UTC, stored in `ParkEventState_v2` key `window_ParkTakeover_2026_09_S3`.
If that write reached the live DataStore, the event is **already over** and will show as
ended on publish. Restarting it = bump `ParkEventConfig.EventId` (e.g. S3 → S4).

## Last task — admin panel in every server (done, unreported)

- `AdminService` + `AdminPanelClient` are byte-identical and enabled in Park, 1v1, 2v2, MY COURT
  (rewrote the old `*_DISABLED_2026_09_05` copies).
- Cross-server bus: MessagingService topic `MNH_Admin_v1` — announcements, head logos,
  mascots, bans now reach every server in every place.
- Kick/ban work game-wide by name or UserId (new name box in the panel); no longer need to be
  in the target's server.
- Not yet verified live: cross-server delivery, admin mascot grants outside Park.

## Open work

**Matchmaking / multi-place**
- "Matchmaking start glitch" (#11) — skipped by request, never investigated
- Forfeit winner disagreement: `CourtMain` picks by who left, `ParkMatchmaking` by score
- 1v1 loser still teleports to old "MyCourtFR" (107962634568011) outside Park
- Squad spots never seen working with real players; phone UI unwired; squad leadership
  transfer / travel-together / find-squad not built
- `SquadService` 2v2 teleport queue requires exactly 2 per squad (left alone when `MAX_SQUAD` → 4)

**Gameplay**
- Dunk replication: other players' dunks lag / animation plays twice
- 3v3 inbound positions, dribble input delay, defense close-out speed
- Ball disappearing after a rim bounce; teleport-under-rim (#8, needs repro); MyCourt #5
- Swirl-then-in misses; second shot path near ServerMain ~line 7819
- Sweep findings: blow-by never fires; pushback cooldown watches the wrong sibling
- `ShotRecoveryLock` and `BallHeadGuard` still disabled via workspace attributes
  `DisableShotRecovery` / `DisableBallHeadGuard`

**Systems**
- Pass variants (bounce/flashy/overhead) need a type field in `Pass_Schema`
- Networking rewrite (profile/audit first); 20–25 player load test
- Font sweep of pre-existing UIs (`UITheme` exists only in Park)
- Mobile menu button scaling (which button was never answered)
- Server-side Park tester whitelist gate in `BuildSaveServer`
- 2nd/3rd place event coin rewards (10k / 5k) are placeholders

## Gotchas for whoever edits the places next

- Studio edits are **not live until each place is published**. "Bug is back" usually means unpublished.
- Park, 1v1, 2v2 and MY COURT share scripts — port fixes to all four; anchor edits on verbatim text.
- Code that only scans `Gameplay.HoopSystems` silently misses Park courts — use `allHoopSystems()`.
- A player can have three `leaderstats` folders (Economy, Reputation, SecureStats) — find the one
  holding `MNHCoins`, never trust `FindFirstChild("leaderstats")`.
- Studio MCP: cannot create scripts (only rewrite `.Source`); `.Source` fails at ≥200,000 chars,
  so edit `ServerMain` with `multi_edit`; `replace_all` silently no-ops; Studio IDs change every launch.
- ViewportFrame does not render inside a SurfaceGui; `Destroying` unparents children before handlers run.

## Data stores

- Park (current): `ParkPlayerDataStore_v3`, `ParkGlobalWins_v3`, `MNH_ParkReputation_V3`,
  `ParkGlobalRep_v3`, event `ParkTakeover_2026_09_S3`
- Matchmaking (untouched): `MatchmakingStats_v1`, `MatchmakingGlobalWins_v1`, `MNH_Reputation_V1`
- Resets are done by bumping version suffixes — old data is still there.
