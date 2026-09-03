BiocPropagate
# BiocPropagate

This package evaluates the Bioconductor R Universe build by Bioconductor
propagation criteria.

In general, there are two types of criteria. At the package-level, gates
evaluate criteria, such as a valid version bump, as opposed to platform criteria,
which check, for example, that a platform passed `R CMD check`. All gates are
checked. By default, it checks the following the propagation criteria

```
> library(BiocPropagate)
> dc <- default_criteria()
> names(dc$gates)
[1] "vignettes"          "package_version"    "no_large_files"    
[4] "no_remotes"         "no_secrets"         "no_merge_conflicts"
> names(dc$platform)
[1] "status"           "unsupported"      "platform_version"
```

where `vignettes` expects a passing `R CMD check` on source, `package_version`
checks for a valid version bump, `status` is the per-platform `R CMD check`,
and `platform_version` checks that the candidate for a platform hasn't been
previously propagated. Any unsupported platforms as defined in the
`DESCRIPTION` result in `FALSE`. Platforms not distributed by Bioconductor
result in `NA`.

The propagation function is `check_propagation(args)`. `args` expects a named
list with the following information:

# Note: This example uses data from the API but `check_propagation()` will be
# run in bioc-checks

```
library(BiocPropagate)

pkg <- jsonlite::fromJSON("https://bioc-release.r-universe.dev/mypackage/json")

dir <- tempfile()
dir.create(dir)
writeLines(
    c(paste("Package:", pkg[["Package"]]), paste("Version:", pkg[["Version"]])),
    file.path(dir, "DESCRIPTION")
)

args <- list(
    package = pkg[["Package"]],	# package name
    universe = "bioc-release",	# r universe name
    jobs = jobs,				# from r-universe
    source_path = dir			# path to source so that we can check the DESCRIPTION file
)

propagation <- check_propagation(args)
print(propagation)
##            job time                 config     r check   artifact propagate
## 1  99139136439  216            bioc-checks 4.6.1 ERROR 9719028444        NA
## 2  99139136414  173      linux-devel-arm64 4.7.0    OK 9719020200        NA
## 3  99139136455  313     linux-devel-x86_64 4.7.0    OK 9719049015        NA
## 4  99139136453  185    linux-release-arm64 4.6.1    OK 9719022539      TRUE
## 5  99139136378  244   linux-release-x86_64 4.6.1    OK 9719034420      TRUE
## 6  99139136444  172     macos-oldrel-arm64 4.5.3    OK 9719019271        NA
## 7  99139136474  434    macos-oldrel-x86_64 4.5.3    OK 9719076534        NA
## 8  99139136404  136    macos-release-arm64 4.6.1    OK 9719012015      TRUE
## 9  99139136480  351   macos-release-x86_64 4.6.1    OK 9719065623      TRUE
## 10 99138641621  245                 source 4.6.1    OK 9718981856        NA
## 11 99139136468  143           wasm-release 4.6.0    OK 9719012984        NA
## 12 99139136557  277    windows-devel-arm64 4.7.0    OK 9719041528        NA
## 13 99139136403  201   windows-devel-x86_64 4.7.0    OK 9719025405        NA
## 14 99139136442  159  windows-oldrel-x86_64 4.5.3    OK 9719017026        NA
## 15 99139136431  294  windows-release-arm64 4.6.1    OK 9719045068      TRUE
## 16 99139136408  178 windows-release-x86_64 4.6.1    OK 9719020631      TRUE

# another example against a recent build of BiocCheck
# note since it has a universal build, it doesn't display arch-specific builds
##            job time                 config     r check   artifact propagate
## 1  99075675783  184            bioc-checks 4.6.1  NOTE 9711984248        NA
## 2  99075675798  398     linux-devel-x86_64 4.7.0  NOTE 9712027909        NA
## 3  99075675750  325   linux-release-x86_64 4.6.1  NOTE 9712012642     FALSE
## 4  99075675789  221     macos-oldrel-arm64 4.5.3  NOTE 9711992204        NA
## 5  99075675759  225    macos-release-arm64 4.6.1  NOTE 9711993015     FALSE
## 6  99075221441  255                 source 4.6.1    OK 9711944533        NA
## 7  99075675752  163           wasm-release 4.6.0    OK 9711979807        NA
## 8  99075675767  253   windows-devel-x86_64 4.7.0  NOTE 9711998583        NA
## 9  99075675792  229  windows-oldrel-x86_64 4.5.3  NOTE 9711993921        NA
## 10 99075675770  242 windows-release-x86_64 4.6.1  NOTE 9711996252     FALSE
```

Additional functionality exists to exempt a package from gates, such as the
restriction for large files. It requires an `exemptions.txt` in the package manifest (from Bioconductor).

```
# manifest/exemptions.txt
Package: package1
Exemptions: no_large_files, no_merge_conflicts

Package: package2
Exemptions: no_merge_conflicts
So it clones the manifest to check for exemptions when check_propagation runs.
```
