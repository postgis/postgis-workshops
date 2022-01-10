# Makefile for Sphinx documentation
#

# You can set these variables from the command line.
SPHINXOPTS    =
SPHINXBUILD   = sphinx-build

# Internal variables.
OUTPUT          = ../../doc
BUILDDIR				= ../_build
SOURCE          = .
CACHE           = $(OUTPUT)/.build/en/doctrees
ALLSPHINXOPTS   = -d $(CACHE) $(SPHINXOPTS)

.PHONY: help
help: ## Show this help screen
	@grep -E '^[a-zA-Z0-9_-]+:.*?## .*$$' $(MAKEFILE_LIST) | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'

# @echo "Please use \`make <target>' where <target> is one of"
# @echo "  pickle    to make pickle files (usable by e.g. sphinx-web)"
# @echo "  htmlhelp  to make HTML files and a HTML help project"
# @echo "  latex     to make LaTeX files, you can set PAPER=a4 or PAPER=letter"
# @echo "  changes   to make an overview over all changed/added/deprecated items"
# @echo "  linkcheck to check all external links for integrity"
# @echo "  clean     to remove built artefacts"

.PHONY: clean
clean: ## Remove all output files
	@rm -rf $(OUTPUT)

.PHONY: html
html: ## to make standalone en HTML files
	mkdir -p $(OUTPUT)/en $(CACHE)
	$(SPHINXBUILD) -b html $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/en
	@echo
	@echo "Build finished. The HTML pages are in $(OUTPUT)."

.PHONY: html-translation
html-translation: ## use `LANG=ja make html-translation`, replace ja with language of your choosing 
	echo "Building for $(LANG)"
	mkdir -p $(OUTPUT)/$(LANG) $(OUTPUT)/.build/$(LANG)/doctrees
	$(SPHINXBUILD) -b html -d $(OUTPUT)/.build/$(LANG)/doctrees -D language=$(LANG) $(SOURCE) $(OUTPUT)/$(LANG)

.PHONY: pot
pot: ## Make translation template files
	mkdir -p $(OUTPUT) $(CACHE)
	$(SPHINXBUILD) -M gettext "$(SOURCE)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)
	mv $(BUILDDIR)/gettext/*.pot ../locale/pot

# Catch-all target: route all unknown targets to Sphinx using the new
# "make mode" option.  $(O) is meant as a shortcut for $(SPHINXOPTS).
# %: Makefile
# 	mkdir -p $(OUTPUT) $(CACHE)
# 	$(SPHINXBUILD) -M $@ "$(SOURCE)" "$(BUILDDIR)" $(SPHINXOPTS) $(O)

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

PHONY: latex
latex: ## to make en LaTeX files, you can set PAPER=a4 or PAPER=letter
	mkdir -p $(CACHE) $(OUTPUT)/en/.build/latex
	$(SPHINXBUILD) -b latex $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/en/.build/latex
	@echo
	@echo "Build finished; now you can run 'make' in $(OUTPUT)/en/.build/latex."

PHONY: pdf
pdf: ## to make en PDF, you can set PAPER=a4 or PAPER=letter
	mkdir -p $(CACHE) $(OUTPUT)/en/.build/latex
	$(SPHINXBUILD) -b latex $(ALLSPHINXOPTS) $(SOURCE) $(OUTPUT)/en/.build/latex && cd $(OUTPUT)/$(LANG)/.build/latex && make && cp *.pdf ../../
	@echo "Build finished; pdf file is in $(OUTPUT)/en/"

PHONY: pdf-translation
pdf-translation: ## to make translated LaTeX files, you can set PAPER=a4 or PAPER=letter. Do `LANG=de make pdf-translation` replacing de with language of your choosing
	mkdir -p $(CACHE) $(OUTPUT)/$(LANG)/.build/latex
	$(SPHINXBUILD) -b latex $(ALLSPHINXOPTS) -D language=$(LANG) $(SOURCE) $(OUTPUT)/$(LANG)/.build/latex && cd $(OUTPUT)/$(LANG)/.build/latex && make && cp *.pdf ../../ 

	@echo
	@echo "Build finished; pdf is located in $(OUTPUT)/$(LANG)"

.PHONY: changes
changes: ## to make an overview over all changed/added/deprecated items"
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

