# Presentation guidelines

This presentation was created using [Reveal.js](https://revealjs.com/#/), the HTML presentation framework.

In order to present, we spin a node.js server wrapped in a docker container:

[https://www.npmjs.com/package/reveal.js-online](https://www.npmjs.com/package/reveal.js-online)

## INSTALL

``` bash
npx reveal.js-online@1.0.1
```

A server will run on:

http://localhost:8001/

Use this page to build your presentation, using markdown.

Use [reveal.html](reveal.html) to style the presentation, using CSS.

Preview the presentation on:

[http://localhost:8001/reveal.html#/](http://localhost:8001/reveal.html#/)

Press `f` for full-screen and `s` for speakers notes.

## EXPORT

You can export the slide deck to pdf, by calling this url on your browser:

[http://localhost:8001/reveal.html?print-pdf#/](http://localhost:8001/reveal.html?print-pdf#/)

Then printing it to a file, using the pdf driver.

You can convert the pdf to multiple images, using [imagemagick](https://imagemagick.org/index.php):

 convert -density 150 input_file.pdf -quality 100 output_file.png


## MAINTAINER

This presentation was created by [Joana Simoes](mailto:joana@codeop.tech).