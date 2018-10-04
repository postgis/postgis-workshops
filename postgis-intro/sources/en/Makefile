# Makefile for Sphinx documentation
#

# You can set these variables from the command line.
SPHINXOPTS    =
SPHINXBUILD   = sphinx-build
PAPER         =

# Internal variables.
DOC             = ../../doc/en
PAPEROPT_a4     = -D latex_paper_size=a4
PAPEROPT_letter = -D latex_paper_size=letter
ALLSPHINXOPTS   = -d $(DOC)/.build/doctrees $(PAPEROPT_$(PAPER)) $(SPHINXOPTS) .

.PHONY: help clean html web pickle htmlhelp latex changes linkcheck

help:
	@echo "Please use \`make <target>' where <target> is one of"
	@echo "  html      to make standalone HTML files"
	@echo "  pickle    to make pickle files (usable by e.g. sphinx-web)"
	@echo "  htmlhelp  to make HTML files and a HTML help project"
	@echo "  latex     to make LaTeX files, you can set PAPER=a4 or PAPER=letter"
	@echo "  changes   to make an overview over all changed/added/deprecated items"
	@echo "  linkcheck to check all external links for integrity"

clean:
	@rm -rf $(DOC)

html:
	mkdir -p ../../doc $(DOC) $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b html $(ALLSPHINXOPTS) $(DOC)
	@echo
	@echo "Build finished. The HTML pages are in $(DOC)."

#	find $(DOC) -name .svn -type d -exec /bin/rm -rf {} ';'

pickle:
	mkdir -p $(DOC)/.build/pickle $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b pickle $(ALLSPHINXOPTS) $(DOC)/.build/pickle
	@echo
	@echo "Build finished; now you can process the pickle files or run"
	@echo "  sphinx-web $(DOC)/.build/pickle"
	@echo "to start the sphinx-web server."

web: pickle

htmlhelp:
	mkdir -p $(DOC)/.build/htmlhelp $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b htmlhelp $(ALLSPHINXOPTS) $(DOC)/.build/htmlhelp
	@echo
	@echo "Build finished; now you can run HTML Help Workshop with the" \
	      ".hhp project file in $(DOC)/.build/htmlhelp."

latex:
	mkdir -p $(DOC)/.build/latex $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b latex $(ALLSPHINXOPTS) $(DOC)/.build/latex
	@echo
	@echo "Build finished; the LaTeX files are in $(DOC)/.build/latex."
	@echo "Run \`make all-pdf' or \`make all-ps' in that directory to" \
	      "run these through (pdf)latex."
	
#sed -i bak 's/includegraphics/includegraphics\[scale=0.45\]/g' "$(DOC)/.build/latex/PostGISIntro.tex"

changes:
	mkdir -p $(DOC)/.build/changes $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b changes $(ALLSPHINXOPTS) $(DOC)/.build/changes
	@echo
	@echo "The overview file is in $(DOC)/.build/changes."

linkcheck:
	mkdir -p $(DOC)/.build/linkcheck $(DOC)/.build/doctrees
	$(SPHINXBUILD) -b linkcheck $(ALLSPHINXOPTS) $(DOC)/.build/linkcheck
	@echo
	@echo "Link check complete; look for any errors in the above output " \
	      "or in $(DOC)/.build/linkcheck/output.txt."
