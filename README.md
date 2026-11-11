# NCAR-NEON — offline OSBS forcing fork

This fork of [NEONScience/NCAR-NEON](https://github.com/NEONScience/NCAR-NEON) (Lombardozzi et al., 2023, *Geosci. Model Dev.* 16, 5979–6000, [doi:10.5194/gmd-16-5979-2023](https://doi.org/10.5194/gmd-16-5979-2023)) builds CTSM DATM forcing for the NEON **OSBS** site **offline**, from a pre-downloaded archive of raw NEON products. It is used as a git submodule of the OSBS representative-hillslope repository, whose `scripts/ncar_neon_forcing/` holds the environment build, the raw-data downloader and the run instructions. Upstream's README follows below.

All changes are in `TowerTools_ForcingData/flow.api.clm.R`. The rest of the repository is upstream's.

## What differs from upstream

| Change | Why |
|---|---|
| Missing R packages stop the run instead of installing from CRAN | The conda environment is complete; no live installs mid-run |
| Output written locally (`MethOut` = local) | Upstream uploads to a Google Cloud Storage bucket |
| `doDnld <- FALSE` gates every download and cleanup step | Offline mode |
| Raw archive read from `NEON_DIRDNLD` (the site's directory, e.g. `data/neon_raw/OSBS`); the run stops if it is missing | Replaces upstream's local path |
| Archive zips copied to a per-session `tempdir()` before stacking | `stackByTable`/`stackEddy` modify and delete their inputs; the archive must stay intact |
| Met products stacked from the archive (`stackByTable`) instead of `loadByProduct` | The core online → offline change |
| Flux data clipped to `DATEBGN`/`DATEEND` | Lets a full archive be processed one window at a time. REddyProc needs ≥ 90 days of flux, so use annual windows |
| End-of-run file deletions run only when `doDnld` is TRUE | Never delete the archive |
| Secondary precipitation: DP1.00006.001 → DP1.00045.001 | OSBS's secondary gauge is the tipping bucket |
| Primary-precipitation site list fixed to `"OSBS"` | Removes the last live API call |
| CO₂ flux partitioning made non-fatal; missing CO₂ evaluation columns filled with NaN | OSBS eddy-covariance CO₂ flux is absent at the NEON source. The forcing uses no CO₂ variable |
| Global attributes: `created_by` from `NEON_CREATED_BY` (default: system user), `supported_by` from `NEON_SUPPORTED_BY` (default: upstream's NSF line), `created_from` = file name only | Upstream wrote its author's name and an absolute path |

**This fork is OSBS-specific.** The precipitation changes assume OSBS's instruments; other sites should use upstream.

## Running

Set by environment variable (`METHPARAFLOW=1` makes the script read them):

| Variable | Meaning |
|---|---|
| `SITE` | NEON site code (`OSBS`) |
| `DATEBGN`, `DATEEND` | Window, e.g. `2018-01-01`, `2018-12-31` |
| `DIROUT` | Output base; files go to `$DIROUT/<SITE>/atm/` and `eval/` |
| `NEON_DIRDNLD` | Raw archive for the site |
| `LOWMEM` | `FALSE` (the `TRUE` path fails upstream) |
| `TMPDIR` | Scratch space for the working copies of the zips |

```bash
Rscript TowerTools_ForcingData/flow.api.clm.R
```

The R environment and the downloader are in the parent repository. An annual window needs roughly 32–64 GB of memory.

## License

AGPL-3.0, as upstream (see `LICENSE`).

---

# NCAR-NEON
Repository for developing code related to NCAR-NEON workshop

Fork this repository to your github account

Bring the fork into your local Rstudio (or other) environment.

## To generate monthly forcing files from tower met data
modify and run `/TowerTools_ForcingData/flow.api.clm.R`  This code does the following:
- Collates NEON data from API, 
- Gap-filling with ReddyProc, and 
- Packages output in NCAR CLM netcdf format
- Makes simple plots to highlight missing data fields
