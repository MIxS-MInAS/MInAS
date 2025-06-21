# MInAS

This repository holds the combined base MIxS schema, plus the various extensions generated in the scope of the MInAS project.

The source YAML file is generated using the `yq` tool, merging each of the individual YAML files into a single one.

This merging happens once per extension release, and the resulting file is then used to generate the JSON schema and TSV using LinkML tools and scripts.

The resulting output files are stored in the `src/` directory in file format specific subdirectories.

## Schema Generation Instructions

### Required tools and dependencies

The following tools are required to generate the schema files:

- [linkml-toolkit](https://github.com/genomewalker/linkml-toolkit)
  - Not yet on pip/conda etc! Will need to manually install
- [`linkml`](https://github.com/linkml/linkml) (Version 1.8.1)
  - Available on pip: `pip install linkml==1.8.1`

Deprecated

- [`yq`](https://github.com/mikefarah/yq/?tab=readme-ov-file#install) (version 4.44.2)
  - Note: version not on `conda` or `pip` requires binary or OS distribution installation

### Merging the YAML files

To generate the combined YAML file, we can use a combination of `yq` and `curl` to download specific tagged releases from the MIxS and various MInAS repositories.

#### Development

For updating during development:

```bash
MIXS_VERSION=6.2.0
EXTANCIENT_VERSION=0.5.0 ## Only used ofr releases
EXTRADIOCARBONDATING_VERSION=0.1.2 ## Only used for releases

## Core MIxS Schema
curl -o src/mixs/schema/mixs-v$MIXS_VERSION.yaml "https://raw.githubusercontent.com/GenomicsStandardsConsortium/mixs/v$MIXS_VERSION/src/mixs/schema/mixs.yaml" ## Base MIxS schema

## MInAS Extensions
curl -o src/mixs/schema/ancient-main.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-ancient/refs/heads/main/src/mixs/schema/ancient.yml" ## Ancient DNA extension
curl -o src/mixs/schema/radiocarbon-dating-main.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-radiocarbon-dating/refs/heads/main/src/mixs/schema/radiocarbon-dating.yml" ## Radiocarbon extension

## MInAS Combinations
curl -o src/mixs/schema/minas-combinations-main.yaml "https://raw.githubusercontent.com/MIxS-MInAS/minas-combinations/main/src/mixs/schema/minas-combinations.yaml" ## Combinations

## Merge together
lmtk combine --mode merge --schema src/mixs/schema/mixs-v$MIXS_VERSION.yaml \
  -a src/mixs/schema/ancient-main.yaml \
  -a src/mixs/schema/radiocarbon-dating-main.yaml \
  -a src/mixs/schema/minas-combinations-main.yaml \
  --output src/mixs/schema/mixs-minas.yaml

## OLD METHOD Merge together. Note you need a select(fileIndex == X) for each yaml file!
## yq eval-all 'select(fileIndex == 0) *+ select(fileIndex == 1) *+ select(fileIndex == 2) *+ select(fileIndex == 3)' \
## src/mixs/schema/mixs-v$MIXS_VERSION.yaml \
## src/mixs/schema/ancient-main.yaml \
##  src/mixs/schema/radiocarbon-dating-main.yaml \
##  src/mixs/schema/minas-combinations-main.yaml \
##  > src/mixs/schema/mixs-minas.yaml
##

## Fix some metadata
sed -i 's#source: https://github.com/MIxS-MInAS/extension-radiocarbon-dating/raw/main/proposals/0.1.0/extension-radiocarbon-dating-v0_1_0.csv#source: https://github.com/MIxS-MInAS/MInAS/#g' src/mixs/schema/mixs-minas.yaml
```

#### Release

And then, for a release, (making sure updating the versions in the variables):

```bash
## Set versions
MIXS_VERSION=6.2.0
EXTANCIENT_VERSION=0.5.0
EXTRADIOCARBONDATING_VERSION=0.1.2
COMBINATIONS_VERSION=0.1.7
```

Download schemas

```bash
## Core MIxS Schema
curl -o src/mixs/schema/mixs-v$MIXS_VERSION.yaml "https://raw.githubusercontent.com/GenomicsStandardsConsortium/mixs/v$MIXS_VERSION/src/mixs/schema/mixs.yaml" ## Base MIxS schema

## MInAS Extensions
curl -o src/mixs/schema/ancient-v$EXTANCIENT_VERSION.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-ancient/v$EXTANCIENT_VERSION/src/mixs/schema/ancient.yml" ## Ancient DNA extension
curl -o src/mixs/schema/radiocarbon-dating-v$EXTRADIOCARBONDATING_VERSION.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-radiocarbon-dating/v$EXTRADIOCARBONDATING_VERSION/src/mixs/schema/radiocarbon-dating.yml" ## Radiocarbon extension
curl -o src/mixs/schema/minas-combinations-v$COMBINATIONS_VERSION.yaml "https://raw.githubusercontent.com/MIxS-MInAS/minas-combinations/refs/tags/v$COMBINATIONS_VERSION/src/mixs/schema/minas-combinations.yml" ## Combinations
```

Merge together with `linkml-toolkit`

```bash
## Merge together
lmtk combine --mode merge --schema src/mixs/schema/mixs-v$MIXS_VERSION.yaml \
  -a src/mixs/schema/ancient-v$EXTANCIENT_VERSION.yaml \
  -a src/mixs/schema/radiocarbon-dating-v$EXTRADIOCARBONDATING_VERSION.yaml \
  -a src/mixs/schema/minas-combinations-v$COMBINATIONS_VERSION.yaml \
  --output src/mixs/schema/mixs-minas.yaml

## OLD METHOD Merge together. Note you need a select(fileIndex == X) for each yaml file!
## yq eval-all 'select(fileIndex == 0) *+ select(fileIndex == 1) *+ select(fileIndex == 2) *+ select(fileIndex == 3)' \
## src/mixs/schema/mixs-v$MIXS_VERSION.yaml \
## src/mixs/schema/ancient-main.yaml \
##  src/mixs/schema/radiocarbon-dating-main.yaml \
##  src/mixs/schema/minas-combinations-main.yaml \
##  > src/mixs/schema/mixs-minas.yaml
##

## Fix some metadata
sed -i 's#source: https://github.com/MIxS-MInAS/extension-radiocarbon-dating/raw/main/proposals/0.1.0/extension-radiocarbon-dating-v0_1_0.csv#source: https://github.com/MIxS-MInAS/MInAS/#g' src/mixs/schema/mixs-minas.yaml
```

First we can check that all new YAML files (extensions, combinations) are represented.

```bash
for i in ethics_perm_scope localised_reservoir_offset_sd mims_symbiontassociated_ancient_data; do
  if [[ $(grep "$i" src/mixs/schema/mixs-minas.yaml | wc -l) -ge 2 ]]; then echo "$i: true"; else echo "$i: false"; fi
done
```

> [!WARNING]
> There should be one string per input YAML file, and be aware these strings may change per release.

In both cases of development and release, we lint and validate the newly extended MIxS schema

```bash
linkml lint --validate src/mixs/schema/mixs-minas.yaml
```

We can then use the LinkML package's `gen-json-schema` to generate the JSON schema:

```bash
gen-json-schema src/mixs/schema/mixs-minas.yaml > src/mixs/schema/mixs-minas.json
```

And the python3 script in the `scripts/` directory to generate the TSV files:

```bash
python3 ./scripts/linkml2class_tsvs.py --schema-file src/mixs/schema/mixs-minas.yaml --output-dir project/class-model-tsvs/
```

> [!Note]
> This script has been copied and modified very slightly to include the python3 shebang, and is placed under scripts until properly packaged for the MIxS project.
>
> To use this script, you only need python3 and no other dependencies (it seems).
