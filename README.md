# PostGIS Workshops

Training materials for PostGIS

## Workshops

The workshop pages are viewable at: https://postgis.net/workshops/postgis-intro/

## Building the Workshop Content

* Install Sphinx: http://www.sphinx-doc.org/en/1.8/usage/installation.html
* Go to the source directory: `cd postgis-intro/sources/en/`
* Build the HTML: `make html`
* Check the outputs: `cd ../../doc/en`

If you get an error about TemplateNotFound('sphinxdoc/layout.html') then
Try doing:
`pip install sphinx-rtd-theme`

## running in Python3 with virtual-env
```
sudo apt install python3-pip
sudo apt install python3-venv
python3 -m venv ~/pw-env
source ~/pw-env/bin/activate #enter the venv
pip3 install sphinx-rtd-theme
```

## setting up po
```
pip install sphinx-intl
cd postgis-intro/sources/en
make gettext
#for starters do de, ja, es
sphinx-intl update -p _build/gettext -l de -l ja -l es
```

### Slide Decks

* TBD, probably using reveal.js
