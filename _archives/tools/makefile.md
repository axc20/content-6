# Makefile

```
# makefiletutorial.com
# https://spin.atomicobject.com/2021/03/22/makefiles-vs-package-json-scriptscd
define GetCurrentVersion
	$(shell node -p "require('./package.json').$(1)")
endef

version_test := $(call GetCurrentVersion, version)
version := $(strip $(shell node --print "require('./package.json').version"))
branch := $(shell git rev-parse --abbrev-ref HEAD)

demoA:
	@echo $(version_test)
	@echo $(version)
	@echo $(branch)
	@someVar="45"; \
	echo $$someVar

	validate-release-branch:
	@if [[ "$(branch)" != *"feature/release"* ]]; then \
		echo $(branch) is not a release branch; \
		exit 1; \
	fi;

validate-merge-commit:
	git checkout main
	git log -1 --pretty=%B
	@last_commit_log="$(shell git log -1 --pretty=%B)"; \
	base_version="$(shell echo $(version) | sed "s/-.*//")"; \
	[[ "$$last_commit_log" == *"Merge"* ]] && is_merge=true; \
	[[ "$$last_commit_log" == *"feature/release-$(version) to master"* ]] && merge_latest_version=true; \
	[[ "$$last_commit_log" == *"feature/release-$$base_version to master"* ]] && merge_latest_base_version=true; \
	if { [ "$$merge_latest_version" != true ] || [ "$$merge_latest_base_version" != true ]; } && [ "$$is_merge" != true ]; then \
		echo Please check that the release branch for $(version) has been merged to master.; \
		exit 1; \
	fi;

check-jira-ticket:
	@if [ -z $(t) ]; then \
		echo JIRA ticket not provided.; \
		exit 1; \
	fi;

# make release t=DESLANG-1234 c=1 s=1.2.3
# for rc release, c is required
release: validate-release-branch check-jira-ticket
	@if [[ $(branch) =~ [0-9]{4}\.[0-9]{1,2}\.[0-9]{1,2} ]]; then \
		base_version=$${BASH_REMATCH[0]}; \
	else \
		echo $(branch) - No release date detected; \
		exit 1; \
	fi; \
	hash=$(shell git rev-parse HEAD); \
	rc=$(shell echo$(if $(c), -rc.$(c),)); \
	new_version=$$base_version$$rc; \
	[[ -z "$(s)" ]] && styles_version=$$new_version || styles_version="$(s)"; \
	echo "\nJIRA ticket: $(t)"; \
	echo version: $$new_version; \
	echo styles version: $$styles_version; \
	echo "Updating package.json ...\n"; \
	node -e "let pkg=require('./package.json'); \
		pkg.description='$$new_version'; \
		require('fs').writeFileSync('package.json', JSON.stringify(pkg, null, 2));"; \
	sleep 2; \
	echo "npm install"; \
	git add package.json package-lock.json; \
	git commit -m "$(t) Update version number for $$new_version release" --no-verify; \
	if [ -z $$rc ]; then \
		echo "build docs ... git add ... git commit"; \
		echo "Verify docs build with http-server docs_site\n"; \
	fi; \
	echo "Ready to be pushed: please verify changes before pushing (npm run start)\n"; \
	echo "To revert: git reset --hard $$hash\n";

publish:
	@echo Publishing to npm...
	npm config set registry https://registry.npmjs.org
	npm whoami
	@echo Updating publish config registry to npm...
	@node -e "let pkg=require('./package.json'); \
	 	pkg.publishConfig.registry='https://registry.npmjs.org'; \
	 	require('fs').writeFileSync('package.json', JSON.stringify(pkg, null, 2));"
	@sleep 3
	npm install
	npm publish
	npm config set registry https://repo.etrade.com/registry/npm/npm-all/
	npm install
	@echo Discarding changes to package.json and package-lock.json...
	git checkout HEAD -- package.json package-lock.json
	git tag $(version)
	git push origin $(version) --no-verify
```
