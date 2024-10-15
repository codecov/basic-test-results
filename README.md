# basic-test-results
GitHub Actions to collect test results from a JUnit XML test output and emit a PR comment about the test status.

This action currently works only with JUnit format.

## Arguments

| Input  | Description | Required |
| :---       |     :---     |    :---:   |
| github-token | The secrets.GITHUB_TOKEN to be able to interact with the current PR and comment on it. | Required |
| directory | Directory to search for test result reports. Will run test processing on all files in the directory. | Optional |
| disable-search | Disable search for test result files. This is helpful when specifying what files you want to run test report processing with the --file option. | Optional |
| exclude | Directories to exclude from search. | Optional |
| file | Path to the JUnit file to run test result processing. | Optional |
| version | Specify which version of the Codecov CLI should be used. Defaults to `latest`. | Optional |

## Example Usage

### Basic Usage for Single File

```
steps:
  - ...

  - name: Test with Pytest exporting JUnit file
    run: |
      pytest --cov --junitxml=junit.xml
  - name: Run Basic Test Results Action
    uses: codecov/basic-test-results@v1
    with:
      github-token: ${{ secrets.GITHUB_TOKEN }}
      file: junit.xml
      disable-search: true
```

### Aggregating Multiple Test Files

If the CI includes multiple test files that we want to output one report for,
then the files will need to be uploaded as an artifact
and then downloaded into a directory.

```
jobs:
  run-python-stuff:
    - ...
    - name: Test with Pytest exporting JUnit file
      run: |
        pytest --cov --junitxml=junit.xml
    - name: Upload artifacts for test-results-processing
      if: ${{ !cancelled() }}
      uses: actions/upload-artifact@v4
      with:
        name: python-junit.xml
        path: junit.xml

  run-jest-stuff:
    - ...
    - name: Test with Jest exporting JUnit file
      run: |
        JEST_JUNIT_OUTPUT_NAME="junit.xml" jest
    - name: Upload artifacts for test-results-processing
      if: ${{ !cancelled() }}
      uses: actions/upload-artifact@v4
      with:
        name: jest-junit.xml
        path: junit.xml

  run-codecov-basic-test-results:
    runs-on: ubuntu-latest
    needs: [run-python-stuff, run-jest-stuff]
    steps:
      - name: Download all test results
        uses: actions/download-artifact@v4
        with:
          pattern: '*junit.xml'
          path: 'test_results'
          merge-multiple: true
      - name: Run Basic Test Results Action
        uses: ./
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          directory: 'test_results'
```