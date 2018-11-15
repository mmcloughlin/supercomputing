# File Systems: Data Movement and Provenance

## Dac-Man: Data Change Management for Scientific Datasets on HPC Systems 

Large scientific databases frequently updated.

Limited or no provenenace.

Scientists just copy datasets. Lose connection to original (which might be
updated).

### Why? What's the problem?

Existing tools to detect change.

* Unix cmp
* git
* Python filecmp

Do not scale. Meant to work with text files. Most scientific datasets are
binary.

Give un-interpretable results.

Users need another layer of abstraction.

### Dac-Man: Data Change Management

Identifies, captures and manages change in large scientific datasets.

* Not version control system.
* Designed to scale on HPC system
* Allows users to efficiently interpret and quantify changes (plugins)

### Components

User facing:

* Change tracker
* Query manager: retrieves change information

Metadata management:

* Indexing
* Caching

### Comparitors

Users can customize how changes are detected.

* FileComparitor:
  - modified / metadata-only / added,deleted
* DataComparitor:
  - if the file was changed
  - launches jobs to detect what the changes actual are
  - here, users can provide their own scripts
  - comes with builtin comparators (eg. image, matrix, HDF, csv)


## Stacker: An Autonomic Data Movement Engine for Extreme-Scale Data Staging-Based In Situ Workflows 
