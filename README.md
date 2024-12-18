# MInAS

This repository holds the combined base MIxS schema, plus the various extensions generated in the scope of the MInAS project.

The source YAML file is generated using the `yq` tool, merging each of the individual YAML files into a single one.

This merging happens once per extension release, and the resulting file is then used to generate the JSON schema and TSV using LinkML tools and scripts.

The resulting output files are stored in the `src/` directory in file format specific subdirectories.

## Schema Generation Instructions

### Required tools and dependencies

The following tools are required to generate the schema files:

- [`yq`](https://github.com/mikefarah/yq/?tab=readme-ov-file#install) (version 4.44.2)
  - Note: version not on `conda` or `pip` requires binary or OS distribution installation
- [`linkml`](https://github.com/linkml/linkml) (Version 1.8.1)
  - Available on pip: `pip install linkml==1.8.1`

### Merging the YAML files

To generate the combined YAML file, we can use a combination of `yq` and `curl` to download specific tagged releases from the MIxS and various MInAS repositories.

For example, updating the versions in the variables

```bash
MIXS_VERSION=6.2.0
EXTANCIENT_VERSION=0.3.2
EXTRADIOCARBONDATING_VERSION=0.1.2

## Core MIxS Schema
curl -o src/mixs/schema/mixs-v$MIXS_VERSION.yaml "https://raw.githubusercontent.com/GenomicsStandardsConsortium/mixs/v$MIXS_VERSION/src/mixs/schema/mixs.yaml" ## Base MIxS schema

## MInAS Extensions
curl -o src/mixs/schema/ancient-v$EXTANCIENT_VERSION.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-ancient/v$EXTANCIENT_VERSION/src/mixs/schema/ancient.yml" ## Ancient DNA extension
curl -o src/mixs/schema/radiocarbon-dating-v$EXTRADIOCARBONDATING_VERSION.yaml "https://raw.githubusercontent.com/MIxS-MInAS/extension-radiocarbon-dating/v$EXTRADIOCARBONDATING_VERSION/src/mixs/schema/radiocarbon-dating.yml" ## Radiocarbon extension

## MInAS Combinations
curl -o src/mixs/schema/minas-combinations-vmain.yaml "https://raw.githubusercontent.com/MIxS-MInAS/minas-combinations/main/src/mixs/schema/minas-combinations.yaml" ## Combinations

## Merge together. Note you need a select(fileIndex == X) for each yaml file!
yq eval-all 'select(fileIndex == 0) *+ select(fileIndex == 1) *+ select(fileIndex == 2) *+ select(fileIndex == 3)' \
  src/mixs/schema/mixs-v$MIXS_VERSION.yaml \
  src/mixs/schema/ancient-v$EXTANCIENT_VERSION.yaml \
  src/mixs/schema/radiocarbon-dating-v$EXTRADIOCARBONDATING_VERSION.yaml \
  src/mixs/schema/minas-combinations-vmain.yaml \
  > src/mixs/schema/mixs-minas.yaml

## Fix some metadata
sed -i 's#source: https://github.com/MIxS-MInAS/extension-radiocarbon-dating/raw/main/proposals/0.1.0/extension-radiocarbon-dating-v0_1_0.csv#source: https://github.com/MIxS-MInAS/MInAS/#g' src/mixs/schema/mixs-minas.yaml
```

Next we lint and validate the newly extended MIxS schema

```bash
linkml lint --validate src/mixs/schema/mixs-minas.yaml
```

We can then use the LinkML package's `gen-json-schema` to generate the JSON schema:

```bash
gen-json-schema src/mixs/schema/mixs-minas.yaml > src/mixs/schema/mixs-minas.json
```

And the python3 script in the `scripts/` directory to generate the TSV files:

```bash
./scripts/linkml2class_tsvs.py --schema-file src/mixs/schema/mixs-minas.yaml --output-dir project/class-model-tsvs/
```

> [!Note]
> This script has been copied and modified very slightly to include the python3 shebang, and is placed under scripts until properly packaged for the MIxS project.
>
> To use this script, you only need python3 and no other dependencies (it seems).
