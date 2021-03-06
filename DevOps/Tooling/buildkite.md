## BuildKite

- [Key facts](#key-facts)
- [Concurrency](#concurrency)
- [Metrics to monitor](#metrics)
- [Tips](#tips)

### Key Facts

- A `build` represents the whole process of running steps defined in pipeline upon code commits.
- Pipeline file includes a top-level `steps` which defines an array of separate step.
- Run of each step creates a new `job`
- Use `queue` to control where you want a `job` to run on. Agents are grouped by `queue`.

### Concurrency

Controls how many of jobs can run at any given time. Example below shows preview and deploy cannot happen concurrently across all builds. For instance, 3 builds are running, when the first build enters `preview` it locks the `concurrency_group`, so `preview` from other builds need to wait until first build finishes. The same applies to `deploy` in the same way.

```yaml
- label: ":pulumi: Preview"
  key: "preview"
  command: "preview.sh"
  agents:
    queue: 'aws'
  concurrency: 1
  concurrency_group: 'gate'

- label: ":pulumi: Deploy"
  command: "deploy.sh"
  agents:
    queue: 'aws'
  concurrency: 1
  concurrency_group: 'gate'
```

`concurrency_group` is a global thing that's accessible to all pipelines in your organisation.

### Metrics

[Link](https://github.com/buildkite/buildkite-agent-metrics/blob/master/README.md) shows available metrics you can monitor. Use case - query and visualize them on Grafana.

### Tips

- Look for `Preparing working directory` and unfold it to review source code working directory for debugging.
- `block` step will create implicit deps for steps before/after it. i.e

```yml

- label: 'Lint'
  key: 'lint'
  command: 'task lint'

- block: "block 1"
  key: "block-one"
  prompt: "Block one"
  branches: "!main"

...

# it will wait on block one meaning unless block one is run and passed, block two won't run
# unless depends_on is specified which will overwrite implicit deps
- block: "block 2"
  key: "block-two"
  prompt: "Block two"
  branches: "!main"
# depends_on:
#   - "lint"
```
