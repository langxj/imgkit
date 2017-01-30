# IMGKit: Python library of HTML to IMG wrapper

[![Build Status](https://travis-ci.org/travis-ci/travis-web.svg?branch=master)](https://travis-ci.org/travis-ci/travis-web)

## Installation


1. Install imgkit:

  ``` python
  $ pip install imgkit
  ```

2. Install wkhtmltopdf:

  * Debian/Ubuntu:

    ``` bash
    sudo apt-get install wkhtmltopdf
    ```
    **Warning!** Version in debian/ubuntu repos have reduced functionality (because it compiled without the wkhtmltopdf QT patches), such as adding outlines, headers, footers, TOC etc. To use this options you should install static binary from [wkhtmltopdf](http://wkhtmltopdf.org/) site or you can use this [script](https://github.com/JiaKunUp/imgkit/blob/master/travis/init.sh).

  * MacOSX

    ``` bash
    brew install wkhtmltopdf
    ```

  * Windows and other options: check [wkhtmltopdf homepage](http://wkhtmltopdf.org/) for binary installers.

## Usage

Simple example:

``` python
import imgkit

imgkit.from_url('http://google.com', 'out.jpg')
imgkit.from_file('test.html', 'out.jpg')
imgkit.from_string('Hello!', 'out.jpg')
```

You can pass a list with multiple URLs or files:

``` python
imgkit.from_url(['google.com', 'yandex.ru', 'engadget.com'], 'out.jpg')
imgkit.from_file(['file1.html', 'file2.html'], 'out.jpg')
```

Also you can pass an opened file:

``` python
with open('file.html') as f:
    imgkit.from_file(f, 'out.jpg')
```

If you wish to further process generated IMG, you can read it to a variable:

``` python
# Use False instead of output path to save pdf to a variable
img = imgkit.from_url('http://google.com', False)
```

You can find all wkhtmltoimage options by type `wkhtmltoimage` command. You can drop '--' in option name. If option without value, use *None, False* or *''* for dict value:. For repeatable options (incl. allow, cookie, custom-header, post, postfile, run-script, replace) you may use a list or a tuple. With option that need multiple values (e.g. --custom-header Authorization secret) we may use a 2-tuple (see example below).

``` python
options = {
    'format': 'jpg',
    'crop-h': '3',
    'crop-w': '3',
    'crop-x': '3',
    'crop-y': '3',
    'encoding': "UTF-8",
    'custom-header' : [
        ('Accept-Encoding', 'gzip')
    ]
    'cookie': [
        ('cookie-name1', 'cookie-value1'),
        ('cookie-name2', 'cookie-value2'),
    ],
    'no-outline': None
}

imgkit.from_url('http://google.com', 'out.jpg', options=options)
```

By default, IMGKit will show all `wkhtmltoimage` output. If you don't want it, you need to pass `quiet` option:


``` python
options = {
    'quiet': ''
    }

imgkit.from_url('google.com', 'out.jpg', options=options)
```

Due to wkhtmltoimage command syntax, **TOC** and **Cover** options must be specified separately. If you need cover before TOC, use `cover_first` option:

``` python
toc = {
    'xsl-style-sheet': 'toc.xsl'
}

cover = 'cover.html'

imgkit.from_file('file.html', options=options, toc=toc, cover=cover)
imgkit.from_file('file.html', options=options, toc=toc, cover=cover, cover_first=True)
```

You can specify external CSS files when converting files or strings using *css* option.

``` python
# Single CSS file
css = 'example.css'
imgkit.from_file('file.html', options=options, css=css)

# Multiple CSS files
css = ['example.css', 'example2.css']
imgkit.from_file('file.html', options=options, css=css)
```

You can also pass any options through meta tags in your HTML:


``` python
body = """
    <html>
      <head>
        <meta name="imgkit-format" content="jpg"/>
        <meta name="imgkit-orientation" content="Landscape"/>
      </head>
      Hello World!
      </html>
    """

imgkit.from_string(body, 'out.jpg')
```

## Configuration

Each API call takes an optional config paramater. This should be an instance of `imgkit.config()` API call. It takes the config options as initial paramaters. The available options are:

* `wkhtmltoimage` - the location of the wkhtmltoimage` binary. By default imgkit` will attempt to locate this using which` (on UNIX type systems) or where` (on Windows).
* `meta_tag_prefix` - the prefix for pdfkit` specific meta tags - by default this is imgkit-`

Example - for when wkhtmltopdf` is not in $PATH`:

``` python
config = imgkit.config(wkhtmltoimage='/opt/bin/wkhtmltoimage')
imgkit.from_string(html_string, output_file, config=config)
```


## Troubleshooting

* `IOError: 'No wkhtmltopdf executable found'`:

  Make sure that you have wkhtmltoimage in your `$PATH` or set via custom configuration (see preceding section). *where wkhtmltoimage* in Windows or *which wkhtmltoimage* on Linux should return actual path to binary.

* `IOError: 'Command Failed'`:

  This error means that IMGKit was unable to process an input. You can try to directly run a command from error message and see what error caused failure (on some wkhtmltoimage versions this can be cause by segmentation faults)
