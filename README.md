# TryCI

A script to emulate a soft-dev buildbot CI job run.

If a build job fails, `TryCI` can conduct a post-mortem (using the
`--post-mortem` flag) by attaching a shell which allows you to prod around
inside the image.

This must be run from within a softdev repo which uses buildbot for CI, in the
same directory as the repo's `.buildbot.sh` file.

## Example usage

The example below shows how a CI build for Alloy (located on my local machine)
can be run on remotely on `bencher16@soft-dev.org` (with the `bencher16` alias
in my `~/.ssh/config`): 

```sh
cd /path/to/alloy
tryci --remote bencher16 --post-mortem
```

The `--remote` arg specifies the ssh host where the build should be run. 

The `--post-mortem` flag attaches a shell to the build if it fails so that you
can prod around inside the container.

The docker image and container for the build are removed before the script exits.

## Checking out a specific CI job

By default, `TryCI` uses the working tree as the build context for the CI job.
This works well for iterating on features, but can be annoying when you want to
debug a remote CI job which failed.

You can specify which version of the repo you want to `TryCI` with the
`-c/--checkout <ref>` option. `<ref>` can be either a branch, commit, or tag of
the local repository; or a remote URL of the git repository. Here are some
examples:

```sh
# run tryci on the local HEAD
tryci -c HEAD

# run tryci on the local feature-branch
tryci -c feature-branch

# run tryci on the master branch of jacob-hughes/yk
tryci -c https://github.com/jacob-hughes/yk

# run tryci on the 'trying' branch of ykjit/yk
tryci -c https://github.com/ykjit/yk#trying

# run tryci on a specified commit ykjit/yk
tryci -c https://github.com/ykjit/yk#0a6902a
```

This works with the `--post-mortem` flag, so -- provided docker is installed --
you can even use `TryCI` to prod around in a CI job on machines which you
haven't cloned the original repo or setup to develop on.

## Using bindmounts for faster build times

Sometimes a CI build will depend on a submodule or external program which the
`.buildbot.sh` script builds from source each time. For large projects (like
`ykllvm`, which can take up to an hour to build) this can make build times
significantly slower.

Instead, you can mount arbitrarily many prebuilt programs from the host to be
used inside the docker container using `-m/--mount` flag. You can specify
multiple bind-mounts by repeating the option or by providing a comma-separated
list:

```
-m /src:/dst -m /foo:/bar:ro
-m /src:/dst,/foo:/bar:ro
```
If using this with the `--remote` flag, you must ensure the directory you wish
to bind-mount exists on the remote machine -- it does not `scp` / `rsync` it
over.

## Troubleshooting

* Docker must be installed on both the local machine and remote machine used to
  compile and run the build.
* Your user account on both the local and remote machine must be a member of the
  `docker` group.
* If running CI on a remote machine (`--remote <server_name>`) you must use ssh
  passwordless login (using public/private key pair).


