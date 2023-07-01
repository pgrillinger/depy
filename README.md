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

