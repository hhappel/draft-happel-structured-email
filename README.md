# draft-ietf-sml-structured-email

This repo contains sources for https://datatracker.ietf.org/doc/draft-ietf-sml-structured-email/

## Convert
This section describes how the markdown source can be converted to XML or PDF.

### Markdown to XML
* Make sure you have installed [Mmark](https://mmark.miek.nl/)
* Run:
% ./mmark filename.md > filename.xml

### XML to PDF
* Make sure you have installed [XML2RFC](https://github.com/ietf-tools/xml2rfc)
* Run:
% xml2rfc --v3 filename.xml --pdf
