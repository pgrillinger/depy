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
- Native support for VHDL / SystemVerilog
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
   * Steps:
     * create respin.yaml with IP name, version (similar to pyproject.toml) in root folder of the design
     * run `resping build`
2. Upload IP package
   * Repository for upload should be defined in a configuration file with command-line override option
   * Overwriting existing packages in repository (same name/version) should not be possible
   * Authenthication via token / key / credentials
   * Meta-information for indexing is expected to be part of the package (see use case 3)
   * Package index will be managed in a database (location/name configurable)
   * Upload and indexing should be atomic if possible (rollback if neededed)
   * Steps:
     * run `respin push`
3. Classify/index IP packages
   * Each package must be classified to allow indexing/lookup
   * A single configuration yaml file called `respin.yaml` will be used for each IP package
   * Mandatory configuration items:
     * Package name
     * Package version
        * <number> | <@git_tag> | <@file_ref> | <@env_variable> | <@git_hash>
     * Author/Owner (email)
   * Optional configuration items (extensions allowed)
     * Package status (development, released, obsolete, deprecated) -> can be changed later (new use case)
     * Quality level
     * Maintainers (email)
     * Type of IP: (FPGA_project, ASIC_project, inetrnal IP, 3rd party IP, VIP)
     * Labels
     * IP dependencies
     * Tool dependencies (Vivado=version, Python=version, GHDL=version, ...)
     * Compile script
     * Compile library
     * Regression test script
     * License / copyright
     * Documentation link
     * Jenkins link
     * Issue tracker link
     * VCS link (GIT assumed for this version)
   * Steps:
     * Store relevant data in `respin.yaml`
     * Build package via `respin build`
     * Upload package via `respin push` -> this step updates the database and checks if the dependencies exists
4. Download and extract IP package locally
   * Allow download of specified package / version via command line
   * First lookup is always into database, if found the package location (repository) will be obtained from there
       * Override via command line can be considered
   * Allow automated extraction of download packages into a local cache
   * Manage multiple versions of the same package inside the cache
   * Steps:
     * run `respin pull <package-specification>`
5. Support project dependecies and compilation
    * Steps:
      * Store project dependencies in `respin.yaml`
      * Run `respin update` to pull all specified dependencies locally and generate compile scripts
        * includes library-management to prevent name-space conflicts
      * Run `respin status` to check if local copy is consistent, prints versions / modification status
      * Run `respin clean` to restore all dependencies to pristine status
6. Support for upstreaming of changes
   * If an IP is modified, regardless of its location in project (part of project or dependency), change upstreaming should be possible
       * Assumes the IP package contains reference to the original VCS
       * Current version will be limited to GIT
   * Steps:
     * Use `respin update --devel` to clone all dependencies from VCS (GIT)
     * Use `resping pull <package-specification> --devel` tp clone a specific package from VCS (GIT)
     * Use `respin path <package>` to determine the path where a package was stored locally
     * Use GIT commands as usual to manipulate the cloned package, `respin build && respin push` to upload the new version

## Concept

### Database Scheme

* Table 'Packages'
  * Purpose:
    * Track all packages (one entry for each package/version combination)
  * Fields:
    * id, primary key
    * name, required
    * version, required
    * author
    * date
    * source
  * Properties
    * unique (name, version)
* Table 'Links'
  * Purpose:
    * Allow each package to have multiple associated links
    * Expected kinds: tracker, wiki, source, archive, repository
  * Fields:
    * name, primary key
    * url, required
    * package, foreign key (Packages, id)
    * kind
* Table 'Dependencies'
  * Purpose:
    * Track depencendies between packages
    * A package listed as dependency cannot be deleted
  * Fields:
    * id, primary key
    * origin, foreign key (Packages, id)
    * dependency, foreign key (Packages, id)
* Table 'Repositories'
  * Purpose:
    * List of repositories that contain packages
    * Unsure if needed: Could be better left to user-side configuration
  * Fields:
    * id, primary key
    * name, required, unique
    * url, required
    * active, required
    * priority, required
