## Introduction

Welcome to MONAI Model Zoo! We're excited you're here and want to contribute. This documentation is intended for individuals and institutions interested in contributing medical imaging models to MONAI Model Zoo.

## Preparing a bundle

Please refer to [MONAI Bundle Specification](https://docs.monai.io/en/latest/mb_specification.html) and the description of bundle [config syntax](https://docs.monai.io/en/latest/config_syntax.html) to prepare your bundle.

The [get started](https://github.com/Project-MONAI/tutorials/blob/main/bundle/get_started.md) notebook is a step-by-step tutorial to help developers easily get started to develop a bundle. And [bundle examples](https://github.com/Project-MONAI/tutorials/tree/main/bundle) show the typical bundle for 3D segmentation, how to use customized components in a bundle, and how to parse bundle in your own program as "hybrid" mode, etc.

As for the path related varibles within config files (such as "bundle_root"), we suggest to use path that do not include personal information (such as `"/home/your_name/"`).The following is an example of path using:

`"bundle_root": "/workspace/data/<bundle name>"`.

### Readme Format

A [template](https://github.com/Project-MONAI/model-zoo/blob/dev/docs/readme_template.md) on how to prepare a readme file is provided.

### License Format

As described in [README](https://github.com/Project-MONAI/model-zoo#readme), please include `LICENSE` into the root directory of the bundle, and please also include `docs/data_license.txt` if there are any license conditions stated for data your bundle uses. Here is an example of [the brats_mri_segmentation bundle](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/docs/data_license.txt).

### Model naming

The name of a bundle is suggested to contain its characteristics, such as include the task type and data type. The following are some of the examples:

```
spleen_deepedit_annotation
spleen_ct_segmentation
chest_xray_classification
pathology_metastasis_detection
```

### Model storage

Github limits the size of files allowed in the repository (see [About size limits on GitHub](https://docs.github.com/en/repositories/working-with-files/managing-large-files/about-large-files-on-github)). Therefore, MONAI Model Zoo limits each single file to be no larger than **25MB**.

### Prepare a config file for large files

If a bundle has large files, please upload those files into a publicly accessible source, and provide a config file called `large_files.yml` (or `.yaml`, `.json`) that contains the corresponding download links. During the pull request, only the config file should be included (large files should be excluded). Please put the config file within the root directory of the bundle, and it should contain the following information:

1. `path`, relative path of the large file in the bundle.
2. `url`, URL link that can download the file.
3. `hash_val`, (**optional**) expected hash value of the file.
4. `hash_type`, (**optional**) hash type. Supprted hash type includes "md5", "sha1", "sha256" and "sha512".

The template is as follow, and you can also click [here](https://github.com/Project-MONAI/model-zoo/blob/dev/models/spleen_ct_segmentation/large_files.yml) to see an actual example of `spleen_ct_segmentation`:

```
large_files:
  - path: "models/model.pt"
    url: "url-of-model.pt"
    hash_val: ""
    hash_type: ""
  - path: "models/model.ts"
    url: "url-of-model.ts"
```

### Preferred Files and Keys

In order to be compatible with other apps such as MONAI FL, MONAI deploy and MONAI Label, except the format requirements from MONAI Bundle Specification, a bundle in MONAI Model Zoo should also contain the necessary files `LICENSE`, `configs/metadata.json` ([click here for instance](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/configs/metadata.json)), and the following preferred files:

1. `models/model.pt` (or a download link in the config file for large files, [click here for instance](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/large_files.yml))
1. `configs/inference.json` (or `.yaml`, `.json`, [click here for instance](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/configs/inference.json))

If your bundle does not have any of the preferred files, please add the bundle name into `exclude_verify_preferred_files_list` in `ci/bundle_custom_data.py`.

Except the requirements of files, there are also some requirements of keys within config files:

In inference config file (if exists), please include the following keys: `bundle_root` (root directory of your bundle), `device` (required device), `network_def` (definition of the network components), `inferer`, and if there are any output files, the directory key should be defined as `output_dir` ([click here for instance](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/configs/inference.json)).

In train config file (if exists), please follow the following requirements in order to maintain consistent naming format ([click here for instance](https://github.com/Project-MONAI/model-zoo/blob/dev/models/brats_mri_segmentation/configs/train.json)):

1. Please include keys `bundle_root`, `device` and `dataset_dir`, and if there are any output files, the directory key should be defined as `output_dir`.
1. If having `train`, components `train#trainer`, `train#trainer#max_epochs`, `train#dataset`, `train#dataset#data`, `train#handlers` should be defined.
1. If having `validate`, components `validate#evaluator`, `validate#handlers`, `validate#dataset`, `validate#dataset#data` should be defined.
1. In `train` and/or `validate`, please define `preprocessing`, `postprocessing`, `inferer` and `key_metric` if they are used.
1. If `ValidationHandler` is used, please define the key `val_interval` and use it for the argument `interval`.


## Verifying the bundle

We prepared several premerge CI tests to verify your bundle.

### Necessary verifications

1. Check if necessary files are existing in your bundle.
1. Check if keys naming are consistent with our requirements.
1. If an existing bundle has been modified, check if `version` and `changelog` are updated.
1. Check if metadata format is correct. You can also run the following command locally to verify your bundle before submitting a pull request:

```bash
python -m monai.bundle verify_metadata --meta_file configs/metadata.json --filepath eval/schema.json
```

### Optional verifications

#### Verify data shape and data type
Check if the input and output data shape and data type of network defined in the metadata are correct. You can also run the following command locally to verify your bundle before submitting a pull request.

```bash
python -m monai.bundle verify_net_in_out --net_id network_def --meta_file configs/metadata.json --config_file configs/inference.json
```

`net_id` is the ID name of the network component, `config_file` is the filepath (within the bundle) of the config file to get the network definition. Please modify the default values if needed.

If this test is not suitable for your bundle, please add your bundle name into `exclude_verify_shape_list` in `ci/bundle_custom_data.py`.

#### Verify torchscript
Check the functionality of exporting the checkpoint to TorchScript file. You can also run the following command locally to verify your bundle before submitting a pull request.

```bash
python -m monai.bundle ckpt_export --net_id network_def --filepath models/verify_model.ts --ckpt_file models/model.pt --meta_file configs/metadata.json --config_file configs/inference.json
```

If your bundle does not support TorchScript, please mention it in `docs/README.md`, and add your bundle name into `exclude_verify_torchscript_list` in `ci/bundle_custom_data.py`.


## Checking the coding style

After verifying your bundle, if there are any `.py` files, coding style is checked and enforced by flake8, black, isort, pytype and mypy.
Before submitting a pull request, we recommend that all checks should pass, by running the following command locally:

```bash
# optionally update the dependencies and dev tools
python -m pip install -U pip
python -m pip install -U -r requirements-dev.txt

# run the linting and type checking tools
./runtests.sh --codeformat

# try to fix the coding style errors automatically
./runtests.sh --autofix
```

## Submitting pull requests

All code changes to the dev branch must be done via [pull requests](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/proposing-changes-to-your-work-with-pull-requests).
1. Please create a new ticket from [the issue list][monai model zoo issue list].
1. [create a new branch in your fork](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request-from-a-fork)
of the codebase named `[ticket_id]-[task_name]`.
Ideally, the new branch should be based on the latest `dev` branch.
1. Make changes to the branch ([use detailed commit messages if possible](https://chris.beams.io/posts/git-commit/)).
1. [Create a new pull request](https://help.github.com/en/desktop/contributing-to-projects/creating-a-pull-request) from the task branch to the dev branch, with detailed descriptions of the purpose of this pull request.
1. For any large files inside the bundle, please exclude them and provide the download links instead. Please follow the instructions mentioned above to prepare the necessary `large_files.yml`.
1. Wait for reviews; if there are reviews, make point-to-point responses, make further code changes if needed.
1. If there are conflicts between the pull request branch and the dev branch, pull the changes from the dev and resolve the conflicts locally.
1. Reviewer and contributor may have discussions back and forth until all comments addressed.
1. Wait for the pull request to be merged.

[monai model zoo issue list]: https://github.com/Project-MONAI/model-zoo/issues

## Validate and release

As for a pull request, a CI program will try to download all large files if mentioned and do several validations. If the pull request is approved and merged, the full bundle (with all large files if exists) will be archived and send to [Releases](https://github.com/Project-MONAI/model-zoo/releases).

## Remove a bundle

If an existing bundle needs to be removed, please follow the steps mentioned above to do the delete manipulations during the created new branch in your fork, and then create a new pull request.
However, bundles that are already stored in the release will not be changed if the corresponding source code is removed.
