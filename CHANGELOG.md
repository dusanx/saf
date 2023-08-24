# Changelog

## [0.16] - 2023-08-24

- Added optional [local-path] parameter at `saf backup`, to conveniently set directory before running backup, so it can be used without having to `cd` first.
- Improved error output at `saf backup` if backup doesn't finish with success.
- New `saf revisions [-h] [--quiet | --verbose {0,1,2} | --debug] [target-name] path` command to list all the changes in any local folder troughout backup target versions.
- New [Vifm](https://vifm.info/) plugin, to provide full time-machine like restore! Starfield! Warp! In Vifm!

## [0.15] - 2023-08-19

- Initial public fully working version.
