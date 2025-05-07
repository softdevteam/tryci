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

## Using bindmounts for faster build times

Sometimes a CI build will depend on a submodule or external program which the
`.buildbot.sh` script builds from source each time. For large projects (like
`ykllvm`, which can take up to an hour to build) this can make build times
significantly slower.

Instead, you can mount prebuilt programs from the host to be used inside the
docker container using the `TRYCI_BINDMOUNTS` environment variable.
`TRYCI_BINDMOUNTS` takes a comma-separated list of bindmounts. For example:

    `~/research/ykllvm/build/bin:/ci/ykllvm/inst/bin`

This will map `~/research/ykllvm/build/bin` on the host to
`/ci/ykllvm/inst/bin` inside the CI container. Note that in order to prevent
unintended overwriting, such mappings are always mounted as read-only. If your
`.buildbot.sh` tries to write to a bind-mounted directory you will encounter
errors.

If using this with the `--remote` flag, you must ensure the directory you wish
to bind-mount exists on the remote machine -- it does not `scp` / `rsync` it
over.

## Troubleshooting

* Docker must be installed on both the local machine and remote machine used to
  compile and run the build.
* Your user account on both the local and remote machine must be a member of the
  `docker` group.
* If running CI on a remote machine (`--remote server_name`) you must use ssh
  passwordless login (using public/private key pair).


