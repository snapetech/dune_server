# Specialization XP Multipliers

## Production visible 3x mode

Applied on `kspls0` on 2026-09-06.

Production uses the native Landsraad mission reward multiplier:

```ini
dw.LandsraadMissionRewardMultiplierSpecializationXP=3.0
```

The setting is present in all three tracked engine overlays:

- `config/UserEngine.ini`
- `config/UserEngine.deep-desert.ini`
- `config/UserEngine.deep-desert-pvp.ini`

The database trigger `dune.specialization_tracks_apply_3x_xp_rate` is disabled in
the live `dune_sb_1_4_10_0` database. This keeps the native multiplier from
stacking with the database multiplier. In visible mode, a base 125 XP Landsraad
mission reward should display and persist as 375 XP, not 1125 XP.

This is a Landsraad mission reward setting. Disabling the database trigger also
means other specialization XP sources are no longer globally tripled by that
database-side path.

## Activation and validation

Game processes read the cvar at startup. The production fleet was restarted with
`scripts/restart-target.sh all` after the config deployment, so the setting took
effect immediately after that restart; it does not wait for the next scheduled
downtime.

The live rollout verified:

- the cvar in all three engine overlays;
- the database trigger state as disabled (`D`); and
- all 35 map containers running after the restart.

No player XP rows were edited. End-to-end popup validation still requires
completing a Landsraad mission; the expected visible reward for a 125 XP base
reward is 375 XP.

The pre-change live config backup is at:

```text
kspls0:/home/keith/Documents/code/DuneAwakeningSelfHost/backups/manual/specialization-xp-visible-20260906T213825Z/
```

## Rollback

Rollback must be performed on `kspls0` and followed by the approved fleet
restart path:

1. Restore or remove the `dw.LandsraadMissionRewardMultiplierSpecializationXP`
   line from all three engine overlays.
2. Re-enable the database trigger:

   ```sql
   ALTER TABLE dune.specialization_tracks
     ENABLE TRIGGER specialization_tracks_apply_3x_xp_rate;
   ```

3. Run `scripts/restart-target.sh all` and verify the trigger and map health.

Do not enable both the native cvar and the database trigger unless a 9x total
multiplier is intentionally desired.

## Operational note

The rollout restart also reported an existing logoff-timer runtime auto-remap
warning for `deep-desert-pvp`; it is unrelated to specialization XP.
