# Prototype repo to build demos for Nessie

[![Main CI](https://github.com/snazy/nessie-demos/actions/workflows/main.yml/badge.svg)](https://github.com/snazy/nessie-demos/actions/workflows/main.yml)

This git repo is a temporary repo, which will eventually be retired and maybe deleted. The results
of this work will either go into the [Project Nessie repo](https://github.com/projectnessie/nessie/)
or a separate repo under [Project Nessie](https://github.com/projectnessie/).

## Scope

Build a framework for demos that run on Jupyter notebook environments like 
[Google Colaboratory](https://colab.research.google.com/) and
[Binder](https://mybinder.org) and locally (for development purposes),
and on tutorial/course/learning platforms like [Katacoda](https://katacoda.com).

As long as it is feasible and doesn't delay the work
for demos, have a base for production-like performance testing, which has the same or at least
similar restrictions/requirements as the demos.

Every demo must work, we want to eliminate manual testing, so all demos must be unit-testable.

## Directory structure

| directory | purpose |
| --------- | ------- |
| `colab/` | Contains unit-testable Jupyter notebooks that can run in Google Colaboratory and Binder.
| `pydemolib/` | Python library that helps to eliminate nearly all boilerplate code in Jupyter notebook based demos.
| `configs/` | Configuration files that define Nessie, Iceberg, Spark versions and dependencies, supporting demos using different product versions.
| `datasets/` | Various datasets that can be used by the demos. The nessiedemo Python library in `pydemolib/` supports downloading and accessing these datasets.
| `/` | Also contains `apt.txt` + `requirements.txt` used by Binder. This `requirements.txt` has nothing to do with the `requirements.txt` in `pydemolib/`.

## Jupyter notebook environments

### Google Colaboratory

[Google Colaboratory](https://colab.research.google.com/) provides a Jupyter environment.
Users basically just provide a URL to a `.ipynb` file, which is then loaded. The runtime
environment is a container/image that is pre-defined by Google. Changing the base operating
system and such is not possible. Bumping the pre-installed Python dependencies is not feasible.

### Binder

[Binder](https://mybinder.org) is a more customizable platform for Jupyter notebooks and
more (see their website). Binder generates a Dockerfile + image based on the settings in the
source GitHub repository (other sources are possible). It is possible to pre-install both
e.g. Ubuntu and/or Python packages into the Docker image generated by Binder.

Of course, Binder just lets a user "simply start" a notebook via a simple "click on a link".

### Assets needed by notebooks

Nessie demo notebooks need assets like for example the "Nessie Quarkus Runner" binary, example
data sets and Apache Spark tarballs. Since we cannot assume that every Jupyter notebook environment
has access to the source tree on the "local dist", the `nessiedemo` Python library takes care
of downloading such assets, if necessary. 

In other words: it would be wrong to assume that a notebook has direct access to a file in
this Git repository.

## Version restrictions

Nessie and for example Apache Iceberg need to be compatible to each other and must be based
on released versions. This means, that for example Iceberg 0.11.1 declares
[Nessie 0.3.0](https://github.com/apache/iceberg/blob/apache-iceberg-0.11.1/versions.props#L21),
but Nessie 0.4.0 and 0.5.1 work as well.

Apache Spark 3.0 is supported in Iceberg 0.11, Spark 3.1 is only supported since Iceberg 0.12. 

It's very likely that we have to change the Nessie code base to adjust for example the pynessie
dependencies, because of dependency issues in the hosted runtime of Colaboratory notebooks.

## Version flexibility

Different demos probably require different dependencies, like different versions of Apache Spark,
of Apache Iceberg, of Nessie, etc. This is why the set of dependencies in
[`requirements.txt`](pydemolib/requirements.txt) is pretty short and quite relaxed.

Config files in the `configs/` directory contain various sets of dependencies. The `pydemolib`
code takes care or installing the Python dependencies, Apache Spark and the nessie-runner.

It shall also be possible to run (or at least test) the demos against non-released versions
of Nessie and Iceberg, if that's possible "without a ton of code and complexity".

## Datasets for demos

Since some notebooks environments like Google Colaboratory only support uploading a single `.ipynb`
file, which cannot access "files next to it", the `pydemolib` code allows downloading
named datasets to the local file system. Files in a dataset can then be identified by name via a
Python `dict`.

## Build

Since demos must be unit-testable, we will need some build tool and GitHub workflows.

## Bumping versions of Nessie and Iceberg et al

Bumping versions should eventually work via a pull-request. I.e. a human changes the versions
used by the demos, submits a PR, CI runs against that PR, review, merge.

At least in the beginning, Nessie will evolve a lot, and we should expect a lot of changes
required to the demos for each version bump. For example, the current versions require a
"huge context switch" in the Demos: jumping from running Python code to executing binaries
and/or running SQL vs. executing binaries to perform tasks against Nessie.

It feels nicer to ensure that everything in the demos repo works against the "latest & greatest"
versions.

If someone wants to run demos against an older "set of versions":
* In Google Colaboratory it's as easy as opening a different URL.
* In Katakoda there seems to be no way to "just use a different URL/branch/tag".

We might either accept that certain environments just don't support "changing versions on the fly"
or we use a different strategy, if that's necessary. So the options are probably:
* Demos (in Katakoda) only work against the "latest set of versions"
* "Archive" certain, relevant demos in the demo-repo's "main" branch in separate directories

I suspect, this chapter requires some more "brain cycles".

### Preparing version bumps

It would be nice to prepare PRs for Nessie and Iceberg version bumps before those are actually
released.

Maybe we can prepare this convenient and "nice to have" ability.

It might also help to have this ability to run demos (and production-like perf tests) as a
"pre-release quality gate" to ensure that user-facing stuff works and there are no performance
regressions.

## Git history

The history in Git should work the same way as for [Project Nessie repo](https://github.com/projectnessie/nessie/),
i.e. a linear git history (no merges, no branches, PR-squash-and-merge).

## Compatibility Matrix

| Current | Nessie | Apache Iceberg | Apache Spark | Notes
| ------- | ------ | -------------- | ------------ | -----
| No |0.4.0 | 0.11.1 | 3.0 | Iceberg declares Nessie 0.3.0, but there are no (REST)API changes between Nessie 0.3.0 and 0.4.0
| **Yes** |0.5.1 | 0.11.1 | 3.0 | Iceberg declares Nessie 0.3.0, but there are no _breaking_ (REST)API changes between Nessie 0.4.0 and 0.5.1
| No |0.5.1 | (recent HEAD of `master` branch) | 3.1 |
| No |(recent HEAD of `main` branch) | n/a | n/a | **incompatible**, would require a build from a developer's branch
