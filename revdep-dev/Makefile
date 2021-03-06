all: test

TESTTHAT_REPORTER ?= 'check'

REVDEP := RSQLite RKazam
TEST_TARGETS := $(patsubst %,test-%,${REVDEP})

LIB_DIR := $(shell Rscript -e "cat(.libPaths()[1])")

${LIB_DIR}/DBItest/DESCRIPTION: ../R/*.R ../DESCRIPTION
	R CMD INSTALL ..

install: ${LIB_DIR}/DBItest/DESCRIPTION

RKazam RSQLite RPostgres RMySQL:
	# Use local checkout if available, otherwise clone from rstats-db GitHub
	if [ -d ../../$@ ]; then ln -s $(realpath ../../$@) .; else git clone https://github.com/rstats-db/$@.git; Rscript -e "devtools::install_deps('$@', dependencies = TRUE)"; fi

bigrquery:
	# Use local checkout if available, otherwise clone from GitHub
	if [ -d ../../$@ ]; then ln -s $(realpath ../../$@) .; else git clone https://github.com/krlmlr/$@.git -b feature/dbi-test; Rscript -e "devtools::install_deps('$@', dependencies = TRUE)"; fi

test: ${TEST_TARGETS}

safe-load-%:
	R CMD INSTALL '$*' && Rscript --vanilla -e "devtools::load_all('$*')"

# Necessary because of https://github.com/hadley/devtools/issues/1070
load-%:
	make --quiet -j1 safe-load-$*

test-%: install load-% %
	Rscript --vanilla -e "options(crayon.enabled=TRUE, deparse.max.lines=2); devtools::test('$*', filter = 'DBItest', reporter = ${TESTTHAT_REPORTER})"

clean:
	rm -rf ${REVDEP}
