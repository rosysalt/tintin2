---
{"dg-publish":true,"permalink":"/9-sound-cloud/services/compare-sca-differ-job/"}
---

[[9_SoundCloud/Services/supply-chain-albums\|supply-chain-albums]]; 

# Metadata
- Links:: 
- Action:: #to_publish
- Keyword:: #tech

# Compare SCA Differ Job

Run the job 2022-10-13
- Given that we have the production run at TIME_SLOT `2022-10-21_06_00` in `/srv/contentingestion/supply-chain-albums`
- We want to run the job with `dev` configuration, and the HADOOP path will be in `/tmp/content-team-staging/supply-chain-albums/dev`
- Let's assume that we test the new logic with  TIME_SLOT `2022-10-21_06_42`. It will be shorten to 1442
    - prod path http://elefs.lb.db.s-cloud.net/srv/contentingestion/supply-chain-albums/2022/10/28/06
    - dev path 00 http://elefs.lb.db.s-cloud.net/tmp/content-team-staging/supply-chain-albums/dev/2022/10/2/06/00
    - dev path 42 http://elefs.lb.db.s-cloud.net/tmp/content-team-staging/supply-chain-albums/dev/2022/10/28/06/42

| TIME_SLOT                                               | production path                                                 | dev path                                                                |
| ------------------------------------------------------- | --------------------------------------------------------------- | ----------------------------------------------------------------------- |
| 2022-10-19_06_00 (original logic / master branch logic) | (PO) /srv/contentingestion/supply-chain-albums/2022/10/19/06/00 | (DO) /tmp/content-team-staging/supply-chain-albums/dev/2022/10/19/06/00 |
| 2022-10-19_06_42 (new logic / PR branch)                |                                                                 | (DN) /tmp/content-team-staging/supply-chain-albums/dev/2022/10/19/06/42 |

**In Hadoop cluster**

```
export PROD_PATH=2022/10/28/06/00
export DEV_PATH_1=2022/10/28/06/00
export DEV_PATH_2=2022/10/28/06/42
```

Copy Snapshot output from production at time 06_00 to dev at time 06_42 so we can run ReleaserJob for the minute 06_42
```
HADOOP_USER_NAME=content-team-staging hadoop distcp /srv/contentingestion/supply-chain-albums/${PROD_PATH}/snapshot /tmp/content-team-staging/supply-chain-albums/dev/${DEV_PATH_2}/snapshot
```

Copy Releaser output from production at time 06_00 to dev at time 06_00 so that we can run the Differ
```
HADOOP_USER_NAME=content-team-staging hadoop distcp /srv/contentingestion/supply-chain-albums/${PROD_PATH}/releaser /tmp/content-team-staging/supply-chain-albums/dev/${DEV_PATH_1}/releaser
```

Add last-successful-run file for the `2022-10-13-06-00`
```
echo '2022-10-28_06_00' > last-successful-run
HADOOP_USER_NAME=content-team-staging hadoop fs -copyFromLocal -f last-successful-run /tmp/content-team-staging/supply-chain-albums/dev/
```

**Deploy staging prometheus**
```
make deploy-staging-prometheus
```

**In `releaser` container**

Build the new `releaser` image from this branch, and run the Releaser Job with `--config=dev` at timeslot 42
```
TIME_SLOT=2022-10-28_06_42 ./releaser --config=dev
```

**In `differ` container**

Build the new `differ` image from this branch. Then generate the empty parquet file for UploaderFailures to dev at time 14_00, to make sure that the DifferJob will only compare the original logic with the new logic. If we have some UploaderFailures, DifferJob will add these failures in the output to retry, and the Differ output will have more information than we expect.
```
export DEV_PATH_1=2022/10/28/06/00
HADOOP_USER_NAME=content-team-staging UPLOADER_FAILURES_PATH="/tmp/content-team-staging/supply-chain-albums/dev/${DEV_PATH_1}/uploader/failures" ./empty-parquet-writer --config=empty-parquet-writer
```

With the `differ` image, run the Differ Job with `--config=dev` at timeslot 42
```
TIME_SLOT=2022-10-28_06_42 ./differ --config=dev
```