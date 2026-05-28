# Jam Jam — downloadable audio assets

> This repository hosts the **audio models** the Jam Jam Android app fetches
> at first launch. It exists so the app's APK can stay under the Play Store
> size budget by lazy-downloading the heavy bits.
>
> The Jam Jam application source code is **NOT** here — keep it that way.

## Contents

| File | Size | Purpose | License |
|---|---|---|---|
| `MuseScore_General.sf3` | 39 MB | General-MIDI SoundFont used for chord previews + backing tracks | CC-BY-4.0 (MuseScore, see [`LICENSE-MuseScore_General`](./LICENSE-MuseScore_General)) |
| `btc_model_170_ft.onnx` | 13 MB | Bi-directional chord-recognition ONNX (170 classes, fine-tuned). Drives offline song analysis. | Apache-2.0 (BTC-ISMIR2019 derivative) |
| `btc_model_170_causal.onnx` | 5.8 MB | Causal variant of the same model — fast path for real-time chord detection (no look-ahead delay). | Apache-2.0 (BTC-ISMIR2019 derivative) |

Total ≈ 58 MB.

## How the app finds these

The Jam Jam app downloads each file via its raw GitHub URL at first use:

```
https://github.com/<your-username>/jam-jam-assets/releases/download/v1/<file>
                                                                  ^^ tag — bump on every update
```

We use **GitHub Releases** (not raw git push) so each file gets a stable
permalink, the repo stays small, and rolling out a new soundfont is a
one-click action. The release tag is wired into
`src/services/audio-model-registry.ts` (added in Phase 2).

## Integrity

Every asset is delivered with a SHA-256 fingerprint the app re-checks after
download. Mismatches raise a clear UI error instead of silently shipping a
corrupted or tampered binary.

| File | SHA-256 |
|---|---|
| `MuseScore_General.sf3` | `5b85b6c2c61d10b2b91cddd41efcce7b25cd31c8271d511c73afafbef20b6fa3` |
| `btc_model_170_ft.onnx` | `ad624d5be6212bbf4cde5b84b60ce0523ec695061e072ed4192f768e5662141e` |
| `btc_model_170_causal.onnx` | `724db9f3fda8a6a5d72a5b51a6d2d25ad60332cd330baf548ced081bd1723041` |

`SHA256SUMS.txt` ships next to each release for scripted verification.

## Why a separate repo

* The Jam Jam app source is private and must never touch GitHub.
* These three files are the only assets that legitimately leave the app
  bundle — keeping them in their own tiny repo means the app build stays
  smaller and updates to the soundfont don't churn the main repo.
* GitHub Releases is free for public repos with no bandwidth cap.

## Updating a file

```bash
# 1. Replace the file locally (this directory)
cp /path/to/new-soundfont.sf3 ./MuseScore_General.sf3

# 2. Re-emit the integrity manifest
sha256sum *.sf3 *.onnx > SHA256SUMS.txt

# 3. Commit + tag a new release
git add . && git commit -m "Update SoundFont to vX.Y"
git tag v2 && git push origin main --tags

# 4. Attach the file to the GitHub Release (one-shot)
gh release create v2 *.sf3 *.onnx SHA256SUMS.txt -n "Release notes…"
```

After step 4, update `src/services/audio-model-registry.ts` in the Jam Jam
app to point at the new `vX.Y` tag, and ship a new app version.
