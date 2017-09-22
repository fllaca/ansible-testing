export ROLE_NAME=ansible-role-web-server

define run_playbook
	docker-compose -f tests/docker-compose.yaml run --rm \
		-e ANSIBLE_HOST_KEY_CHECKING=False \
		-e ROLE_NAME=$(ROLE_NAME) \
		ansible-tester \
		ansible-playbook -i /$(ROLE_NAME)/tests/inventory \
		/$(ROLE_NAME)/tests/$(1).yaml
endef

.PHONY: setup
setup:
	docker-compose -f tests/docker-compose.yaml up -d 

.PHONY: converge
converge:
	$(call run_playbook,converge)

.PHONY: verify
verify:
	$(call run_playbook,verify)

.PHONY: test
test: setup converge verify 

.PHONY: clean
clean:
	docker-compose -f tests/docker-compose.yaml down