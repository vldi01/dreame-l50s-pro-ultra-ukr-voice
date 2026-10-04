---
name: l50-uk-voice
description: >-
  Build and install a custom Ukrainian voice pack for the Dreame L50s Pro Ultra
  (dreame.vacuum.r5039h, Home Assistant vacuum.pilip). Use when replacing ogg
  clips, packing a tar.gz, computing md5/size, hosting a pack, or calling
  dreame_vacuum.vacuum_install_voice_pack.
---

# L50 Ukrainian voice pack

Official clips and phrase meanings live in this repo. Read `README.md` and `phrases.csv` before replacing anything. Do not install a pack on the robot until the user explicitly says to.

## What the robot plays

- 610 mono 16 kHz Ogg Vorbis files, named `N.ogg`, plus the json sidecars from `voice/`.
- `phrases.csv` is a human index. It must not go inside the archive.
- `tts.json` IDs are not ogg filenames.
- On `dreame.vacuum.r5039h` these files are the ones actually played:

| Intent | File to replace |
|---|---|
| Charging (classic 18) | `856.ogg` |
| Move robot to the dock (17) | `863.ogg` |
| Remove from dock before power off (19) | `864.ogg` |
| Place on dock first (112) | `858.ogg` |
| New environment, returning (140) | `859.ogg` |
| Washboard water level (663) | `823.ogg` |

Replacing only `18.ogg` does nothing on this model.

## Encode one clip

Homebrew ffmpeg has no `libvorbis`. Its native `vorbis` encoder is stereo-only. Pipe wav into `oggenc` from `vorbis-tools`.

```bash
ffmpeg -i input.wav -ar 16000 -ac 1 -f wav - | oggenc -Q -q 4 -o voice/7.ogg -
afinfo voice/7.ogg   # Ogg Vorbis, 16000 Hz, 1 channel
```

Keep the original filename. Do not add a folder around the files.

## Pack

Copy `voice/` to a build directory, overwrite only the clips being changed, and leave every other ogg and all sidecars (`dmr_audio.json`, `first_audio.json`, `mini_broad.json`, `tts.json`, `voice_mapping.json`, `time.txt`).

```bash
COPYFILE_DISABLE=1 tar -czf pack.tar.gz -C build \
  --exclude phrases.csv --exclude README.md --exclude .DS_Store .
```

The archive members must be `7.ogg`, not `voice/7.ogg` or `build/7.ogg`. Check:

```bash
tar -tzf pack.tar.gz | grep -c '\.ogg$'    # 610
tar -tzf pack.tar.gz | grep -E '(^|/)\./'  # empty is fine; no nested folder name
md5 -q pack.tar.gz
stat -f%z pack.tar.gz
```

`md5` and `size` are of the `.tar.gz` bytes, not the uncompressed tar.

## Install

The robot downloads a URL. A local path does not work. Put `pack.tar.gz` on a public HTTPS URL the vacuum can fetch (GitHub release asset of this repo, or another host that does not require login). Redirects sometimes fail; prefer a direct file URL.

Call this only after the user confirms the URL. Entity in this household is `vacuum.pilip`.

```yaml
action: dreame_vacuum.vacuum_install_voice_pack
target:
  entity_id: vacuum.pilip
data:
  lang_id: "MEME"
  url: "https://example.com/pack.tar.gz"
  md5: "<md5 of pack.tar.gz>"
  size: 12345678
```

`lang_id` is a short string stored as the current pack id (`VOICE_CHANGE`, siid 7 piid 4). Use a custom id such as `MEME`. `UK` is the official Ukrainian pack, and the Dreame app can overwrite a custom pack that reuses that id.

Afterward, `voice_packet_id` on the device should match `lang_id`. Do not reboot or reinstall the official pack unless the user asks.
