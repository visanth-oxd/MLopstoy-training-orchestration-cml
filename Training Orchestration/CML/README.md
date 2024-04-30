**What is CML?** Continuous Machine Learning (CML) is an open-source CLI tool
for implementing continuous integration & delivery (CI/CD) with a focus on
MLOps. Use it to automate development workflows — including machine
provisioning, model training and evaluation, comparing ML experiments across
project history, and monitoring changing datasets.

CML can help train and evaluate models — and then generate a visual report with
results and metrics — automatically on every pull request.

### GitHub

The key file in any CML project is `.github/workflows/cml.yaml`:

```yaml
name: your-workflow-name
on: [push]
jobs:
  run:
    runs-on: ubuntu-latest
    # optionally use a convenient Ubuntu LTS + DVC + CML image
    # container: ghcr.io/iterative/cml:0-dvc2-base1
    steps:
      - uses: actions/checkout@v3
      # may need to setup NodeJS & Python3 on e.g. self-hosted
      # - uses: actions/setup-node@v3
      #   with:
      #     node-version: '16'
      # - uses: actions/setup-python@v4
      #   with:
      #     python-version: '3.x'
      - uses: iterative/setup-cml@v1
      - name: Train model
        run: |
          # Your ML workflow goes here
          pip install -r requirements.txt
          python train.py
      - name: Write CML report
        env:
          REPO_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        run: |
          # Post reports as comments in GitHub PRs
          cat results.txt >> report.md
          cml comment create report.md
```

## Usage

We helpfully provide CML and other useful libraries pre-installed on our
[custom Docker images](https://github.com/iterative/cml/blob/master/Dockerfile).
In the above example, uncommenting the field
`container: ghcr.io/iterative/cml:0-dvc2-base1`) will make the runner pull the
CML Docker image. The image already has NodeJS, Python 3, DVC and CML set up on
an Ubuntu LTS base for convenience.

### CML Functions

CML provides a number of functions to help package the outputs of ML workflows
(including numeric data and visualizations about model performance) into a CML
report.

Below is a table of CML functions for writing markdown reports and delivering
those reports to your CI system.

| Function                  | Description                                                      | Example Inputs                                              |
| ------------------------- | ---------------------------------------------------------------- | ----------------------------------------------------------- |
| `cml runner launch`       | Launch a runner locally or hosted by a cloud provider            | See [Arguments](https://github.com/iterative/cml#arguments) |
| `cml comment create`      | Return CML report as a comment in your GitLab/GitHub workflow    | `<path to report> --head-sha <sha>`                         |
| `cml check create`        | Return CML report as a check in GitHub                           | `<path to report> --head-sha <sha>`                         |
| `cml pr create`           | Commit the given files to a new branch and create a pull request | `<path>...`                                                 |
| `cml tensorboard connect` | Return a link to a Tensorboard.dev page                          | `--logdir <path to logs> --title <experiment title> --md`   |

#### CML Reports

The `cml comment create` command can be used to post reports. CML reports are
written in markdown ([GitHub](https://github.github.com/gfm),
[GitLab](https://docs.gitlab.com/ee/user/markdown.html), or
[Bitbucket](https://confluence.atlassian.com/bitbucketserver/markdown-syntax-guide-776639995.html)
flavors). That means they can contain images, tables, formatted text, HTML
blocks, code snippets and more — really, what you put in a CML report is up to
you. Some examples:

:spiral_notepad: **Text** Write to your report using whatever method you prefer.
For example, copy the contents of a text file containing the results of ML model
training:

```bash
cat results.txt >> report.md
```

:framed_picture: **Images** Display images using the markdown or HTML. Note that
if an image is an output of your ML workflow (i.e., it is produced by your
workflow), it can be uploaded and included automaticlly to your CML report. For
example, if `graph.png` is output by `python train.py`, run:

```bash
echo "![](./graph.png)" >> report.md
cml comment create report.md
```
