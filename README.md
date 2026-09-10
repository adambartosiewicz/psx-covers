> **This is a fork of [xlenore/psx-covers](https://github.com/xlenore/psx-covers).**
> Upstream is the canonical collection and stays the source of the raw URLs below.
> Changes made here are not automatically upstreamed.

*⭐**Star this repo if it was useful to you**⭐*

- [Covers Stats](https://github.com/xlenore/psx-covers#covers-stats "Covers Stats")
- [PSCoverDL App](https://github.com/xlenore/psx-covers#PSCoverDL)
- [Duckstation Setup](https://github.com/xlenore/psx-covers#duckstation-setup "Duckstation Setup")
- [Development](#development "Development")

## PSCoverDL

[![](https://user-images.githubusercontent.com/57191159/275665605-4c4b3042-85e4-45b5-8f1b-48a6f00a93ea.png)](https://user-images.githubusercontent.com/57191159/275665605-4c4b3042-85e4-45b5-8f1b-48a6f00a93ea.png)

Small tool to download PS1/PS2 covers for DuckStation and PCSX2.
You can download it from here: [PSCoverDL](https://github.com/xlenore/pscoverdl "PSCoverDL")

## Duckstation setup

[![](https://i.imgur.com/8rD1P5C.gif)](https://i.imgur.com/FJWeE0e.gif)

DuckStation has its own cover downloader, upgrade to version **0.1-5227** or higher.
- Open DuckStation
- Tools -> Cover Downloader
- Use this URL for default covers
  ```python
  https://raw.githubusercontent.com/xlenore/psx-covers/main/covers/default/${serial}.jpg
- or use this one for 3D covers.
  ```python
  https://raw.githubusercontent.com/xlenore/psx-covers/main/covers/3d/${serial}.png
- Check "Use Serial Files Name"
- Click Start
- Enjoy :)

## Covers Stats

| Serial |  Available/Total |  Percentage  |
| ------ |  --------------- |  ----------  |
| CPCS | 1/1 | 100.00% |
| ESPM | 3/7 | 42.86% |
| HASH | 0/25 | 0.00% |
| HPS | 2/2 | 100.00% |
| LSP | 79/195 | 40.51% |
| PAPX | 3/68 | 4.41% |
| PBPX | 0/11 | 0.00% |
| PCPD | 0/1 | 0.00% |
| PCPX | 54/127 | 42.52% |
| PEPX | 0/1 | 0.00% |
| PL | 0/1 | 0.00% |
| PSRM | 0/1 | 0.00% |
| PTPX | 0/1 | 0.00% |
| PUPX | 0/2 | 0.00% |
| SCAJ | 1/2 | 50.00% |
| SCED | 50/354 | 14.12% |
| SCES | 591/616 | 95.94% |
| SCPM | 0/4 | 0.00% |
| SCPS | 398/518 | 76.83% |
| SCUS | 188/391 | 48.08% |
| SCZS | 0/6 | 0.00% |
| SIPS | 18/20 | 90.00% |
| SLED | 1/236 | 0.42% |
| SLES | 2264/2318 | 97.67% |
| SLKA | 3/3 | 100.00% |
| SLPM | 1119/1787 | 62.62% |
| SLPS | 3083/3662 | 84.19% |
| SLSA | 0/3 | 0.00% |
| SLUS | 1218/1316 | 92.55% |
| SPUS | 0/1 | 0.00% |

## Development

This repository is an asset collection first. The Python files in `tools/` are
maintenance scripts, not an application.

### Setup

The tools carry no dependency manifest — there is no `requirements.txt` and no
lockfile. Install the packages by hand:

```bash
python -m venv .venv
source .venv/bin/activate
pip install pillow numpy "opencv-python>=5,<6" "prompt_toolkit>=3,<4"
```

`tkinter` comes with the standard library. `save_cover.py` needs it.

### Repository layout

```
covers/
  default/     500x500 JPEG   2D covers, filename = serial
  3d/          226x226 PNG    3D covers, filename = serial
tools/
  gamedb.json                       the expected-serial list
  save_cover.py                     clipboard  ->  covers/default/
  update_cover_size.py              normalise covers/default/ to 500x500
  update_stats.py                   stats table + missing_covers.txt
  2D_to_3D_cover/
    2D_to_3D_cover.py               perspective warp + case overlay
    overlays/*.png                  4 fixed 226x226 overlays
missing_covers.txt                  generated - do not edit by hand
```

### Naming

A cover's path and filename are the public API. DuckStation configs point at
them directly, so never rename, move, or re-case a file that already exists.

- The filename is the game serial in uppercase, with the publisher hyphen:
  `SLUS-00001.jpg`.
- A multi-disc variant appends the disc number: `SLPS-02020-disc2.jpg`.
  Disc 1 carries no suffix.
- `HASH-<hex>` and `LSP-<digits>` are valid serial forms. Do not "correct" them.

### The tools

#### `2D_to_3D_cover.py` — make a 3D cover from a 2D one

Run it from its own directory. The script resolves `overlays/` and `output/`
against the working directory, so it finds no overlay from the repository root.

```bash
cd tools/2D_to_3D_cover
python 2D_to_3D_cover.py                       # interactive prompts
python 2D_to_3D_cover.py \
  --base ../../covers/default/SLUS-00001.jpg \
  --overlay DEFAULT                            # no prompts
```

`--overlay` takes a name, a list number, or a path to a `.png`. The four names
map to real PS1 case variants:

| Number | Name | File |
| --- | --- | --- |
| 1 | `DEFAULT` | `overlays/default.png` |
| 2 | `SIDE_LOGO` | `overlays/side_logo.png` |
| 3 | `GREATEST_HITS` | `overlays/greatest_hits.png` |
| 4 | `GREATEST_HITS_NO_COLUMN` | `overlays/greatest_hits_no_column.png` |

The script warps the cover at 500x500, then downsamples to 226x226 with
`INTER_AREA`. It writes `tools/2D_to_3D_cover/output/<stem>.png`. It does **not**
write into `covers/3d/`. Review the result by eye, then copy it across by hand.

#### `save_cover.py` — clipboard to `covers/default/`

```bash
python tools/save_cover.py
```

The script polls the clipboard every 0.5 seconds and runs until you press
Ctrl+C. It waits for an image, then waits for a text string containing `-` to
use as the filename. It resizes to 500x500 and saves into `covers/default/`.

#### `update_cover_size.py` — normalise to 500x500

> **Warning:** this script rewrites images in place. There is no dry-run flag
> and no backup. It lists what it will touch and blocks on a prompt first.

```bash
python tools/update_cover_size.py
```

`IGNORE_LOWER_RESOLUTION = True` is deliberate. An image smaller than 500x500 is
left alone rather than enlarged. Do not upscale.

#### `update_stats.py` — coverage stats

```bash
python tools/update_stats.py
```

The script compares `tools/gamedb.json` against the files in `covers/default/`.
It overwrites `missing_covers.txt`, and it prints the Covers Stats table to
standard output. Pasting that table into this README is a manual step.

### Adding a cover

1. Save the 2D cover as 500x500 JPEG into `covers/default/<SERIAL>.jpg`.
2. Add a 3D cover only alongside its 2D source, or when the 2D cover already
   exists.
3. Run `python tools/update_stats.py` and paste the new table into this README.
4. Commit one cover, or one small related set, per commit. A human reviews this
   repository by eye.

### Conventions

- Commit subjects are lowercase and imperative, and they name the serial:
  `add SLUS-00401 3D cover`.
- Work on a branch named after the serial, then open a pull request.
- Integrate by rebase. Do not merge.
- Never mass-rewrite the image tree. A bulk resize or recompress produces a
  five-figure diff that nobody can review.
- Flag any new Python dependency before you import it. If you add one, add a
  manifest in the same change.
- Treat `missing_covers.txt` and the Covers Stats table as generated. Change the
  input, then re-run `update_stats.py`.

## Credits
* psxdatacenter.com
* duckstation.org
