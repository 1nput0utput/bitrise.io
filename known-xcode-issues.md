# Known Xcode (`xcodebuild`) issues

## Performance related

*Note: mainly affects UI tests.*

The root cause of the issue is that Xcode / the iOS Simulator has issues
in performance limited environments. This included Virtual Machines (which is
how your builds are running on [bitrise.io](https://www.bitrise.io)),
MacBook Airs, Mac Minis with HDD storage, ...

This issue is not Bitrise specific, it can happen even if you use
[Apple's Xcode Bots CI server](http://www.openradar.me/23386199) on non SSH
Mac Mini.

Examples:

* UI Tests fail to start
* One or more UI Test case hangs

### Possible solutions

* Try another Xcode version.
* Try another Simulator device (e.g. instead of running the test in "iPhone 6"
  try it with "iPhone 6s Plus")
* Some users had success with splitting the tests into multiple Schemes,
  and running those separately, with separate Test steps.


## Flaky UI tests, UI test cases failing randomly

This can happen with a really simple project too. Even something as
simple as:

```
func testAddAnItemGoToDetailsThenDeleteIt() {
        // Use recording to get started writing UI tests.
        // Use XCTAssert and related functions to verify your tests produce the correct results.


        let app = XCUIApplication()
        let masterNavigationBar = app.navigationBars["Master"]
        masterNavigationBar.buttons["Add"].tap()

        let tablesQuery = app.tables
        let firstElemQuery = tablesQuery.cells.elementBoundByIndex(0)
        firstElemQuery.tap()
        app.navigationBars.matchingIdentifier("Detail").buttons["Master"].tap()
        masterNavigationBar.buttons["Edit"].tap()

        firstElemQuery.buttons.elementBoundByIndex(0).tap()
        firstElemQuery.buttons["Delete"].tap()

        masterNavigationBar.buttons["Done"].tap()

        XCTAssert(tablesQuery.cells.count == 0)
    }
```

can trigger this issue.

### Possible solutions

We could reproduce this issue with the code above, using `Xcode 7.3`.
The exact same code worked perfectly with `Xcode 7.2.1` while it randomly
failed with `7.3`. The solution was to use a different iOS Simulator device.
The test failed *2 out of 3* on average with the "iPhone 6" simulator device
using Xcode 7.3, while it worked perfectly with Xcode 7.2.1.

Changing the simulator device to "iPhone 6s Plus" solved the issue with `Xcode 7.3`.


## Xcode Unit Test fails without any error, with exit code 65

This can be caused by a lot of things, Xcode or some other tool simply
omits / does not present any error message.

You can find a long discussion, with possible reasons & solutions [here](https://github.com/bitrise-io/bitrise.io/issues/5).

A quick summary:

* First of all, if you use `xcpretty` to format the output try a build without it
  (if you use the Xcode Test step you can set `xcodebuild` as the "Output Tool" option/input
  to not to format the log produced by `xcodebuild`). The cause is: `xcpretty` sometimes
  omits the error message in it's output. [Related GitHub issue](https://github.com/bitrise-io/bitrise.io/issues/27).
* If you don't use our `Xcode Test` step to run your UI Test you should try to run
  it with our Xcode Test step. We always try to improve the reliability of the step,
  implementing known workarounds for common issues.
* If you use our Xcode Test step: make sure you use the latest version, as it
  might include additional workarounds / fixes.
* Try [another Xcode version](http://devcenter.bitrise.io/docs/available-stacks#section-how-to-switch-to-the-new-beta-stacks),
  there are issues which are present in one Xcode version but not in another one.
* It might also be a [project configuration issue in your Xcode project](https://github.com/bitrise-io/bitrise.io/issues/5#issuecomment-140188658),
  or a [code issue in your tests](https://github.com/bitrise-io/bitrise.io/issues/5#issuecomment-160171566),
  or a [multi threading issue in your code](https://github.com/bitrise-io/bitrise.io/issues/5#issuecomment-190163069).
* We received reports that this might also be caused by Code Coverage report generation,
  you can disable the `Generate code coverage files?` option of the Xcode Test step
  to not to generate Code Coverage files.


## Every/Any Xcode command hangs

This is a rare issue, caused by running a **non shared Scheme**.

`xcodebuild` can only work with **shared Schemes**. If you try to run
a command on a non shared Scheme it usually manifests in a "scheme not found"
error, but we saw projects where it resulted in `xcodebuild` hanging, instead
of an error message.

If this is the case then any `xcodebuild` command will hang, even something
as simple as `xcodebuild -list`.

### Solution

[Make sure that you marked the Scheme as shared, and that you actually committed & pushed it into your repository](http://devcenter.bitrise.io/v1.0/docs/scheme-cannot-be-found).


## Build hangs

This is not Xcode related, but might be triggered by something in your
project when it runs in an Xcode step (Xcode Test, Xcode Archive, ...),
for example if you have a Run Phase Script in your Xcode project.

You can find pointers to identify and solve the issue [on our DevCenter](http://devcenter.bitrise.io/docs/step-hangs-times-out-after-a-period-without-any-logs-on-bitrise).
