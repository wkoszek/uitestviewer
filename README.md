# UI Test Viewer: view results of UI tests


## Introduction

The purpose of this repo is to write a server that can handle an upload of screenshots
and instrumentation from UI unit tests, and present those results in a logical way to the software
engineer via a simple and readable web UI.


## Introduction

`pytest` is a test framework popular within the Python ecosystem.
It can be used for writing backend and frontend unit tests.
The frontend functionality usually requires that you use one of the UI "accessory" libraries.
Selenium is an example: it gives you a way to control a multitude of web browsers through
a convenient Python API.
You can decide to look things up by the HTML `id` and `class` parameters, as well as input 
keystrokes and click buttons.

`pytest` and Selenium create a great technology together, however UI tests have a shortcomming:
unit tests will pass as long as HTML DOM structure is there, and backend code works; the
page can be "red letters on red background" is completely oblivious to the software.
In other words: software works as specified, but is useless to humans.

To see if UI makes sense, we'd benefit from seeing results of those tests in a simple and readable
web interface, organized by the hierarchy of those tests.
We can achieve that by snapping screenshots from the different stages of the tests, and then sending
them to the presentation server.
Along the screenshots, it'd be nice to present some additional instrumentation, like for example time
it took between certain calls, so that some information about the length of those tests could be specified.

## Resources

At least the timining info may be in JUnit test output.
Junit output is a standardized format of reporting the progress of unit test.
First research needs to be done on whether JUnit already supports the images, and if so, whether we could capture it in Python.

There's some IBM docs on this:

https://www.ibm.com/docs/en/elm/6.0.1?topic=rjsaet-adding-screen-captures-created-by-junit-selenium-test-execution-results

Maybe we can have a standardized way of capturing this output?

## Functional specs

The idea is simple: software engineers captures screenshots in the unit tests. For example, the code
could look like this:

```
    def test():
        self.screenshot("purchase_1.jpg")
        self.click_button("add")
        self.type_keys("item_name", "bananas")
        self.type_keys("item_price", 3.99)
        self.screenshot("purchase_2.jpg")
        self.click_button("submit")
       
```

And the `self.screenshot` could be (pseudocode):

```
def screenshow(fn):
    send a ping about starting the test NAME at time T
    capture the screenshot fn
    submit fn with appropriate test metadata to uitestviewer
```

In other words: we would grab the JPEG and send it to the UI Server Viewer.

## Specs

PyTest has two major concepts: test suite and a test.
Test suite is just a list of tests.
There's usually one test suite, but sometimes there can be very many for complicated projects.
For this project, let's call "run" a situation where we run all available test suites.

This test run must have a run ID: a name, let's say "ui-tr-yymmdd", and it's picked when you're running your python3 unit tests.
Think of it as a process ID (PID) of this very unique run.

Every run, we would:
- create a new unique runID (if I run all the tests on my laptop or CI server, it'd all be unique runIDs)
- every run should preferably have some additional meta-info, for example: hostname of the computer where the runID was created
- upon creation of runID at the beginning of a test run, runID would be reused for all the tests
- all tests would upload their screenshots to this runID
- upload of screenshots would happen with a test name, so that a reasonable hierarchy in HTML can be created.


### Proposed endpoints:
- GET to `/` will render a list of all run IDs via HTML
- POST to `/runid` will create a new runID. Submit "X-App-Name" as name of the app. Add this to DB.
- POST to `/runid/<ID>/upload` will cause the upload of the image to the server. Returned should be `uploadID`. Add this to DB.
  - required headers: `X-Test-Name` will have a test name
- GET to `/runid/<ID>` will show a pretty HTML page with thumbnails
  - this get will need `uploadID` of all items for this `runID`. Just get it from DB

Then every screenshot submitted to a given `runID` will allow the server to show appropriate JPEGs.

## Proposed UI

First screen after visiting main page:

```
Run IDs:

RunID   App name    Time created
c72DIA  app_name    feb23 2023
zx28aD  app_name    feb23 2023
bd63ij  app_name    feb24 2023
```

Once you click a RunID, you're routed to a page with a bunch of thumbnails:

```
Run IDs -> bd63ij

# App name

Test test_purchase (took N s)
  Started: <time>
  Ended: <time>

+------------+     +-------------+
| purch1.jpg |     |  purch2.jpg |
+------------+     +-------------+

Test another:

+------------+     +-------------+
| purch3.jpg |     |  purch4.jpg |
+------------+     +-------------+
+------------+     +-------------+
| purch5.jpg |     |  purch6.jpg |
+------------+     +-------------+

```

In other words, if there are 3 tests with 2, 5 and 3 screenshots respectively, the nice layout could be:

```
t1 t1

t5 t5 t5 t5 t5

t3 t3 t3
```

There t1/t5/t3 represent screenshots of respective tests.

Upon clicking of any of the thumbnail, it'd be great if big image popped up.

## Security

It has to have a mechanism to do authentication - we can start with YAML file with a flat-list of API keys.
If the Authorization header submitted to the server

## Tech specs

- Python3
- Flask
- SQLite
- "Don't bother with Docker"
- HTML5
- CSS3
- If there's a need for a grid etc, use Tachyons
- No React/Vue.js etc.
