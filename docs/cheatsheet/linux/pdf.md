# PDF

Mass decrypt and concat pdfs via fish:

```console
for f in crypt/*.pdf; qpdf --password='<password>' --decrypt $f ( basename $f ); end
qpdf --empty --pages *.pdf 1 -- "output.pdf"
```
