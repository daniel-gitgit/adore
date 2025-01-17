include make_gadgets/Makefile

SHELL:=/bin/bash

.DEFAULT_GOAL := all

ROOT_DIR:=$(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

MAKEFLAGS += --no-print-directory

.EXPORT_ALL_VARIABLES:
DOCKER_BUILDKIT?=1
DOCKER_CONFIG?=

#CMAKE_PREFIX_PATH?=$(shell realpath $$(find . -type d | grep "build/install" | grep -v "build/install/") | tr '\n' ';') 
CMAKE_PREFIX_PATH?=$(shell realpath $$(find . -type d | grep "build" | grep -v "build/") | tr '\n' ';') 

LIBADORE_PROJECT="libadore"
LIBADORE_VERSION="latest"
LIBADORE_TAG="${LIBADORE_PROJECT}:${LIBADORE_VERSION}"

.PHONY: build_libadore  
build_libadore: set_libadore_env build

.PHONY: set_libadore_env 
set_libadore_env: 
	$(eval PROJECT := ${LIBADORE_PROJECT}) 
	$(eval TAG := ${LIBADORE_TAG})

.PHONY: all 
all: root_check docker_group_check build_external build_libadore

.PHONY: build 
build: all## build libadore

.PHONY: clean_libadore 
clean_libadore:
	rm -rf "${ROOT_DIR}/libadore/build"

.PHONY: build 
build:
	rm -rf ${PROJECT}/build
	docker compose build libadore_build
	docker compose build libadore
	docker compose build libadore_test
	docker cp $$(docker create --rm $(shell echo ${TAG} | tr A-Z a-z)):/tmp/${PROJECT}/${PROJECT}/build ${ROOT_DIR}/${PROJECT}

.PHONY: test 
test:## run libadore unit tests
	@if [[ `docker images -q ${LIBADORE_TAG} 2> /dev/null` == "" ]]; then \
            echo "ERROR: Docker image: ${LIBADORE_TAG} does not exist. Run make build on libadore and try again." 1>&2; \
            exit 1; \
        fi;

	docker run --rm libadore_test:latest /bin/bash -c 'cd build && ctest' 2>&1 | tee -a libadore/build/ctest.log
	@echo "ctest log can be found at libadore/build/ctest.log"

	@if grep -q "Errors while running CTest" "libadore/build/ctest.log"; then \
            exit 1; \
        fi;

.PHONY: build_external 
build_external: ## builds all libadore external libraries 
	cd "${ROOT_DIR}/libadore/external/" && make

.PHONY: clean_external 
clean_external:
	cd "${ROOT_DIR}/libadore/external/" && make clean

.PHONY: clean 
clean: clean_external clean_libadore
	docker rm $$(docker ps -a -q --filter "ancestor=${LIBADORE_TAG}") 2> /dev/null || true
	docker rmi $$(docker images -q ${LIBADORE_TAG}) 2> /dev/null || true

.PHONY: clean_docker 
clean_docker:
	docker rmi $$(docker images --filter "dangling=true" -q) --force
	docker rmi ${LIBADORE_TAG} --force

.PHONY: lint 
lint: ## returns a linting report for libadore
	cd cpplint && \
        make lint CPP_PROJECT_DIRECTORY=$(realpath ${ROOT_DIR}/libadore/adore)

.PHONY: lintfix 
lintfix: ## Automated source formatting/lint fixing using clang-format 
	cd cpplint && \
        make lintfix CPP_PROJECT_DIRECTORY=$(realpath ${ROOT_DIR}/libadore/adore)

.PHONY: lintfix_simulate 
lintfix_simulate:
	cd cpplint && \
        make lintfix_simulate CPP_PROJECT_DIRECTORY=$(realpath ${ROOT_DIR}/libadore/adore)

.PHONY: cppcheck
cppcheck: ## Runs cppcheck static analysis tool on libadore
	cd cppcheck && \
        make cppcheck CPP_PROJECT_DIRECTORY=$(realpath ${ROOT_DIR}/libadore/adore)

.PHONY: lizard
lizard: ## generates a lizard static analysis report on libadore
	cd lizard && \
        make lizard CPP_PROJECT_DIRECTORY=$(realpath ${ROOT_DIR}/libadore/adore)
