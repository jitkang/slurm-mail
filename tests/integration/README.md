# Integration Tests

## Introduction

The `tests/integration` directory contains an automated test harness for Slurm-Mail. It makes use of https://github.com/neilmunday/rocky8-slurm to create a container to run the integration tests under.

## Adding new Slurm version support

> **_Note:_** When a new version of Slurm is released https://github.com/neilmunday/rocky8-slurm needs to have been updated to support this new version first.

Update the `../.github/workflows/testing.yml` workflow to include the new Slurm version.

## Adding tests

Slurm tests should be added to `tests.yml`.

A test is defined like so:

```yaml
  testX:
    # commands to run in batch script
    commands: |
      srun hostname  
    # test description
    description: one node test
    # is the job expected to fail?
    job_fail: false
    # sbatch options
    options:
      mail-type: ALL
      time: "01:00"
    # optionally specify any commands to run after job submission
    post_submit: |
      echo "I was run"
    # are errors expected in the slurm-send-mail log?
    send_errors: false
    # are errors expected in the slurm-spool-mail log?
    spool_errors: false
    # how many spool files are expected?
    spool_file_total: 2
```

## Running tests

The `run.sh` script is used to run the test suite against a particular version of Slurm.

The script first compiles the Slurm-Mail RPM for RHEL 8.

Next the script starts a Rocky Linux 8 container with the given version of Slurm and Slurm-Mail installed.

Within the image the `mail-server.py` script is used to create a simple mail server to process e-mails generated by Slurm-Mail.

The `run-tests.py` together with `tests.yml` are copied inside the image.

Once the container is up and running the `run.sh` script executes `run-tests.py` which submits the jobs defined in `tests.yml`. As each job runs the `run-tests.py` script checks if each job was processed as expected by Slurm-Mail.

A summary of how many tests passed and failed is printed at the end of the tests.

Excute all tests against Slurm 22.05.6:

```bash
./run.sh -s 22.05.6
```

Execute a particular test:

```bash
./run.sh -s 22.05.6 -t test2
```

Enable verbose logging:

```bash
./run.sh -s 22.05.6 -v
```

> **_Tip:_** As `run.sh` will build a Slurm-Mail RPM you can use the `-r` flag on subsequent invocations to skip building the RPM and thus save time.

## Testing e-mails

If you want to see how the e-mails will look in an e-mail client you can use the provided Docker compose file to start-up a Slurm-Mail container together with a [MailHog](https://hub.docker.com/r/mailhog/mailhog/) container.

A helper `compose-up.sh` script is provided in this directory for ease of use.

Usage:

```bash
./compose-up.sh -s SLURM_VERSION [-r]
```

If the Slurm-Mail RPM has already been built and exists in this directory you can use `-r` option to skip building the RPM.

```bash
./compose-up.sh -s 22.05.6
```

Once the containers are up and running you can access the [MailHog](https://hub.docker.com/r/mailhog/mailhog/) web GUI at http://localhost:8025

To submit jobs you can run the following command to launch an interactive bash shell:

```bash
docker exec -it slurm-mail /usr/bin/bash -i
```
