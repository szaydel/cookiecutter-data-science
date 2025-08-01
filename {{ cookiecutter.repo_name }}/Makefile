#################################################################################
# GLOBALS                                                                       #
#################################################################################

PROJECT_NAME = {{ cookiecutter.repo_name }}
PYTHON_VERSION = {{ cookiecutter.python_version_number }}
PYTHON_INTERPRETER = python

#################################################################################
# COMMANDS                                                                      #
#################################################################################

{% if cookiecutter.dependency_file != 'none' %}
## Install Python dependencies
.PHONY: requirements
requirements:
	{% if "requirements.txt" == cookiecutter.dependency_file -%}
	{% if "uv" == cookiecutter.environment_manager -%}
	uv pip install -r requirements.txt
	{% else -%}
	$(PYTHON_INTERPRETER) -m pip install -U pip
	$(PYTHON_INTERPRETER) -m pip install -r requirements.txt
	{% endif -%}
	{% elif "pyproject.toml" == cookiecutter.dependency_file -%}
	{% if "uv" == cookiecutter.environment_manager -%}
	uv sync
	{% elif "pixi" == cookiecutter.environment_manager -%}
	pixi install
	{% elif "poetry" == cookiecutter.environment_manager -%}
	poetry install
	{% else -%}
	pip install -e .
	{% endif -%}
	{% elif "environment.yml" == cookiecutter.dependency_file -%}
	conda env update --name $(PROJECT_NAME) --file environment.yml --prune
	{% elif "Pipfile" == cookiecutter.dependency_file -%}
	pipenv install
	{% elif "pixi.toml" == cookiecutter.dependency_file -%}
	pixi install
{% endif %}
{% endif %}


## Delete all compiled Python files
.PHONY: clean
clean:
	find . -type f -name "*.py[co]" -delete
	find . -type d -name "__pycache__" -delete

{% if cookiecutter.linting_and_formatting == 'ruff' %}
## Lint using ruff (use `make format` to do formatting)
.PHONY: lint
lint:
	ruff format --check
	ruff check

## Format source code with ruff
.PHONY: format
format:
	ruff check --fix
	ruff format
{% elif cookiecutter.linting_and_formatting == 'flake8+black+isort' %}
## Lint using flake8, black, and isort (use `make format` to do formatting)
.PHONY: lint
lint:
	flake8 {{ cookiecutter.module_name }}
	isort --check --diff {{ cookiecutter.module_name }}
	black --check {{ cookiecutter.module_name }}

## Format source code with black
.PHONY: format
format:
	isort {{ cookiecutter.module_name }}
	black {{ cookiecutter.module_name }}
{% endif %}

{% if cookiecutter.testing_framework != 'none' %}
## Run tests
.PHONY: test
test:
{%- if cookiecutter.testing_framework == 'unittest' %}
	python -m unittest discover -s tests
{%- elif cookiecutter.testing_framework == 'pytest' %}
	python -m pytest tests
{%- endif -%}
{%- endif -%}

{% if not cookiecutter.dataset_storage.none %}
## Download Data from storage system
.PHONY: sync_data_down
sync_data_down:
	{% if cookiecutter.dataset_storage.s3 -%}
	aws s3 sync s3://{{ cookiecutter.dataset_storage.s3.bucket }}/data/ \
		data/ {% if cookiecutter.dataset_storage.s3.aws_profile != 'default' %} --profile {{ cookiecutter.dataset_storage.s3.aws_profile }}{% endif %}
	{% elif cookiecutter.dataset_storage.azure -%}
	az storage blob download-batch -s {{ cookiecutter.dataset_storage.azure.container }}/data/ \
		-d data/
	{% elif cookiecutter.dataset_storage.gcs -%}
	gsutil -m rsync -r gs://{{ cookiecutter.dataset_storage.gcs.bucket }}/data/ data/
	{% endif %}

