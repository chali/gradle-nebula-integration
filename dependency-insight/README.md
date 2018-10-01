# Dependency insight investigations 
###### Using core gradle features

## Purpose
Investigating the interactions with dependency selection processes to make sure that all participants, all reasons, and all rejected versions are shown. 

## Selection reasons
The following selection reasons are combined to ensure all selection reasons are shown. 

- Basic version selection reasons
    - Static configuration
    - Dynamic configuration
    - A recommendation (BOM or properties file)
- Enforcement selection reasons
    - Locks
    - Forces
- Selection rules
    - Alignments 
    - Excludes
    - Rejections
    - Replacements
    - Substitutions

## Results

Results are located in `docs/<grouping>/<scenario>` with 
- an `output.txt` file showing the tasks run, the console output, and assertions we're validating against.
- a folder `input` with the contributing `build.gradle` and other related files, such as generated locks and a local bom 

These results are directly written from test results, so they will regenerate when tests are run with `./gradlew clean build`

#### Force messages with recommendations
Using a bom 
- Forces do not show up when there is a recommendation in place
- ★ `rec-force` is the simplest example. See also:
    - `rec-force-lock`
    - `reject-rec-force`
    - `reject-rec-force-lock`
    - `replacement-rec-force`
    - `replacement-rec-force-lock`
    - `substitute-rec-force`
    - `substitute-rec-force-lock`
- Related open issues:
    - [gradle-nebula-integration issue #3](https://github.com/nebula-plugins/gradle-nebula-integration/issues/3): Dependency insight: "Force" selection reason does not show up when there is a recommendation in place

#### Force messages with substitutions
Using `substitute module('<module>') because (substitutionMessage) with module('<other-module>')`
- Forces do not show up when there is a substitution in place
- ★ `substitute-static-force` is the simplest example. See also:
    - `substitute-dynamic-force`
    - `substitute-dynamic-force-lock`
    - `substitute-static-force-lock`
    - `substitute-rec-force`
    - `substitute-rec-force-lock`
- Related open issues:
    - [gradle-nebula-integration issue #4](https://github.com/nebula-plugins/gradle-nebula-integration/issues/4): Dependency insight: "Force" selection reason does not show up when there is a substitution in place

#### Rejection messages with non-dynamic dependencies
Using `selection.reject(rejectionMessage)`
- Rejections do not show up as a contributor when user has not selected a dynamic version
    - ★ `reject-static` is the simplest example. See also the other failing tests starting with `reject-`
- Related open issues:
    - [gradle-nebula-integration issue #5](https://github.com/nebula-plugins/gradle-nebula-integration/issues/5): Dependency insight: Rejections do not show up as a contributor when user has not selected a dynamic version

#### Exclude messages
Using `exclude group: '<group>', module: '<name>'`
- Given I have an exclusion, then I would like to see this as a selection reason
- When running `dependencyInsight`, the exclusion selection information does not show up. Would prefer to see something like:
 ```
By constraint : dependency was excluded beacause <reason>
 ```
 whereas we currently see: 
```
No dependencies matching given input were found in configuration ':compileClasspath'
```
- Other information contributing to selection reasons also does not show up.
    - Forces, locks, and more selection reasons do not show up 
    - More importantly, `exclude-substitute-static` (and related projects) would be a great place to see that substitutions contributed, but did not take priority to the resolution
- Related open issues:
    - [gradle-nebula-integration issue #6](https://github.com/nebula-plugins/gradle-nebula-integration/issues/6): Dependency insight: exclusions do not have any selection reasons

#### Alignment
Using a `ComponentMetadataRule` and `ComponentMetadataDetails.belongsTo(...)`
- Given I have two dependencies in the same platform, and I have multiple forces
    - Related open issues:
        - [gradle-nebula-integration issue #2](https://github.com/nebula-plugins/gradle-nebula-integration/issues/2): Aligned group through belongsTo needs to be easily downgraded to a specific version
    
- Given I have a force on `a:a` and `a:b`, then I expect the build to fail. 
    - ★ `alignment-static-both-force` is the simplest example.
        
#### Second order contributors insight
- Given I have `dependencyA` that is brought in by `dependencyB` that was already conflict resolved
    - then I would like to see an indication that `dependencyB` has been conflict resolved rather than looking like a particular version of `dependencyB` had been simply requested
    -  ★ `second-order-contributer` is the example
    - Related open issues:
        - [gradle-nebula-integration issue #9](https://github.com/nebula-plugins/gradle-nebula-integration/issues/9): Dependency insight: second order contributors are missing information