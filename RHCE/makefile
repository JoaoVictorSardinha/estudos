.PHONY: help check-start-env attach-iso start-vms start-ansible provision shutdown-vms destroy-vm destroy-vms force-destroy force-destroy-vms status ansible-ping ansible-play

help: ## Show help
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}' $(MAKEFILE_LIST)

# SHELL := /usr/bin/zsh
SHELL := /bin/bash

.DEFAULT_GOAL := help

RED = \033[1;31m
GREEN = \033[1;32m
RESET = \033[0m

VMS_WITH_ANSIBLE := ansible servera serverb serverc serverd servere
VMS := servera serverb serverc serverd servere
LIBVIRT_DOMAINS := $(addprefix RHCE_,$(VMS_WITH_ANSIBLE))
INVENTORY := inventory
PLAYBOOK := playbooks/master.yml
VAGRANT := LIBVIRT_DEFAULT_URI=qemu:///system VAGRANT_DEFAULT_PROVIDER=libvirt vagrant
VIRSH := virsh -c qemu:///system
ANSIBLE_DOMAIN := RHCE_ansible
ISO_TARGET_DEV := hdc
REQUIRED_START_ENV := ISO
CALLING_USER_HOME := $(shell getent passwd "$${SUDO_USER:-$$(id -un)}" | cut -d: -f6)

define zshrc_var
$(shell /usr/bin/zsh -fc 'source "$(CALLING_USER_HOME)/.zshrc" >/dev/null 2>&1; print -r -- "$$$(1)"' 2>/dev/null)
endef

ifndef ISO
ISO := $(call zshrc_var,ISO)
endif
ifndef RH_USER
RH_USER := $(call zshrc_var,RH_USER)
endif
ifndef RH_TOKEN
RH_TOKEN := $(call zshrc_var,RH_TOKEN)
endif
ifndef RHSM_ORG
RHSM_ORG := $(call zshrc_var,RHSM_ORG)
endif
ifndef RHSM_ACTIVATION_KEY
RHSM_ACTIVATION_KEY := $(call zshrc_var,RHSM_ACTIVATION_KEY)
endif

export ISO RH_USER RH_TOKEN RHSM_ORG RHSM_ACTIVATION_KEY

# VARIABLE="some-value"

# hello: ## Show ${VARIABLE} value
# 	@echo ${VARIABLE}

# shell: hello ## Calls hello task and Print $SHELL value
# 	@echo ${SHELL}

check-start-env:
	@for var in $(REQUIRED_START_ENV); do \
		if [ -z "$${!var}" ]; then \
			printf "$(RED)Missing required environment variable: %s$(RESET)\n" "$$var"; \
			echo "Usage: ISO=/path/to/rhel.iso make start-vms"; \
			echo "When running with sudo, this makefile also tries to load exported variables from the calling user's ~/.zshrc."; \
			echo "Optional RHSM registration: ISO=/path/to/rhel.iso RH_USER=user RH_TOKEN=token make start-vms"; \
			echo "Optional activation key registration: ISO=/path/to/rhel.iso RHSM_ORG=org RHSM_ACTIVATION_KEY=key make start-vms"; \
			exit 1; \
		fi; \
	done; \
	if [ ! -f "$$ISO" ]; then \
		printf "$(RED)ISO file not found: %s$(RESET)\n" "$$ISO"; \
		exit 1; \
	fi

attach-iso: check-start-env
	@echo "Attaching ISO to $(ANSIBLE_DOMAIN)..."
	@if $(VIRSH) domblklist $(ANSIBLE_DOMAIN) --inactive | awk '{print $$3}' | grep -qx "$(ISO_TARGET_DEV)"; then \
		$(VIRSH) change-media $(ANSIBLE_DOMAIN) $(ISO_TARGET_DEV) "$$ISO" --insert --force --config; \
	else \
		$(VIRSH) attach-disk $(ANSIBLE_DOMAIN) "$$ISO" $(ISO_TARGET_DEV) --targetbus ide --type cdrom --mode readonly --config; \
	fi

start-vms: check-start-env ## Start all lab VMs with Vagrant (libvirt provider)
	@echo "Starting Vagrant environment..."
	@$(VAGRANT) up --no-provision
	@$(MAKE) attach-iso
	@$(VAGRANT) reload ansible --no-provision
	@$(VAGRANT) provision ansible
	@$(VAGRANT) provision $(VMS)

start-ansible: ## Start only the Ansible control node
	@echo "Starting Ansible control node..."
	@$(VAGRANT) up ansible

provision: ## Re-run provisioning on the Ansible control node
	@echo "Re-provisioning all lab VMs..."
	@$(VAGRANT) provision ansible
	@$(VAGRANT) provision $(VMS)

shutdown-vms: ## Gracefully stop all lab VMs
	@echo "Stopping Vagrant environment..."
	@$(VAGRANT) halt

destroy-vm: destroy-vms ## Alias for destroy-vms

destroy-vms: ## Destroy all lab VMs
	@printf "$(RED)Destroying Vagrant environment...$(RESET)\n"
	@$(VAGRANT) destroy -f

force-destroy: force-destroy-vms ## Alias for force-destroy-vms

force-destroy-vms: ## Force destroy lab VMs directly with virsh when Vagrant state is broken
	@if ! $(VIRSH) list --all >/dev/null 2>&1; then \
		printf "$(RED)Cannot access qemu:///system with the current user.$(RESET)\n"; \
		echo "Add the user to the libvirt group and start a new login session, or run: newgrp libvirt"; \
		echo "Fallback: sudo make force-destroy"; \
		exit 1; \
	fi
	@printf "$(RED)Force destroying libvirt domains...$(RESET)\n"
	@for domain in $(LIBVIRT_DOMAINS); do \
		if $(VIRSH) dominfo "$$domain" >/dev/null 2>&1; then \
			printf "Destroying %s...\n" "$$domain"; \
			$(VIRSH) destroy "$$domain" >/dev/null 2>&1 || true; \
			if [ "$$domain" = "$(ANSIBLE_DOMAIN)" ]; then \
				$(VIRSH) detach-disk "$$domain" "$(ISO_TARGET_DEV)" --config >/dev/null 2>&1 || true; \
			fi; \
			$(VIRSH) undefine "$$domain" --remove-all-storage --nvram >/dev/null 2>&1 || \
				$(VIRSH) undefine "$$domain" --remove-all-storage >/dev/null 2>&1 || \
				$(VIRSH) undefine "$$domain" >/dev/null 2>&1 || true; \
		fi; \
	done
	@rm -rf .vagrant/machines
	@printf "$(GREEN)Forced cleanup complete.$(RESET)\n"

status: ## Show Vagrant status for the lab
	@$(VAGRANT) status

ssh-%: ## SSH into a VM using Vagrant (usage: make ssh-servera)
	@$(VAGRANT) ssh $*

ansible-ping: ## Run ansible ping against all hosts
	ansible -i $(INVENTORY) all -m ping

ansible-play: ## Run main Ansible playbook
	ansible-playbook -i $(INVENTORY) $(PLAYBOOK)
