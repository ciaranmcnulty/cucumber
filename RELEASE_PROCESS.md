# Release Process

The release process is supported by scripts and bash functions in the `scripts` directory.
To follow the instructions below you first have to load the bash scripts into your
current shell:

    source ./scripts/functions.sh

## Library groups

Some libraries are implemented in several programming languages.
Examples are _Gherkin_, _Cucumber Expressions_ and _Tag Expressions_.

In the instructions below, `group_path` refers to the main directory
containing several implementations, for example `gherkin`,
`cucumber-expressions` or `tag-expressions`.

If a library has multiple implementations, they should always be released at the
same time, even if only one of them has been modified. The reason for this is
that they all follow the same versioning scheme ([semver](http://semver.org/)).

## Syncing files with rsync

Some of the subrepos need to have a copy of the same file. Examples of this is
`LICENSE` and expected results for approval tests such as `gherkin/testdata/*`.

To simplify the maintenance of these duplicated files we use `rsync` to copy a
master to the subrepos. What files to copy are listed in various `.rsync` files,
and files are synchronised by doing:

    rm -f .rsynced
    make .rsynced

## Releasing a library

First of all, make sure all files are rsynced (see above).

Then, make sure you have the right release karma:

    release_karma_all ${group_path}

If there is a Java library to release, run `mvn javadoc:javadoc` in the subrepo 
folder to make sure JavaDoc is good. It sometimes fails if there are bad tags.

Update `CHANGELOG.md` of the library in the following places:
* Change `Unreleased` to `[X.Y.Z] - yyyy-MM-DD`
* Remove any Removed/Added/Changed/Fixed sections without entries
* Add a new empty `Unreleased` section at the top of the page
* Update the `Unreleased` link at the bottom of the page
* Add a new `[X.Y.Z]` link at the bottom of the page

Then commit:

    git commit -m "library-name: Prepare for release vX.Y.Z"

Make sure all subrepos build successfully:

    build_subrepos ${group_path}

Make sure anything modified in the monorepo is pushed to subrepos:

    push_subrepos ${group_path}

Wait for the CI of each subrepo to build successfully.
Release all the subrepos in the group:

Release each language independently:

    release_subrepo ${subrepo_path} ${version} ${next_version}

Occasionally, Java releases will fail. In that case, redeploy is possible:

    cd ${subrepo_path}/.release
    git checkout v${version}
    mvn --batch-mode release:perform -Psign-source-javadoc -DskipTests=true

JavaScript releases may ask you for a new version. Just hit enter (or enter)
the version of the release you're making now (not the post-release version!)

After the release is done, you'll have a dirty working copy with minor changes
updating to the version you released. Commit these changes:

    git add .
    git commit -m "library-name: Release vX.Y.Z"

Tag the monorepo:

    # group_name is typically the same as group_path, e.g. "cucumber-expressions"
    git tag "${group_name}-v${version}"
    git push && git push --tags

Pushing the tags to the monorepo will push the same tags to the subrepos.
