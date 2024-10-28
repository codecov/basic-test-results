<p align="center">
  <img src="images/Codecov umbrella only.png" alt="Codecov Logo" width=100 />
</p>


Basic Test Results lets you easily find test run failures as a pull request comment. 
All analysis is done locally on your CI. 
This is a standalone test results action that does not include code coverage 
or other metrics. 
Using this action does not require signing up to Codecov or any other service.

# 🏛️ Codecov - The Leading Test Coverage Solution 

If you’re interested in uncovering test coverage reporting, 
test analytics (including flake detection), 
and more code health insights for your repo, check out Codecov. 
Free forever for open source and public repos. 
Learn more at [https://about.codecov.io/](https://about.codecov.io/).


# 📢 Have thoughts?

Reach out to us on [Github](https://github.com/codecov/basic-test-results/issues).

# 🛠️ Setup

This action currently works only with the JUnit XML format. The action processes a JUnit XML test report locally and outputs results as a formatted PR comment.

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
      if: ${{ !cancelled() && github.event_name == 'pull_request'}}
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
      if: ${{ !cancelled() && github.event_name == 'pull_request'}}
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


# ❓FAQ

## How should the comment show up?

`basic-test-results` should create (or update an existing) comment when it is triggered. 
![image](https://github.com/user-attachments/assets/40fd768e-a53d-48c4-9b1d-3d31b2c81643)

The assumption is that there will be one comment per PR,
and that it gets updated on every PR push update.
If you see multiple messages, that means that there was a race condition
when creating the first message.
To avoid having the race condition, please ensure that your workflow
only runs `basic-test-results` once per workflow.
You can call `basic-test-results` for multiple JUnit files;
please see [this section](#aggregating-multiple-test-files) on how to set this up.
