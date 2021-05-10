# Makefile for Sphinx documentation
#

# You can set these variables from the command line.
SPHINXOPTS    =
SPHINXBUILD   = sphinx-build

# Internal variables.
OUTPUT          = ../../doc/en
SOURCE          = .
CACHE           = $(OUTPUT)/.build/doctrees
ALLSPHINXOPTS   = -d $(CACHE) $(SPHINXOPTS)

.PHONY: help clean html web pickle htmlhelp latex changes linkcheck

help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  html      to make standalone HTML files"
	@echo "  pickle    to make pickle files (usable by e.g. sphinx-web)"
	@echo "  htmlhelp  to make HTML files and a HTML help project"
	@echo "  latex     to make LaTeX files, you can set PAPER=a4 or PAPER=letter"
	@echo "  changes   to make an overview over all changed/added/deprecated items"
	@echo "  linkcheck to check all external links for integrity"
	@echo "  clean     to remove built artefacts"

clean:
	@rm -rf $(OUTPUT)

html:
	mkdir -p $(OUTPUT) $(CACHE)
	$(SPHINXBUILD) -b html $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)
	@echo
	@echo "Build finished. The HTML pages are in $(OUTPUT)."

#	find $(OUTPUT) -name .svn -type d -exec /bin/rm -rf {} ';'

pickle:
	mkdir -p $(OUTPUT) $(OUTPUT)/.build/pickle $(CACHE)
	$(SPHINXBUILD) -b pickle $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/.build/pickle
	@echo
	@echo "Build finished; now you can process the pickle files or run"
	@echo "  sphinx-web $(OUTPUT)/.build/pickle"
	@echo "to start the sphinx-web server."

web: pickle

htmlhelp:
	mkdir -p $(CACHE) $(OUTPUT)/.build/htmlhelp 
	$(SPHINXBUILD) -b htmlhelp $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/.build/htmlhelp
	@echo
	@echo "Build finished; now you can run HTML Help Workshop with the" \
	      ".hhp project file in $(OUTPUT)/.build/htmlhelp."

changes:
	mkdir -p $(CACHE) $(OUTPUT)/.build/changes 
	$(SPHINXBUILD) -b changes $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/.build/changes
	@echo
	@echo "The overview file is in $(OUTPUT)/.build/changes."

linkcheck:
	mkdir -p $(CACHE) $(OUTPUT)/.build/linkcheck 
	$(SPHINXBUILD) -b linkcheck $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/.build/linkcheck
	@echo
	@echo "Link check complete; look for any errors in the above output " \
	      "or in $(OUTPUT)/.build/linkcheck/output.txt."

