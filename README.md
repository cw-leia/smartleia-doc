# Smartleia package documentation 

The current repository contains the smartleia python package documentation, and how
to compile it.

**NOTE**: the **smartleia** [Python package](https://github.com/cw-leia/smartleia.git) must
be installed for the documentation to compile!

## Dependencies

To generate the whole documentation, the following utilities need to be installed:

```
pandoc
sphinx
pdflatex
rst2man (which is part of the python-docutils package on Debian)
```

Some Python packages are also required and are compiled in the `requirements.txt` file
(in addition to the wheel and setuptools package).

On a Debian distro you can simply execute:

```sh
apt install pandoc latexmk texlive-latex-recommended texlive-latex-extra texlive-fonts-recommended

apt install python3-wheel python3-setuptools

pip3 install --user wheel
pip3 install --user setuptools
pip3 install --user -r requirements.txt
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


