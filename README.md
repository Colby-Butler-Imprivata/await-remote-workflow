
# await-remote-workflow
A Github action that triggers a remote workflow and tracks down its exit status

In a multi git repository environment we have today, we sometimes face with the need to trigger a remote workflow from a running workflow when both are located in different repositories. Triggering a remote workflow is not a big deal, however waiting for a reliable exit status and building job dependencies based on that exit status - is not inherently supported in Github and requires some implementation effort on our end. This is due to the fact that when you submit a POST request to Github’s API to trigger a workflow, you get no output back. Ideally, one should have at least received the ‘Run ID’ of the triggered workflow, but that’s not the case.

In this document I’ll describe how my solution is implemented by harnessing Github’s ‘artifacts’ feature to track a workflow’s exit code.
