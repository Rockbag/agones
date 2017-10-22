# Copyright 2017 Google Inc. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

#
# Makefile for building, testing and developing Agon
#

#  __     __         _       _     _
#  \ \   / /_ _ _ __(_) __ _| |__ | | ___ ___
#   \ \ / / _` | '__| |/ _` | '_ \| |/ _ \ __|
#    \ V / (_| | |  | | (_| | |_) | |  __\__ \
#     \_/ \__,_|_|  |_|\__,_|_.__/|_|\___|___/
#

NAME = agon-build
VERSION = 0.1
TAG = $(NAME):$(VERSION)
CLUSTER_NAME = test-cluster
KUBECONFIG = $(build_path)/.kube

# Directory that this Makefile is in.
mkfile_path := $(abspath $(lastword $(MAKEFILE_LIST)))
build_path := $(dir $(mkfile_path))
agon_path := $(realpath $(build_path)/..)

mount_path = /go/src/github.com/agonio/agon
common_mounts = -v $(build_path)/.config/gcloud:/root/.config/gcloud \
				-v $(KUBECONFIG):/root/.kube \
				-v $(agon_path):$(mount_path)

#   _____                    _
#  |_   _|_ _ _ __ __ _  ___| |_ ___
#    | |/ _` | '__/ _` |/ _ \ __/ __|
#    | | (_| | | | (_| |  __/ |_\__ \
#    |_|\__,_|_|  \__, |\___|\__|___/
#                 |___/

# Run a bash shell with the developer tools in it. (Creates the image if it doesn't exist)
# Can use ARGS for extra arguments.
shell: ensure-image
	docker run -it --rm \
		$(common_mounts) \
		-w $(mount_path) \
		$(ARGS) \
		--entrypoint=bash $(TAG) -l

.PHONY: build-image

# Creates the build docker image
build-image:
	docker build --tag=$(TAG) $(build_path)/build-image

# Deletes the local build docker image
clean-image:
	docker rmi $(TAG)

ensure-config:
	-mkdir -p $(build_path)/.kube
	-mkdir -p $(build_path)/.config/gcloud

ensure-image: ensure-config
	@if [ -z $$(docker images -q $(TAG)) ]; then\
		echo "Could not find $(TAG) image. Building...";\
		$(MAKE) build-image;\
	fi

# Initialise the gcloud login and project configuration, if you are working with GCP
gcloud-init: ensure-config
	docker run --rm -it \
		$(common_mounts) \
		--entrypoint="gcloud" $(TAG) init

# Creates and authenticates a small, 3 node GKE cluster to work against
gcloud-test-cluster:
	docker run --rm -it $(common_mounts) \
		--entrypoint="gcloud" $(TAG) \
		deployment-manager deployments create test-cluster --config=$(mount_path)/build/gke-test-cluster/deployment.yml
	$(MAKE) gcloud-auth-cluster

# Pulls down authentication information for kubectl against a cluster, name can be specified through CLUSTER_NAME
# (defaults to 'test-cluster')
gcloud-auth-cluster:
	docker run --rm $(common_mounts) --entrypoint="gcloud" $(TAG) config set container/cluster $(CLUSTER_NAME)
	docker run --rm $(common_mounts) --entrypoint="gcloud" $(TAG) config set compute/zone \
		`grep zone: $(build_path)/gke-test-cluster/deployment.yml | sed 's/zone: //'`
	docker run --rm $(common_mounts) --entrypoint="gcloud" $(TAG) container clusters get-credentials $(CLUSTER_NAME)

# Clean the kubernetes and gcloud configuration
clean-config:
	-sudo rm -r $(build_path)/.kube
	-sudo rm -r $(build_path)/.config