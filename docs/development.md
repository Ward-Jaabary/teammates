# Development Guidelines

These are the common tasks involved when working on features, enhancements, bug fixes, etc. for TEAMMATES.

* [Managing the dev server](#managing-the-dev-server)
* [Logging in to a TEAMMATES instance](#logging-in-to-a-teammates-instance)
* [Testing](#testing)
* [Measuring code coverage](#measuring-code-coverage)
* [Deploying to a staging server](#deploying-to-a-staging-server)
* [Running client scripts](#running-client-scripts)
* [Config points](#config-points)

The instructions in all parts of this document work for Linux, OS X, and Windows, with the following pointers:
- Replace `./gradlew` to `gradlew.bat` if you are using Windows.
- All the commands are assumed to be run from the root project folder, unless otherwise specified.
- It is assumed that the development environment has been correctly set up. If this step has not been completed, refer to [this document](setting-up.md).

> If you encounter any problems during the any of the processes, please refer to our [troubleshooting guide](troubleshooting-guide.md) before posting a help request on our [issue tracker](https://github.com/TEAMMATES/teammates/issues).

## Managing the dev server

> `Dev server` is the server run in your local machine.

### Building JavaScript files

Our JavaScript code is written in modular ECMAScript 6 (ES6) syntax, which is not supported in many of the existing Web browsers today.<br>
To resolve this, we need to *bundle and transpile* ("build" afterwards) them into standard ECMAScript 5 which is supported by (almost) all browsers.

Run the following command to build the JavaScript files for the application's use in development mode:
```sh
npm run dev
```

This command needs to be run before starting the dev server if there are updates to the JavaScript files.

> Run the command `npm run build` instead for Production mode. This will optimize the build with minified JavaScript Files and produce full source maps.

### With command line

#### Starting the dev server

To start the server in the background, run the following command
and wait until the task exits with a `BUILD SUCCESSFUL`:
```sh
./gradlew appengineStart
```

To start the server in the foreground (e.g. if you want the console output to be visible),
run the following command instead:
```sh
./gradlew appengineRun
```

The dev server URL will be `http://localhost:8080` as specified in `build.gradle`.

#### Stopping the dev server

If you started the server in the background, run the following command to stop it:
```sh
./gradlew appengineStop
```

If the server is running in the foreground, press `Ctrl + C` to stop it or run the above command in a new console.

### With Eclipse

#### Starting the dev server

Right-click on the project folder and choose `Run As → App Engine`.<br>
After some time, you should see this message (or similar) on the Eclipse console: `Dev App Server is now running`.
The dev server URL will be given at the console output, e.g `http://localhost:8080`.

#### Stopping the dev server

Click the "Terminate" icon on the Eclipse console.

### With IntelliJ

#### Starting the dev server

Go to `Run → Run...` and select `Google App Engine Standard Local Server` in the pop-up box.

#### Stopping the dev server

Go to `Run → Stop 'Google App Engine Standard Local Server'`.

## Logging in to a TEAMMATES instance

This instruction set applies for both dev server and production server, with slight differences explained where applicable.
- The local dev server is assumed to be accessible at `http://localhost:8080`.
- If a URL is given as relative, prepend the server URL to access the page, e.g `/page/somePage` is accessible in dev server at `http://localhost:8080/page/somePage`.

### As administrator

1. Go to any administrator page, e.g `/admin/adminHomePage`.
1. On the dev server, log in using any username, but remember to check the `Log in as administrator` check box. You will have the required access.
1. On the production server, you will be granted the access only if your account has administrator permission to the application.
1. When logged in as administrator, ***masquerade mode*** can also be used to impersonate instructors and students by adding `user=username` to the URL
 e.g `http://localhost:8080/page/studentHomePage?user=johnKent`.

### As instructor

You need an instructor account which can be created by administrators.

1. Log in to `/admin/adminHomePage` as an administrator.
1. Enter credentials for an instructor, e.g<br>
   Name: `John Dorian`<br>
   Email: `teammates.instructor@university.edu`<br>
   Institution: `National University of Singapore`<br>
1. The system will send an email containing the join link to the added instructor.<br>
   On the dev server, this email will not be sent. Instead, you can use the join link given after adding an instructor to complete the joining process.

Alternatively, an instructor can create other instructors for a course if s/he has sufficient privileges. A course co-owner, for example, will have such a privilege.

1. Ensure that there is a course to add instructors to and an instructor in that course with the privilege to add instructors.
1. Log in as that instructor.
1. Add the instructors for the course (`Instructors` → `View/Edit`).
1. The system will send an email containing the join link to each added instructor. Again, this will not happen on the dev server, so additional steps are required.
1. Log out and log in to `http://localhost:8080/admin/adminSearchPage` as administrator.
1. Search for the instructor you added in. From the search results, click anywhere on the desired row to get the course join link for that instructor.
1. Log out and use that join link to log in as the new instructor.

### As student

You need a student account which can be created by instructors (with sufficient privileges).

The steps for adding a student is almost identical to the steps for adding instructors by another instructor:
- Where appropriate, change the reference to "instructor" to "student".
- `Students` → `Enroll` to add students for the course.

**Alternative**: Run the test cases, they create several student and instructor accounts in the datastore. Use one of them to log in.

## Testing

TEAMMATES automated testing requires Firefox or Chrome.

Before running tests, modify `src/test/resources/test.properties` if necessary, e.g. to configure which browser and test accounts to use.

### Using Firefox

* Only Firefox between versions 38.0.5 and 46.0.1 are supported.
  * To downgrade your Firefox version, obtain the executable from [here](https://ftp.mozilla.org/pub/mozilla.org/firefox/releases/).
  * If you want to use a different path for this version, choose `Custom setup` during install.
  * Remember to disable the auto-updates (`Options → Advanced tab → Update`).

* If you want to use a Firefox version other than your computer's default, specify the custom path in `test.firefox.path` value in `test.properties`.

* If you are planning to test changes to JavaScript code, disable JavaScript caching for Firefox:
  * Enter `about:config` into the Firefox address bar and set `network.http.use-cache` (or `browser.cache.disk.enable` in newer versions of Firefox) to `false`.

### Using Chrome

* You need to use chromedriver for testing with Chrome.
  * Download the latest stable chromedriver from [here](https://sites.google.com/a/chromium.org/chromedriver/downloads).
    The site will also inform the versions of Chrome that can be used with the driver.
  * Specify the path to the chromedriver executable in `test.chromedriver.path` value in `test.properties`.

* If you are planning to test changes to JavaScript code, disable JavaScript caching for Chrome:
  * Press Ctrl+Shift+J to bring up the Web Console.
  * Click on the settings button at the bottom right corner.
  * Under the General tab, check "Disable Cache".

* The chromedriver process started by the test suite will not automatically get killed after the tests have finished executing.<br>
  You will need to manually kill these processes after the tests are done.
  * On Windows, use the Task Manager or `taskkill /f /im chromedriver.exe` command.
  * On OS X, use the Activity Monitor or `sudo killall chromedriver` command.

### Running the test suites

#### Test Configurations

Several configurations are provided by default:
* `CI tests` - Runs `src/test/testng-ci.xml`, all the tests that are run by CI (Travis/AppVeyor).
* `Local tests` - Runs `src/test/testng-local.xml`, all the tests that need to be run locally by developers. `Dev green` means passing all the tests in this configuration.
* `Failed tests` - Runs `test-output/testng-failed.xml`, which is generated if a test run results in some failures. This will run only the failed tests.

#### God Mode

To ensure that the UI generated by the application is the same as what is expected, some tests work by comparing the raw HTML text of Web pages generated by the application against the corresponding content of a pre-recorded expected HTML text files. These files need to be updated every time a UI-related change is observed (e.g. when adding a new feature to the UI).
In order to save the time and effort for such update, we have a tool called ["God Mode"](godmode.md). This can be activated by setting the value of `test.godmode.enabled` in `test.properties` to `true`.

Assuming you have already done the UI-related change and tested it manually, the procedure to update the expected HTML files of the relevant tests is as follows:

- Run the test which does the HTML files comparison with God Mode activated. Under this mode, the test runner will capture the Web pages being generated by the application and use a post-processed version of it to overwrite the existing expected HTML files.
- Manually examine the diff of the updated expected HTML files to ensure that the changes are as expected.
- Run the test again without God Mode activated and ensure that it passes.

The same God Mode technique is also used to ensure the content of emails generated by the application are as expected.

### Running the test suite with command line

Before running any test suite, it is important to have the dev server running locally first if you are testing against it.

Test suite | Command | Results can be viewed in
---|---|---
`CI tests` | `./gradlew ciTests` | `{project folder}/build/reports/test-try-{n}/index.html`, where `{n}` is the sequence number of the test run
`Local tests` | `./gradlew localTests` | `{project folder}/build/reports/test-local/index.html`
`Failed tests` | `./gradlew failedTests` | `{project folder}/build/reports/test-failed/index.html`
Any individual test | `./gradlew test -Dtest.single=TestClassName` | `{project folder}/build/reports/tests/index.html`

`CI tests` will be run once and the failed tests will be re-run a few times.
All other test suites will be run once and only once.

### Running the test suite with an IDE

* An additional configuration `All tests` is provided, which will run `CI tests` and `Local tests`.
* Additionally, configurations that run the tests with `GodMode` turned on are also provided.
* When running the test cases, if a few cases fail (this can happen due to timing issues), re-run the failed cases using the `Run Failed Test` icon in the TestNG tab until they pass.

#### Eclipse

Run tests using the configurations available under the green `Run` button on the Eclipse toolbar.

Sometimes, Eclipse does not show these options immediately after you set up the project. "Refreshing" the project should fix that.

To run individual tests, right-click on the test files on the project explorer and choose `Run As → TestNG Test`.

#### IntelliJ

Run tests using the configurations available under `Run → Run...`.

To run individual tests, right-click on the test files on the project explorer and choose `Run`.

### Testing against production server

If you are testing against a production server (staging server or live server), some additional tasks need to be done.

1. You need to setup a `Gmail API`<sup>1</sup> as follows:
   * [Obtain a Gmail API credentials](https://github.com/TEAMMATES/teammates-ops/blob/master/platform-guide.md) and download it.
   * Copy the file to `src/test/resources/gmail-api` (create the `gmail-api` folder) of your project and rename it to `client_secret.json`.
   * It is also possible to use the Gmail API credentials from any other Google Cloud Platform project for this purpose.

1. Edit `src/test/resources/test.properties` as instructed is in its comments.
   * In particular, you will need legitimate Google accounts to be used for testing.

1. Run the full test suite or any subset of it as how you would have done it in dev server.
   * You may want to run `InstructorCourseDetailsPageUiTest` standalone first because you would need to login to test accounts for the first time.
   * Do note that the GAE daily quota is usually not enough to run the full test suite, in particular for accounts with no billing enabled.

<sup>1</sup> This setup is necessary because our test suite uses the Gmail API to access Gmail accounts used for testing (these accounts are specified in `test.properties`) to confirm that those accounts receive the expected emails from TEAMMATES.
This is needed only when testing against a production server because no actual emails are sent by the dev server and therefore delivery of emails is not tested when testing against the dev server.

## Measuring code coverage

After or concurrently with testing, one may want to check the code coverage to see how much of the code base has (not) been covered by tests.

### CLI

You can use Gradle to obtain the coverage data with `jacocoReport` task after running tests, e.g.:
```sh
./gradlew appengineRun ciTests jacocoReport
```
The report can be found in the `build/reports/jacoco/jacocoReport/` directory.

For JavaScript unit tests, coverage is done concurrently with the tests themselves.

### Eclipse

You need [EclEmma Java Code Coverage plugin](https://marketplace.eclipse.org/content/eclemma-java-code-coverage) to run code coverage session with Eclipse.

For Java tests, choose `Coverage as TestNG Test` instead of the usual `Run as TestNG Test` to run the specified test or test suite.
The coverage will be reported in Eclipse after the test run is over.

For JavaScript unit tests, simply open `allJsUnitTests.html` and tick `Enable coverage`, or run `AllJsTests.java`.
The coverage will be reported immediately in the test page.

### IntelliJ IDEA

For Java tests, you can measure code coverage for the project using `Run → Run... → CI Tests → ▶ → Cover `.

Alternatively, you can right click a class with TestNG test(s) and click `Run '$FileClass$' with Coverage`, this will use IntelliJ IDEA's code coverage runner.
You can further configure your code coverage settings by referring to [IntelliJ IDEA's documentation](https://www.jetbrains.com/help/idea/2017.1/code-coverage.html).

For JavaScript unit tests, simply open `allJsUnitTests.html` and tick `Enable coverage`, or run `AllJsTests.java`.
The coverage will be reported immediately in the test page.

### Travis CI

For Java tests, if your build and run is successful, [Codecov](https://codecov.io) will pull the test coverage data and generate a report on their server.
The link to the report will be displayed in each PR, or by clicking the badge on the repository homepage.

For JavaScript unit tests, coverage is done concurrently with the tests themselves.

## Deploying to a staging server

> `Staging server` is the server instance you set up on Google App Engine for hosting the app for testing purposes.

For most cases, you do not need a staging server as the dev server has covered almost all of the application's functionality.
If you need to deploy your application to a staging server, refer to [this guide](https://github.com/TEAMMATES/teammates-ops/blob/master/platform-guide.md#deploying-to-a-staging-server).

## Running client scripts

> Client scripts are scripts that remotely manipulate data on GAE via its Remote API. They are run as standard Java applications.

Most of developers may not need to write and/or run client scripts but if you are to do so, take note of the following:

* If you are to run a script in a production environment, there are additional steps to follow. Refer to [this guide](https://github.com/TEAMMATES/teammates-ops/blob/master/platform-guide.md#running-client-scripts).
* It is not encouraged to compile and run any script via command line; use any of the supported IDEs to significantly ease this task.

## Config points

There are several files used to configure various aspects of the system.

**Main**: These vary from developer to developer and are subjected to frequent changes.
* `build.properties`: Contains the general purpose configuration values to used by the web app.
* `test.properties`: Contains the configuration values for the test driver.
* `appengine-web.xml`: Contains the configuration for deploying the application on GAE.

**Tasks**: These do not concern the application directly, but rather the development process.
* `build.gradle`: Contains the server-side third-party dependencies specification, as well as configurations for automated tasks/routines to be run via Gradle.
* `gradle.properties`, `gradle-wrapper.properties`: Contains the Gradle and Gradle wrapper configuration.
* `package.json`: Contains the client-side third-party dependencies specification.
* `.travis.yml`: Contains the Travis CI job configuration.
* `appveyor.yml`: Contains the AppVeyor CI job configuration.

**Static Analysis**: These are used to maintain code quality and measure code coverage. See [Static Analysis](static-analysis.md).
* `static-analysis/*`: Contains most of the configuration files for all the different static analysis tools.
* `.stylelintrc`: Equivalent to `static-analysis/teammates-stylelint.yml`, currently only used for Stylelint integration in IntelliJ.

**Other**: These are rarely, if ever will be, subjected to changes.
* `logging.properties`: Contains the java.util.logging configuration.
* `web.xml`: Contains the web server configuration, e.g servlets to run, mapping from URLs to servlets/JSPs, security constraints, etc.
* `cron.xml`: Contains the cron jobs specification.
* `queue.xml`: Contains the task queues configuration.
* `datastore-indexes.xml`: Contains the Datastore indexes configuration.
