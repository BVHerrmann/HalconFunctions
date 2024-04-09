# HALCON Jobs

> [!IMPORTANT]
> Most Jobs in this repository rely on Preocedures from [hhdev-procedures](https://github.com/bertram-eu/hhdev-procedures). These procedures are automatically made available by BDevEngine.

## TODO
- [ ] Add template file for project descriptions
- [x] Specify how images relating to a job should be stored (Git LFS)

## Jobs

> [!NOTE]
> This repository only permits `.hdev`, `.md` and  `license_*.dat` files. Everything else is ignored by default.
> Further exceptions: `.tif, .tiff, .png`.

All Programs should also contain separate procedures intended for testing. Testing must *not* require to modify the procedures used in production and be started directly from `main`.

### Common Jobs

Common Procedures are stored as `HDevelop Program File (.hdev)` in a folder structure resembling the Chapter in `General Documentation`. These programs are designed to be re-used as is.

### Project Specifc Jobs

Programs belonging to a specific Project are located in the directory `Projects/P#####/`. These folders should contain a `README.md` with a project description. `Projects/Template/` contains a Template to be used for all new projects.

### Images

The images used for development should be stored in the `images` folder next to the job. 

### Temporary files

The folder is for artifacts that can be generated over and over again when running jobs.
Files that are stored in the `tmp` folder are ignored. 

## General Requirements

### Coding Style

The Coding Style should match the HALCON standard as closely as possible.

* Procedures and Files: `snake_case`
* Parameters and Variables: `CamelCase`

### Usage of External Procedures

> [!IMPORTANT]
> External Procedures should never be included locally. All procedures from [hhdev-procedures](https://github.com/bertram-eu/hhdev-procedures) are made available automatically.