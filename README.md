# sauce-espresso-test

A bitrise step to run espresso tests in sauce labs

## How to integrate this step in your workflow

#### Add the following secrets to your bitrise app:
```
- SAUCE_USERNAME: "<your_sauce_username_here>"
- SAUCE_ACCESS_KEY: "<your_sauce_access_key_here>"
```
Use those **exact names** for the secrets, otherwise this step won't work. The step is based on the [saucectl](https://docs.saucelabs.com/dev/cli/saucectl/), which reads your credentials from those two environment variables.

#### Add the step to your bitrise workflow yml:

```
---
format_version: '11'
default_step_lib_source: https://github.com/bitrise-io/bitrise-steplib.git
project_type: android
workflows:
  primary:
    steps:
    - gradle-runner@1:
        title: Assemble test apk
        description: |-
          Assembles the testApk. The test apk needs to be referenced in your .sauce/config.yml
        inputs:
          - gradle_task: :<your_module_here>:assembleDebugAndroidTest
          - gradlew_path: "./gradlew"
          - gradle_file: "./build.gradle"
    - git::https://github.com/Backbase/bitrise-step-sauce-espresso-test@0.0.1:
        title: sauce run
        inputs: #see other inputs in the step.yml file
        - sauce_config_yml: .sauce/config.yml
```

There are plenty of configurations you can use in [your saucelabs espresso configuration file](https://docs.saucelabs.com/mobile-apps/automated-testing/espresso-xcuitest/espresso/), but a typical .sauce/config.yml looks like this:

```
apiVersion: v1alpha
kind: espresso
sauce:
  region: eu-central-1
espresso:
  app: $BITRISE_STEP_SOURCE_DIR/sauceapp-debug.apk #this apk is provided with this bitrise step, but you can pass your own appApk here if you need.
  testApp: $BITRISE_TEST_APK_PATH
suites:
  - name: "default-test-suite"
    devices:
      - name: ".*"
        platformVersion: 8
        options:
          deviceType: PHONE
artifacts:
  download:
    when: always
    match:
      - junit.xml
    directory: $BITRISE_DEPLOY_DIR/artifacts/
```


## How to run this step locally

Can be run directly with the [bitrise CLI](https://github.com/bitrise-io/bitrise),
just `git clone` this repository, `cd` into it's folder in your Terminal/Command Line
and call `bitrise run test`.

*Check the `bitrise.yml` file for required inputs which have to be
added to your `.bitrise.secrets.yml` file!*

Step by step:

1. Open up your Terminal / Command Line
2. `git clone` the repository
3. `cd` into the directory of the step (the one you just `git clone`d)
5. Create a `.bitrise.secrets.yml` file in the same directory of `bitrise.yml`
   (the `.bitrise.secrets.yml` is a git ignored file, you can store your secrets in it)
6. Check the `bitrise.yml` file for any secret you should set in `.bitrise.secrets.yml`
7. Once you have all the required secret parameters in your `.bitrise.secrets.yml` you can just run this step with the [bitrise CLI](https://github.com/bitrise-io/bitrise): `bitrise run test`

An example `.bitrise.secrets.yml` file:

```
envs:
- SAUCE_CONFIG_YML: ./path/to/your/saucelabs/config.yml
- SAUCE_USERNAME: "<your_sauce_username_here>"
- SAUCE_ACCESS_KEY: "<your_sauce_access_key_here>"
```

## How to contribute to this Step

1. Fork this repository
2. `git clone` it
3. Create a branch you'll work on
4. To use/test the step just follow the **How to use this Step** section
5. Do the changes you want to
6. Run/test the step before sending your contribution
  * You can also test the step in your `bitrise` project, either on your Mac or on [bitrise.io](https://www.bitrise.io)
  * You just have to replace the step ID in your project's `bitrise.yml` with either a relative path, or with a git URL format
  * (relative) path format: instead of `- original-step-id:` use `- path::./relative/path/of/script/on/your/Mac:`
  * direct git URL format: instead of `- original-step-id:` use `- git::https://github.com/user/step.git@branch:`
  * You can find more example of alternative step referencing at: https://github.com/bitrise-io/bitrise/blob/master/_examples/tutorials/steps-and-workflows/bitrise.yml
7. Once you're done just commit your changes & create a Pull Request