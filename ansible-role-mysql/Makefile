

ANSIBLE_HOST_KEY_CHECKING:=False

create:
	docker-compose -f tests/docker-compose.yaml up -d

destroy:
	docker-compose -f tests/docker-compose.yaml down

converge:
	ansible-playbook -i tests/inventory.ini tests/converge.yaml

verify:
	ansible-playbook -i tests/inventory.ini tests/verify.yaml

test: destroy create converge verify

.PHONY: test destroy create converge verify destroy
