# PsychoPy + VPixx + EyeLink — automated setup

Installs **PsychoPy** plus, depending on the facility, the **VPixx `pypixxlib`** and/or
**SR Research EyeLink `pylink`** Python APIs, then verifies every import.

When you run it, it asks two things: which **Scully facility** you're setting up (this
decides which APIs get installed), then how to install (conda env or Standalone app).

### Facility → which APIs

| # | Facility | EyeLink `pylink` | VPixx `pypixxlib` |
|---|----------|:---:|:---:|
| 1 | MRI Facility | ✅ | — |
| 2 | MEG Lab | ✅ | ✅ |
| 3 | EEG Lab | ✅ | — |
| 4 | Eye Tracking Lab | ✅ | — |
| 5 | TMS Lab | — | — |
| 6 | Testing Room | — | — |

PsychoPy itself installs at every facility.

### Install modes

- **Conda environment** — no admin needed; installs a per-user Miniconda if required.
  Also installs `psychtoolbox` for low-latency audio.
- **Standalone app** — no admin needed; downloads the StandalonePsychoPy installer from PsychoPy's
  GitHub releases, runs it, then wires the relevant APIs into the app's bundled Python.
  (Standalone already ships psychtoolbox.)

## Quick start

From PowerShell, in the folder containing the script:

```powershell
powershell -ExecutionPolicy Bypass -File .\Install-PsychoPy.ps1
```

It prompts for the facility (1-6), the **PsychoPy version** (blank = latest), then the
action (1 = conda, 2 = standalone, 3/4 = uninstall). To skip the prompts:

```powershell
.\Install-PsychoPy.ps1 -Facility meg -PsychopyVersion latest -Mode conda
.\Install-PsychoPy.ps1 -Facility tms -PsychopyVersion 2024.2.4 -Mode standalone
```

The VPixx tarball and the EyeLink folder are auto-detected when needed.

## Prerequisites (Windows)

1. **EyeLink Developers Kit** — needed at MRI/MEG/EEG/Eye Tracking. Provides the pylink
   package under `C:\Program Files (x86)\SR Research\EyeLink\SampleExperiments\Python\64\<pyver>\`.
2. **VPixx Software Tools** — needed at the MEG Lab only. Provides `pypixxlib-<ver>.tar.gz`
   under `C:\Program Files\VPixx Technologies\...`.
3. **Internet access** — for the Miniconda / Standalone download and the pip installs.

At TMS / Testing Rooms neither vendor package is required.

No admin rights are required. The vendor API packages live in `Program Files`
(shared across accounts); the script only wires their Python bindings into the target
interpreter.

## Mode 1 — Conda environment

What it does:

0. If conda isn't present, silently installs a per-user **Miniconda** ("Just Me") to
   `%USERPROFILE%\miniconda3` — no admin prompt, no PATH/registry changes for other users.
1. `conda create -n psychopy python=3.10` (reuses an existing env unless `-Force`).
2. `pip install --upgrade psychopy`.
3. Installs `pypixxlib-<ver>.tar.gz`.
4. Writes a **BOM-free** `pylink.pth` into the env's `site-packages` pointing at the
   EyeLink folder that matches the env's Python version.
5. Installs `psychtoolbox` (skip with `-SkipPsychtoolbox`).
6. Verifies `psychopy`, `pylink`, `pypixxlib`, `psychtoolbox`, then runs
   `conda init powershell` so activation works.

When it finishes, the easiest way to start PsychoPy is the **Desktop shortcut** the script
creates ("PsychoPy (psychopy env)") — it launches the app with no shell or activation
needed. Otherwise, from the **Anaconda PowerShell Prompt** (Start menu) or a new PowerShell
window:

```powershell
conda activate psychopy
psychopy            # Coder/Builder GUI
```

## Mode 2 — Standalone app

What it does:

1. Queries PsychoPy's GitHub releases for the latest `StandalonePsychoPy-…-win64-<pyver>.exe`
   and downloads it.
2. Launches the installer **interactively** so you choose the install location.
3. Locates the app's bundled Python, installs `pypixxlib`, writes the `pylink.pth`, and
   verifies imports (psychtoolbox is already bundled).

After it finishes, launch PsychoPy from the Start Menu — pylink and pypixxlib are
available inside Builder/Coder.

## Uninstalling

The menu also offers removal (or pass the mode directly):

- **`[3]` / `-Mode uninstall-conda`** — removes the `psychopy` conda env (`conda env remove`,
  plus any leftover env folder and the Desktop shortcut). Miniconda itself is left in place.
- **`[4]` / `-Mode uninstall-standalone`** — finds PsychoPy in the Windows uninstall registry
  and runs its uninstaller. If no registry entry is found, it offers to delete the install
  folder instead.

Both prompt for confirmation first; add `-Force` to skip the prompt.


## Options

| Flag | Purpose |
|------|---------|
| `-Mode <conda\|standalone\|uninstall-conda\|uninstall-standalone>` | Skip the interactive prompt. |
| `-Facility <mri\|meg\|eeg\|eyetracking\|tms\|testing>` | Set the facility (decides the API set); skips the 1-6 prompt. |
| `-PsychopyVersion <ver>` | PsychoPy version (e.g. `2024.2.4`) or `latest`; skips the version prompt. |
| `-EnvName <name>` | Conda env name (default `psychopy`). |
| `-PyVersion <x.y>` | Python version for the env / the Standalone build to pick (default `3.10`). |
| `-MinicondaPath <path>` | Where to install/find the per-user Miniconda (default `%USERPROFILE%\miniconda3`). |
| `-StandaloneDir <path>` | Existing PsychoPy install folder (standalone mode). |
| `-UseExistingConda` | Use a conda already on PATH instead of installing Miniconda. |
| `-VpixxTarball <path>` | Explicit path to `pypixxlib-*.tar.gz` if auto-detect misses. |
| `-PylinkDir <path>` | Explicit path to `...\Python\64\<ver>` (the folder containing `pylink`). |
| `-SkipPsychtoolbox` | Conda mode: don't install psychtoolbox. |
| `-Force` | Conda: recreate the env. Standalone: re-download/reinstall even if one is found. |

Example with explicit paths (conda mode):

```powershell
.\Install-PsychoPy.ps1 -Mode conda `
  -VpixxTarball "C:\Program Files\VPixx Technologies\Software Tools\pypixxlib\pypixxlib-1.11.3.tar.gz" `
  -PylinkDir "C:\Program Files (x86)\SR Research\EyeLink\SampleExperiments\Python\64\3.10" `
  -Force
```

