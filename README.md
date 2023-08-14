# respin
VHDL source code packaging and dependency framework

## Purpose
This tool aims to provide a framework for dependency managemnt and package of of mainly VHDL code.
The primary goals that should be achieved are:
- Package is described by a configuration file (typicaly in root folder of the source package)
- Information about available source packages tracked in a database (remote)
- Sources packages are uniquely identified by name and a semantic version
- Source packages may specify dependencies
- Published packages become irrevocable once a dependency exist
- Package download solves all dependencies recursively
  - redundant depencies get merged into one download
- Package upload checks that all dependencies are available
- Multiple options where packages are stored (GIT, Artifactory, etc)
- Native support for VHDL
  - ability to map dependencies into VHDL libraries (library names based on pacakge name and version)
  - abiltiy to create compilation scripts

Secondary goals are:
- Deeper GIT integration
  - support tag synchronization
  - consistency checking
  - development checkouts
- Signing of packages and/or other means of authenthication

## Motivation
The development is motivated by the lack of available tools for HDL dependency management across multiple repositories.
Ideally, the result will be something similar like npm or yarn but applicable to designs written in VHDL/Verilog and
intended for both simulation and synthesis.

## Implementation
- Language: Python 3.10+
- Database: PostreSQL, but looking and neo4j as a promissing alternative as well
- Configuation format: yaml
- Frontent/client: command-line only

## Distribution
- PyPy package / source code

## Plan & Status
1. Concept phase: definition of use cases, data flows, main functions, data formats, database data model,
2. (not started) data-flow prototype: upload/download with local database and file-based store, dependency resolution
3. (not started) tool support: support for GHDL, evaluation with simple VHDL projects
4. (not started) remote store support: package upload/download from GIT and/or generic URLs
5. (not started) remote database, authenthication: support for remote database with authenthication

## Use Cases
1. Create IP package
   * IP package is defined by a unique name and version
   * Name is a case-insensitive single string comprised of `[\._-a-zA-Z.0-9]`
   * Version schedme as defined in [semver.org](https://semver.org/)
   * Default format for packaging is zip, full name should be `<name>-version.zip`
2. Upload IP package
   * Repository for upload should be defined in a configuration file with command-line override option
   * Overwriting existing packages in repository (same name/version) should not be possible
   * Authenthication via token / key / credentials
   * Meta-information for indexing is expected to be part of the package (see use case 3)
   * Package index will be managed in a database (location/name configurable)
   * Upload and indexing should be atomic if possible (rollback if neededed)
3. Classify/index IP packages
   * Each package must be classified to allow indexing/lookup
   * A single configuration yaml file called `respin.yaml` will be used for each IP package
   * Mandatory configuration items:
     * Package name
     * Package version
     * Author(s) / Maintainer(s)
   * Optional configuration items (extensions allowed)
     * Package status (development, released, obsolete, deprecated)
     * IP dependencies
     * Tool dependencies
     * Compile script
     * Compile library
     * License / copyright
     * Documentation link
     * Issue tracker link
     * VCS link (GIT assumed for this version)
4. Download and extract IP package locally
   * Allow download of specified package / version via command line
   * First lookup is always into database, if found the package location (repository) will be obtained from there
       * Override via command line can be considered
   * Allow automated extraction of download packages into a local cache
   * Manage multiple versions of the same package inside the cache
5. Support project dependecies and compilation
   * A project will specify all its dependencies via its `respin.yaml` file
   * By running `resping update`, all dependencies will be resolved and loaded into local cache
      * includes checking for new versions if available and prompting for update
      * includes creation of compile scripts for each IP
      * includes library-management to prevent name-space conflicts
6. Support for upstreaming of changes
   * If an IP is modified, regardless of its location in project (part of project or dependency), change upstreaming should be possible
       * Assumes the IP package contains reference to the original VCS
       * Current version will be limited to GIT
   * Option A: All IP packages are directly loaded from GIT if `respin update --devel` option is used
   * Option B: Specific package is cloned via GIT by running `resping update <package> --devel` option is used
   * Switching to the package location in cache is done by `respin cd <package>`
   * Usual GIT commands can then be used to manipulate the cloned package