Retrodep
========

This command inspects a Go source tree with vendored packages and attempts to work out the versions of the packages which are vendored, as well as the version of top-level package itself.

It does this by comparing file hashes of the packages with those from the upstream repositories.

If no semantic version tag matches but a commit is found that matches, a pseudo-version is generated.

Installation
------------

```
go get github.com/release-engineering/retrodep
```

Running
-------

```
retrodep: help requested
usage: retrodep [OPTION]... PATH
  -debug
    	show debugging output
  -deps
    	show vendored dependencies (default true)
  -exclude-from exclusions
    	ignore directory entries matching globs in exclusions
  -help
    	print help
  -importpath string
    	top-level import path
  -o string
    	output format, one of: go-template=...
  -only-importpath
    	only show the top-level import path
  -template string
    	go template to use for output with Reference fields (deprecated)
  -x	exit on the first failure
```

In many cases retrodep can work out the import path for the top-level project. In those cases, simply supply the directory name to examine:
```
$ retrodep src
```

If it cannot determine the import path, provide it with -importpath:
```
$ retrodep -importpath github.com/example/name src
```

By default both the top-level project and its vendored dependencies are examined. To ignore vendored dependencies supply -deps=false:
```
$ retrodep -deps=false -importpath github.com/example/name src
```

If there are additional local files not expected to be part of the upstream version they can be excluded:
```
$ cat exclusions
.git
Dockerfile
$ ls -d src/Dockerfile src/.git
src/Dockerfile
src/.git
$ retrodep -exclude-from=exclusions src
```

Exit code
---------

| Exit code | Reason                                           |
| ---------:|:------------------------------------------------ |
| 0         | all versions were found                          |
| 1         | any error was encountered other than those below |
| 2         | a version was missing                            |
| 3         | import path needed but not supplied              |
| 4         | no Go source code was found at the provided path |

Example output
--------------

```
$ retrodep $GOPATH/src/github.com/docker/distribution
github.com/docker/distribution:v2.7.1
github.com/docker/distribution:v2.7.1/github.com/Azure/azure-sdk-for-go:v16.2.1
github.com/docker/distribution:v2.7.1/github.com/Azure/go-autorest:v10.8.1
github.com/docker/distribution:v2.7.1/github.com/Shopify/logrus-bugsnag:v0.0.0-0.20171204154709-577dee27f20d
github.com/docker/distribution:v2.7.1/github.com/aws/aws-sdk-go:v1.15.11
github.com/docker/distribution:v2.7.1/github.com/beorn7/perks:v0.0.0-0.20160804124726-4c0e84591b9a
github.com/docker/distribution:v2.7.1/github.com/bshuster-repo/logrus-logstash-hook:0.4
github.com/docker/distribution:v2.7.1/github.com/bugsnag/bugsnag-go:v1.0.3-0.20150204195350-f36a9b3a9e01
...
```

In this example,

* github.com/docker/distribution is the top-level package, and the upstream semantic version tag v2.7.1 matches
* github.com/Azure/azure-sdk-for-go etc are vendored dependencies of distribution
* github.com/Azure/azure-sdk-for-go, github.com/Azure/go-autorest, github.com/aws/awk-sdk-go, and github.com/bshuster-repo/logrus-logstash-hook all had matches with upstream semantic version tags
* github.com/bugsnag/bugsnag-go matched a commit from which tag v1.0.2 was reachable (note: v1.0.2, not v1.0.3 -- see below)
* github.com/beorn7/perks matched a commit from which there were no reachable semantic version tags

Pseudo-versions
---------------

The pseudo-versions generated by this tool are:

* v0.0.0-0.yyyyddmmhhmmss-abcdefabcdef (commit with no relative tag)
* vX.Y.Z-pre.0.yyyyddmmhhmmss-abcdefabcdef (commit after semver vX.Y.Z-pre)
* vX.Y.(Z+1)-0.yyyyddmmhhmmss-abcdefabcdef (commit after semver vX.Y.Z)
* tag-1.yyyyddmmhhmmss-abcdefabcdef (commit after tag)

Limitations
-----------

The vendor directory is assumed to be complete.

Original source code is assumed to be available.

Only git and repositories are currently supported, and working 'git' and 'hg' executables are assumed to be available.

Non-Go code is not considered, e.g. binary-only packages, or CGo.

Commits with additional files (e.g. \*\_linux.go) are identified as matching when they should not.

Packages vendored from forks will not have matching commits.

Files marked as "export-subst" in .gitattributes files in the vendored copy are ignored.
