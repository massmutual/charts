CHART_REPO := http://jenkins-x-chartmuseum:8080
NAME := jenkins
OS := $(shell uname)
RELEASE_VERSION := $(shell jx-release-version)
HELM := helm

setup:
	$(HELM) init --client-only
	$(HELM) repo add jenkins-x http://chartmuseum.jenkins-x.io
	$(HELM) repo add incubator https://kubernetes-charts-incubator.storage.googleapis.com
	$(HELM) repo add stable https://kubernetes-charts.storage.googleapis.com
	$(HELM) repo add monocular https://helm.github.io/monocular

build: setup clean
	helm dependency build
	$(HELM) lint

lint:
	helm dependency build
	$(HELM) lint

install: clean setup build
	$(HELM) upgrade --debug --install $(NAME) .

apply: clean setup
	jx step helm apply $(NAME) .

upgrade: clean setup build
	$(HELM) upgrade --debug --install $(NAME) .

delete:
	$(HELM) delete --purge $(NAME)

clean:
	rm -rf charts
	rm -rf ${NAME}*.tgz
	rm -rf requirements.lock

release: setup clean build
ifeq ($(OS),Darwin)
	sed -i "" -e "s/version:.*/version: $(RELEASE_VERSION)/" Chart.yaml
else ifeq ($(OS),Linux)
	sed -i -e "s/version:.*/version: $(RELEASE_VERSION)/" Chart.yaml
else
	exit -1
endif
	git add Chart.yaml
	git commit -a -m "release $(RELEASE_VERSION)" --allow-empty
	git tag -fa v$(RELEASE_VERSION) -m "Release version $(RELEASE_VERSION)"
	git push origin v$(RELEASE_VERSION)
	$(HELM) package .
	curl --fail -u $(CHARTMUSEUM_CREDS_USR):$(CHARTMUSEUM_CREDS_PSW) --data-binary "@$(NAME)-$(RELEASE_VERSION).tgz" $(CHART_REPO)/api/charts
	helm repo update
	rm -rf ${NAME}*.tgz
	echo $(RELEASE_VERSION) > VERSION
