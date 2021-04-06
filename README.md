![Bell Lab Logo](https://bell-lab.s3-us-west-1.amazonaws.com/bell-lab-logo.png "Bell Lab Logo")

Metricsgrade
---

> This tool measures any webpage for performances. This project uses [Puppeteer](https://github.com/GoogleChrome/puppeteer) to make painless automation.

Installation
---

To install metricsgrade:

```bash
git clone https://github.com/bell-lab-apps/metricsgrade.git
cd metricsgrade
npm install -g
```

Usage
---

```bash
metricsgrade <url>
```

Several options are available for ease of use. Use `-h (--help)` to display them.

```console
➜ metricsgrade -h
  Usage: metricsgrade [options] <url ...>

  Measures webpage loading metrics

  Options:

    -r, --repeat [n]                     The number of times the page metrics are measured (default: 5)
    -w, --width [width]                  The viewport's width to set (default: 1920)
    -H, --height [height]                The viewport's height to set (default: 1080)
    -c, --custom-path [custom-path]      Path to custom path configuration file
    -o, --output-format [output-format]  The desired output format (default: table)
    --wait-until [wait-until]            The waitUntil value of the Page.reload options accepted by puppeteer
    --output-file [output-file]          Whether we want to export data in a file, and the desired path to the file
    --no-headless                        Defines if we dont want to use puppeteer headless mode
    --no-sandbox                         Disable chrome sandbox mode, mandatory in some systems
    -h, --help                           output usage information
```

Custom user path
---

A custom file path can be set in the cli options. That way, you can tell puppeteer what he should do before measuring any kind of metric.

This option can be useful if you need to be logged in before being able to access to your application.

To include your file into the process, juste use `-c <relative path to your file>` option.

```bash
metricsgrade localhost:8000 -c '../../custom-path.js'
```

The `custom-path.js` file should contain an exported ES module.

```javascript
// index.js: The custom path function is called like so :
if (customPath) {
    const customPathFunction = require(customPath);
    await customPathFunction(page, logInfo);
}
// custom-path.js: example of login process
const LOGIN_INPUT = 'input[type="login"]';
const PASSWORD_INPUT = 'input[type="password"]';
module.exports = async (page, logInfo) => {
    const login = 'my-secret-login';
    const password = 'my-really-secret-password';
    const loginUrl = 'http://localhost:8080/login';
    logInfo(`Loading ${loginUrl}`);
    // Go to the login page url, and wait for the selector to be ready.
    await page.goto(loginUrl);
    await page.waitForSelector(LOGIN_INPUT);
    logInfo('Logging in...');
    // Type credentials.
    await page.type(LOGIN_INPUT, login);
    await page.type(PASSWORD_INPUT, password);
    logInfo('Redirecting');
    // The process will continue once the redirect is resolved.
    return page.waitForNavigation();
};
```

Those functions have accessed to two arguments :

-   `page` (The `Page` puppeteer object to be able to access to the full [puppeteer page instance API](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-page))
-   `logInfo` (To log custom information)

### Export data in a file

You can choose to export in multiple formats and export formatted data in a file. For now, only `table`, `raw`, `json` and `csv` are available.
`table` and `raw` data will be exported in a `txt` file. To use it, just type.

```bash
metricsgrade localhost:8000 --output-format json --output-file ~/results.json
```

If you don't provide any filename, a file will automatically be created in your current directory.

### "Wait until" option

To make a page reload, `metricsgrade` does a `Page.reload()` from puppeteers `Page` object. This object accepts a `waitUntil` parameter, which defines when the page navigation has succeeded, and when the application should collect the metrics and reload the page. You can find more information about `Page.reload()` [right here](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagereloadoptions).

To use, just add `--wait-until` flag and the desired options. Since `Page.reload` accepts and `Array` of `Strings`, if you want to add multiple options, just split them with a `,`

For example:

```bash
metricsgrade localhost:8000 --wait-until networkidle0,load
```

## Useful Resources

-   [Commander documentation](https://github.com/tj/commander.js)
-   [Puppeteer API](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)
