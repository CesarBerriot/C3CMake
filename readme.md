# C3CMake
C3 support for CMake 

### Usage

- Fetch this repo
- Use one of the following
  - `C3CMake_import_sources(target sources)` \
    import C3 sources using a list of absolute paths
  - `C3CMake_import_sources_glob(target sources_path)` \
    import C3 sources using a source directory path relative to the `target`'s `SOURCE_DIR`
  - `C3CMake_import_sources_glob_absolute(target sources_path)` \
    import C3 sources using an absolute source directory path