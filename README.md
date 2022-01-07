# PostGIS Workshops

Training materials for PostGIS

## Workshops

The workshop pages are viewable at below.
Note that we are working on the translations from English to other languages. 

* English:  https://postgis.net/workshops/en/postgis-intro/
* German:   https://postgis.net/workshops/de/postgis-intro/
* Japanese: https://postgis.net/workshops/ja/postgis-intro/
* Italian:  https://postgis.net/workshops/it/postgis-intro/
* Spanish:  https://postgis.net/workshops/es/postgis-intro/

## Translations
Currently in testing mode, if you want to help out, log into [OSGeo Weblate](https://weblate.osgeo.org/projects/postgis-workshop/)

If you don't already have an OSGeo account, [you can get one here](https://id.osgeo.org/ldap/create).

<a href="https://weblate.osgeo.org/engage/postgis-workshop/">
<img src="https://weblate.osgeo.org/widgets/postgis-workshop/-/287x66-grey.png" alt="Translation status" />
</a>

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
make pot
#for starters do de, ja, es
sphinx-intl update -p _build/gettext -l de -l ja -l es
```

## Building translated docs
```
cd postgis-intro/sources/en
LANG=ja make html-translation
LANG=de make html-translation
LANG=es make html-translation
LANG=it make html-translation
```

### Slide Decks

* TBD, probably using reveal.js
