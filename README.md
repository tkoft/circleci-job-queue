# circleci-job-queue

[![CircleCI Build Status](https://circleci.com/gh/tkoft/circleci-job-queue.svg?style=shield "CircleCI Build Status")](https://circleci.com/gh/tkoft/circleci-job-queue) [![CircleCI Orb Version](https://badges.circleci.com/orbs/tkoft/job-queue.svg)](https://circleci.com/orbs/registry/orb/tkoft/job-queue) [![GitHub License](https://img.shields.io/badge/license-MIT-lightgrey.svg)](https://raw.githubusercontent.com/tkoft/circleci-job-queue/master/LICENSE) [![CircleCI Community](https://img.shields.io/badge/community-CircleCI%20Discuss-343434.svg)](https://discuss.circleci.com/c/ecosystem/orbs)

## Introduction

Forked from <https://github.com/promiseofcake/circleci-workflow-queue>, which was forked from <https://github.com/eddiewebb/circleci-queue>.

Promiseofcake's fork migrated to use CircleCI's v2 API, which has much more complete workflow/job information than v1. In doing so it simplified the orb down a lot so it only blocks on entire workflows.

This fork adds the original functionality to queue by job back in, but using the v2 API which lets us fetch jobs that are both running and scheduled to run in the future. The behavior differs from the original queue orb in that while the original only ensure that jobs do not run in parallel, here we can ensure jobs will queue in order of their creation, and thus run sequentially. This is probably more in line with how most people expect the queue orb to work. We can now block execution of a job/workflow at specific "gates" along a workflow, even doing so multiple times if needed, guaranteeing that all instances of that job run in order.

For example, for a workflow that looks like:
1. Test (wildy variable in runtime)
2. *Block execution, wait for earlier staging deploys to finish* Deploy to staging environment
3. Hold for manual approval
4. *Block execution, wait for earlier prod deploys to finish* Deploy to production environment

Job 2 will only block on earlier instances of job 2, even scheduled ones that aren't actually running yet. Independently, job 4 will also do so for earlier instances of job 4. Here, if multiple pushes are maded to a branch in succession, we can have the latest changes deployed to staging environment automatically (guaranteeing that old changes aren't deployed over newer ones e.g. if their tests happened to run slower), and same for production, even after waiting for manual approvals that could come out of order. This is nice because latest changes can still go to staging, even if production is being blocked purposely on an earlier change. 

Note that we do not paginate on the circleci endpoints, 

## Configuration Requirements

In order to use this orb you will need to export a `CIRCLECI_API_TOKEN` secret added to a context of your choosing. It will authentcation against the CircleCI API to check on workflow status. (see: <https://circleci.com/docs/api/v2/index.html#section/Authentication>)

---

## Resources

[CircleCI Orb Registry Page](https://circleci.com/orbs/registry/orb/tkoft/job-queue) - The official registry page of this orb for all versions, executors, commands, and jobs described.

[CircleCI Orb Docs](https://circleci.com/docs/2.0/orb-intro/#section=configuration) - Docs for using, creating, and publishing CircleCI Orbs.

### How to Contribute

We welcome [issues](https://github.com/tkoft/circleci-job-queue/issues) to and [pull requests](https://github.com/tkoft/circleci-job-queue/pulls) against this repository!

### How to Publish An Update

1. Merge pull requests with desired changes to the main branch.
    - For the best experience, squash-and-merge and use [Conventional Commit Messages](https://conventionalcommits.org/).
2. Find the current version of the orb.
    - You can run `circleci orb info tkoft/job-queue | grep "Latest"` to see the current version.
3. Create a [new Release](https://github.com/tkoft/circleci-job-queue/releases/new) on GitHub.
    - Click "Choose a tag" and _create_ a new [semantically versioned](http://semver.org/) tag. (ex: v1.0.0)
      - We will have an opportunity to change this before we publish if needed after the next step.
4. Click _"+ Auto-generate release notes"_.
    - This will create a summary of all of the merged pull requests since the previous release.
    - If you have used _[Conventional Commit Messages](https://conventionalcommits.org/)_ it will be easy to determine what types of changes were made, allowing you to ensure the correct version tag is being published.
5. Now ensure the version tag selected is semantically accurate based on the changes included.
6. Click _"Publish Release"_.
    - This will push a new tag and trigger your publishing pipeline on CircleCI.
