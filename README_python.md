# CI Demos

## GitHub Templates

GitHub workflow templates for CI:
* (Django)[https://github.com/actions/starter-workflows/blob/master/ci/django.yml]
* (Python app)[https://github.com/actions/starter-workflows/blob/master/ci/python-app.yml]
* (Python package) [https://github.com/actions/starter-workflows/blob/master/ci/python-package.yml]

## actions/setup-python

Documentation:
https://github.com/actions/setup-python

Basic usage:
```
steps:
- uses: actions/checkout@v2
- uses: actions/setup-python@v2
  with:
    python-version: '3.x' # Specify an exact Python version or use semantic versioning to test against the latest minor version of Python, as is the case in this example (e.g. 3.x)
    architecture: 'x64' # optional x64 or x86. Defaults to x64 if not specified
- run: python my_script.py
```

Matrix testing:
```
jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [ '2.x', '3.x', 'pypy2', 'pypy3' ]
    name: Python ${{ matrix.python-version }} sample
    steps:
      - uses: actions/checkout@v2
      - name: Setup python
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
          architecture: x64
      - run: python my_script.py
```

Exclude a specific version of Python when using matrix testing:
```
jobs:
  build:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
        python-version: [2.7, 3.6, 3.7, 3.8, pypy2, pypy3]
        exclude:
          - os: macos-latest
            python-version: 3.8
          - os: windows-latest
            python-version: 3.6
    steps:
      - uses: actions/checkout@v2
      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      - name: Display Python version
        run: python -c "import sys; print(sys.version)"
```

## Packaging workflow data as artifacts

You may need to persist log files, test results or screenshots. For more information, see "(Persisting workflow data using artifacts)[https://docs.github.com/en/github/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts]". The following example demonstrates how you can use (actions/upload-artifact)[https://github.com/actions/upload-artifact] to archive test results.

```
name: Python package

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: [3.7, 3.8]

      steps:
      - uses: actions/checkout@v2
      - name: Setup Python # Set Python version
        uses: actions/setup-python@v2
        with:
          python-version: ${{ matrix.python-version }}
      # Install pip and pytest
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install pytest
      - name: Test with pytest
        run: pytest tests.py --doctest-modules --junitxml=junit/test-results-${{ matrix.python-version }}.xml
      - name: Upload pytest test results
        uses: actions/upload-artifact@v1
        with:
          name: pytest-results-${{ matrix.python-version }}
          path: junit/test-results-${{ matrix.python-version }}.xml
        # Use always() to always run this step to publish test results when there are test failures
        if: ${{ always() }}
```

## Publishing to package registries

You can configure your workflow to publish your Python package to any package registry you'd like when your CI tests pass. Store any access tokens or credentials needed to publish your package using secrets. The following example creates and publishes a package to PyPI using twine and dist. For more information, see "(Creating and using encrypted secrets)[https://docs.github.com/en/github/automating-your-workflow-with-github-actions/creating-and-using-encrypted-secrets]."

```
name: Upload Python Package

on:
  release:
    types: [created]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python
      uses: actions/setup-python@v2
      with:
        python-version: '3.x'
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install setuptools wheel twine
    - name: Build and publish
      env:
        TWINE_USERNAME: ${{ secrets.PYPI_USERNAME }}
        TWINE_PASSWORD: ${{ secrets.PYPI_PASSWORD }}
      run: |
        python setup.py sdist bdist_wheel
        twine upload dist/*
```