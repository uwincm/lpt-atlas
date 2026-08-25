# LPT Atlas Plotting (lpt-atlas)

Create a static plot atlas of everything using all the LPT data in an LPT directory.

This repository contains the plotting and data-reading code used to generate an **atlas of plots for LPT (MJO Lifecycle / Lagrangian Precipitation Tracking) analyses**.

## Example Plots

Coming soon.


## The basic philosophy

> **Point the atlas plotting system at an LPT analysis directory, run one command, and let it generate/update everything that can be generated from the data currently available.**

The plotting system is designed to be **data-aware**. It does not require every possible LPT product to be present. For example, if centroid data are available but spatio-temporal masks have not yet been generated, centroid-based plots will be produced while mask-dependent plots are skipped.

The directory organization intentionally follows the structure of the main LPT repository.

---

## Directory Structure

A typical installation looks like this:

```text
lpt-atlas/
│
├── README.md
├── pixi.toml                  # Pixi project/environment definition
├── pixi.lock                  # Locked dependency versions
│
├── lpt-atlas/                 # Python package: plotting and data-reading code
│   ├── __init__.py
│   ├── io.py                  # Reading LPT data
│   ├── centroid.py            # Centroid-related plotting
│   ├── mask.py                # Spatio-temporal mask plotting
│   ├── tracks.py              # Track/lifecycle plotting
│   ├── ...
│   └── ...
│
└── MASTER_PLOT/               # Template for an atlas plotting run
    ├── lpt-atlas-plot.py      # Main driver script
    ├── config.py              # User configuration, if needed
    └── ...
```

The important distinction is:

```text
Git repository
│
├── pixi.toml           <-- ENVIRONMENT
├── pixi.lock           <-- LOCKED ENVIRONMENT
│
├── lpt-atlas/          <-- CODE
│
└── MASTER_PLOT/        <-- TEMPLATE
       │
       │ copy
       ▼
   my-atlas-run/
       │
       └── lpt-atlas-plot.py  <-- EDIT AND RUN THIS
```

The Pixi environment files remain in the **main repository directory**. They should not be copied into individual `MASTER_PLOT` derivatives.

---

# Setting Up the Pixi Environment

