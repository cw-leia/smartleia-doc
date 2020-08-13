# Smartleia package documentation 

The current repository contains the smartleia python package documentation, and how
to compile it.

## Dependencies

To generate the whole documentation, the following utilities need to be installed:

```
sphinx
pdflatex
rst2man (which is part of the python-docutils package on Debian)
```

On a Debian distro you can simply execute:

```sh
apt install sphinx
apt install latexmk 
apt install texlive-latex-recommended
apt install texlive-latex-extra

pip3 install -r requirements.txt
```

Please note that `texlive` is mainly needed for the PDF output production.

## Building the documentation


### html 

Building the HTML version is as simple as:

```sh
make html
```

### PDF

Building the PDF version is as simple as:

```sh
make latexpdf
```


