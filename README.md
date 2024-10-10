This repo is meant to help illustrate an issue seen with the NPM dependabot updater; jobs are being closed.

The repro steps are meant to mimic (1) an intial job creating a PR, and (2) a subsequent update job.  The result is that the second job is closing the PR opened by the first job.

The first job that creates a single PR can be run with `job1.yml`.  You'll see output similar to this:

```
+---------------------------------------------------------------------------------------------------------------------------------+
|                                               Changes to Dependabot Pull Requests                                               |
+---------+-----------------------------------------------------------------------------------------------------------------------+
| created | path-to-regexp ( from 0.1.7 to 0.1.10 ), react-router-dom ( from 5.3.4 to 6.26.2 ), express ( from 4.18.2 to 4.21.0 ) |
+---------+-----------------------------------------------------------------------------------------------------------------------+
```

The second job meant to update the first PR can be run with `job2.yml`.  You'll see output similar to this:

```
+-------------------------------------------------------------------------------------+
|                         Changes to Dependabot Pull Requests                         |
+------------------------------+------------------------------------------------------+
| closed: dependencies_changed | express,path-to-regexp,react-router,react-router-dom |
| created                      | express ( from 4.18.2 to 4.21.0 )                    |
+------------------------------+------------------------------------------------------+
```

The second job is closing the PR because [this condition](https://github.com/dependabot/dependabot-core/blob/aa68330ec4263626106806b988fafb2ca6634c7f/updater/lib/dependabot/updater/operations/refresh_security_update_pull_request.rb#L177) is
evaluating to `false`, where `dependency_change.updated_dependencies.map { |x| x.name.downcase }` is evaluating to `["express"]` and `job_dependencies` is evaluating to `["express", "path-to-regexp", "react-router", "react-router-dom"]`.

The value `job_dependencies` is obviously coming from `job2.yml`, but `dependency_change.updated_dependencies...` is only evaluating to the single value `"express"`, but when running `job1.yml` all four dependencies are detected and updated (hence the PR created from `job1.yml`).