This project uses [Pixi](https://pixi.sh/) to manage its software environment and dependencies.

After cloning the repository:

```bash
git clone <repository-url>
cd lpt-atlas
```

initialize/create the environment with:

```bash
pixi install
```

This uses:

```text
pixi.toml
pixi.lock
```

to create the reproducible environment.

The environment is associated with the repository root. You can run commands from anywhere within the project using `pixi run`.

For example:

```bash
pixi run python --version
```

or:

```bash
pixi run python MASTER_PLOT/lpt-atlas-plot.py
```

It is generally preferable to use `pixi run` rather than manually activating the environment.

---

## Pixi Configuration

The repository's `pixi.toml` should contain the project's dependencies and configuration.

Conceptually:

```toml
[workspace]
name = "lpt-atlas"
channels = ["conda-forge"]
platforms = ["linux-64", "osx-64", "osx-arm64"]

[dependencies]
python = ">=3.11"
numpy = "*"
xarray = "*"
matplotlib = "*"
...

[pypi-dependencies]
lpt-atlas = { path = ".", editable = true }
```

The exact dependencies should be added as the project develops.

After changing dependencies, update the environment and lock file with:

```bash
pixi install
```

The resulting `pixi.lock` should be committed to Git so that the environment can be reproduced.

### Running Python

Instead of:

```bash
python lpt-atlas-plot.py
```

use:

```bash
pixi run python lpt-atlas-plot.py
```

when working from the appropriate directory.

Alternatively, define Pixi tasks for commonly used operations. For example:

```toml
[tasks]
atlas = "python MASTER_PLOT/lpt-atlas-plot.py"
```

Then the atlas can be run from the repository root with:

```bash
pixi run atlas
```

The task can eventually be expanded to accept configuration arguments if desired.

---

# Relationship to the LPT Repository

This organization intentionally follows the structure of the main LPT repository:

```text
LPT repository                     LPT Atlas repository
──────────────                     ───────────────────

pixi.toml                          pixi.toml
pixi.lock                          pixi.lock

MASTER_RUN/                        MASTER_PLOT/
    │                                  │
    └── run scripts                    └── lpt-atlas-plot.py

lpt/                               lpt-atlas/
    │                                  │
    ├── algorithms                     ├── data readers
    ├── tracking                        ├── plotting functions
    ├── utilities                       ├── plotting utilities
    └── ...                             └── ...
```

The environment files belong to the **repository**, not to an individual run.

In both repositories:

* `pixi.toml` defines the software environment.
* `pixi.lock` records the reproducible dependency versions.
* `MASTER_*` provides a template for a particular run.
* The user makes a copy of the template.
* The copied directory contains user-specific configuration.
* The underlying code remains in the version-controlled package directory.

---

# Creating an Atlas Run

Make a copy of `MASTER_PLOT`:

```bash
cp -r MASTER_PLOT my-atlas-run
```

For example:

```bash
cp -r MASTER_PLOT MJO_ERA5_1980-2025
```

You are now free to modify:

```text
MJO_ERA5_1980-2025/lpt-atlas-plot.py
```

without modifying the template in `MASTER_PLOT/`.

The Pixi environment does **not** need to be copied.

For example:

```text
lpt-atlas/
│
├── pixi.toml
├── pixi.lock
│
├── lpt-atlas/
│   └── ...
│
├── MASTER_PLOT/
│   └── lpt-atlas-plot.py
│
└── MJO_ERA5_1980-2025/
    └── lpt-atlas-plot.py
```

Both `MASTER_PLOT/lpt-atlas-plot.py` and `MJO_ERA5_1980-2025/lpt-atlas-plot.py` use the same Pixi environment and the same `lpt-atlas` package.

---

# Running the Atlas

The main driver is:

```text
lpt-atlas-plot.py
```

The driver should be the **only script that normally needs to be executed by the user**.

For example, if the driver accepts the LPT directory as an argument:

```bash
pixi run python MJO_ERA5_1980-2025/lpt-atlas-plot.py \
    /data/LPT/MJO_ERA5_1980-2025
```

Alternatively, if the LPT directory is specified in the run configuration:

```bash
cd MJO_ERA5_1980-2025
pixi run python lpt-atlas-plot.py
```

The driver determines which LPT products are available and calls the appropriate plotting routines.

The goal is that the user does **not** need to manually determine which plotting functions can be run.

---

# Data-Aware Plot Generation

The atlas is designed to operate on **partially complete LPT datasets**.

For example, an LPT analysis directory might contain:

```text
LPT_OUTPUT/
├── centroid/
│   └── ...
│
├── tracks/
│   └── ...
│
└── ...
```

but not yet contain spatio-temporal masks.

The atlas driver should recognize this and generate the plots that are currently possible:

```text
Available data
      │
      ├── Centroids ───────────────► Generate centroid plots
      │
      ├── Tracks ──────────────────► Generate track plots
      │
      ├── Spatio-temporal masks ───► Not available
      │                                  │
      │                                  └── Skip mask plots
      │
      └── Other products ──────────► Generate if available
```

The absence of a particular product should therefore **not cause the entire atlas generation to fail**.

Instead, the driver should:

1. Inspect the supplied LPT directory.
2. Determine which data products are available.
3. Run all applicable plotting routines.
4. Skip plots requiring unavailable data.
5. Continue to the next plot category.
6. Report what was generated and what was skipped.

For example:

```text
LPT Atlas Plotting
==================

Input directory:
    /data/MJO/LPT_OUTPUT

Available products:
    [x] Centroid data
    [x] Track data
    [ ] Spatio-temporal masks
    [x] Precipitation data

Generating:
    [x] Centroid overview
    [x] Track overview
    [x] Precipitation statistics

Skipping:
    [-] Spatio-temporal mask atlas
        Reason: mask data not available

Atlas generation complete.
```

This behavior is important because LPT analyses may be generated incrementally. The same atlas command can be rerun as additional products become available.

---

# Recommended Code Organization

The driver script should primarily handle **workflow and configuration**, rather than contain plotting implementation.

For example:

```text
lpt-atlas-plot.py
       │
       ├── locate/read configuration
       │
       ├── inspect LPT directory
       │
       ├── determine available products
       │
       ├── call plotting functions
       │
       └── report results
```

The actual implementation should live in the `lpt-atlas/` package:

```text
lpt-atlas/
│
├── io.py
│      └── functions for reading LPT data
│
├── centroid.py
│      └── centroid-related plots
│
├── tracks.py
│      └── track/lifecycle plots
│
├── mask.py
│      └── spatio-temporal mask plots
│
└── ...
```

This separation keeps the driver relatively simple and makes the plotting routines reusable from other Python code.

---

# Example Workflow

Suppose an LPT analysis is being generated in:

```text
/data/LPT/MJO_ERA5_1980-2025/
```

First create a plotting directory:

```bash
cp -r MASTER_PLOT MJO_ERA5_1980-2025-atlas
```

Edit:

```text
MJO_ERA5_1980-2025-atlas/lpt-atlas-plot.py
```

to point to:

```text
/data/LPT/MJO_ERA5_1980-2025/
```

Then run:

```bash
pixi run python MJO_ERA5_1980-2025-atlas/lpt-atlas-plot.py
```

The script examines the LPT output and generates whatever plots are currently possible.

Later, after additional LPT processing has produced spatio-temporal masks, simply run the same command again:

```bash
pixi run python MJO_ERA5_1980-2025-atlas/lpt-atlas-plot.py
```

The newly available plots will then be generated.

---

# Design Goals

### 1. One-command operation

The user should be able to point the system at an LPT analysis and run one command.

### 2. Incremental operation

The atlas should work even when the LPT analysis is incomplete.

### 3. Automatic product detection

The driver should determine which plotting products are possible based on the files/data available.

### 4. Reproducible environments

`pixi.toml` and `pixi.lock` define a consistent software environment for all atlas runs.

### 5. Reproducible configurations

Each atlas run should have its own copy of `MASTER_PLOT`, preserving the configuration used to generate that atlas.

### 6. Separation of code and configuration

Plotting algorithms belong in `lpt-atlas/`.

Run-specific configuration belongs in the user's copy of `MASTER_PLOT`.

### 7. Git-safe workflow

Users should be able to freely modify their copied plotting directory without accidentally modifying the repository's master template.

### 8. Reusable plotting functions

The plotting routines should be implemented as Python functions/classes in `lpt-atlas/`, rather than being embedded in the driver script.

---

# In Short

The intended workflow is:

```text
                    Git repository
                          │
          ┌───────────────┼────────────────┐
          │               │                │
      pixi.toml       lpt-atlas/       MASTER_PLOT/
      pixi.lock          code             template
          │               │                │
          │               │          cp -r  │
          │               │                ▼
          │               │          MY_ATLAS_RUN/
          │               │                │
          │               │         edit configuration
          │               │                │
          └───────────────┴────────────────┤
                                           │
                                           ▼
                                  lpt-atlas-plot.py
                                           │
                                           ▼
                                    inspect LPT data
                                           │
                              ┌────────────┼────────────┐
                              ▼            ▼            ▼
                           centroid      tracks       masks
                              │            │            │
                              ▼            ▼            X
                            plots        plots        skipped
                              │            │
                              └────────────┴──────┐
                                                  ▼
                                           Atlas complete
```

The key ideas are:

* **`pixi.toml` and `pixi.lock` live at the repository root.**
* **`lpt-atlas/` contains the reusable Python code.**
* **`MASTER_PLOT/` is a template for creating individual atlas runs.**
* **Users copy `MASTER_PLOT/` rather than modifying it directly.**
* **Individual atlas runs do not contain their own Python environment.**
* **`pixi run` provides the common environment for every atlas run.**
* **`lpt-atlas-plot.py` is the main entry point.**
* **The driver automatically generates everything possible from the LPT products that currently exist.**