## Upload Data to storage system
.PHONY: sync_data_up
sync_data_up:
	{% if cookiecutter.dataset_storage.s3 -%}
	aws s3 sync data/ \
		s3://{{ cookiecutter.dataset_storage.s3.bucket }}/data {% if cookiecutter.dataset_storage.s3.aws_profile != 'default' %} --profile {{ cookiecutter.dataset_storage.s3.aws_profile }}{% endif %}
	{% elif cookiecutter.dataset_storage.azure -%}
	az storage blob upload-batch -d {{ cookiecutter.dataset_storage.azure.container }}/data/ \
		-s data/
	{% elif cookiecutter.dataset_storage.gcs -%}
	gsutil -m rsync -r data/ gs://{{ cookiecutter.dataset_storage.gcs.bucket }}/data/
	{% endif %}
{% endif %}

{% if cookiecutter.environment_manager != 'none' %}
## Set up Python interpreter environment
.PHONY: create_environment
create_environment:
	{% if cookiecutter.environment_manager == 'conda' -%}
	{% if cookiecutter.dependency_file != 'environment.yml' %}
	conda create --name $(PROJECT_NAME) python=$(PYTHON_VERSION) -y
	{% else -%}
	conda env create --name $(PROJECT_NAME) -f environment.yml
	{% endif %}
	@echo ">>> conda env created. Activate with:\nconda activate $(PROJECT_NAME)"
	{% elif cookiecutter.environment_manager == 'virtualenv' -%}
	@bash -c "if [ ! -z `which virtualenvwrapper.sh` ]; then source `which virtualenvwrapper.sh`; mkvirtualenv $(PROJECT_NAME) --python=$(PYTHON_INTERPRETER); else mkvirtualenv.bat $(PROJECT_NAME) --python=$(PYTHON_INTERPRETER); fi"
	@echo ">>> New virtualenv created. Activate with:\nworkon $(PROJECT_NAME)"
	{% elif cookiecutter.environment_manager == 'pipenv' -%}
	pipenv --python $(PYTHON_VERSION)
	@echo ">>> New pipenv created. Activate with:\npipenv shell"
	{% elif cookiecutter.environment_manager == 'uv' -%}
	uv venv --python $(PYTHON_VERSION)
	@echo ">>> New uv virtual environment created. Activate with:"
	@echo ">>> Windows: .\\\\.venv\\\\Scripts\\\\activate"
	@echo ">>> Unix/macOS: source ./.venv/bin/activate"
	{% elif cookiecutter.environment_manager == 'pixi' -%}
	{% if cookiecutter.dependency_file == 'pixi.toml' %}
	@echo ">>> Pixi environment will be created when running 'make requirements'"
	{% else %}
	@echo ">>> Pixi environment configured in pyproject.toml. Run 'make requirements' to install dependencies."
	{% endif %}
	@echo ">>> Activate with:\npixi shell"
	{% elif cookiecutter.environment_manager == 'poetry' -%}
	poetry env use $(PYTHON_VERSION)
	@echo ">>> Poetry environment created. Activate with: "
	@echo '$$(poetry env activate)'
	@echo ">>> Or run commands with:\npoetry run <command>"
{% endif %}
{% endif %}


#################################################################################
# PROJECT RULES                                                                 #
#################################################################################

{% if cookiecutter.include_code_scaffold == 'Yes' %}
## Make dataset
.PHONY: data
data: requirements
	$(PYTHON_INTERPRETER) {{ cookiecutter.module_name }}/dataset.py
{% endif %}

#################################################################################
# Self Documenting Commands                                                     #
#################################################################################

.DEFAULT_GOAL := help

define PRINT_HELP_PYSCRIPT
import re, sys; \
lines = '\n'.join([line for line in sys.stdin]); \
matches = re.findall(r'\n## (.*)\n[\s\S]+?\n([a-zA-Z_-]+):', lines); \
print('Available rules:\n'); \
print('\n'.join(['{:25}{}'.format(*reversed(match)) for match in matches]))
endef
export PRINT_HELP_PYSCRIPT

help:
	@$(PYTHON_INTERPRETER) -c "${PRINT_HELP_PYSCRIPT}" < $(MAKEFILE_LIST)
