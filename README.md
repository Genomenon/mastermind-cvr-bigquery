# Genomenon CVR BigQuery VCF Annotation Pipeline

## Dependencies

The following dependencies are required to annotate your BigQuery
variant data with the Mastermind Cited Variants Reference public
dataset.

To install these dependencies on a Mac, you can use [Homebrew](https://brew.sh/).

These are needed to run Google Cloud queries from the command line:
- [Google Cloud Platform Billing Account](https://cloud.google.com/billing/docs/how-to/manage-billing-account)
- [Google Cloud SDK](https://cloud.google.com/sdk/install)
    ```
    brew tap caskroom/cask
    brew cask install google-cloud-sdk
    ```

## Running the Pipeline

If you already have variant data in BigQuery tables, you can skip to
[step 2](#2-annotating-your-data-with-the-mastermind-cvr).

1. [Setting up BigQuery and importing your data](#1-setting-up-bigquery-and-importing-your-data)
2. [Annotating your data with the Mastermind CVR](#2-annotating-your-data-with-the-mastermind-cvr)
3. [Exporting your annotated data](#3-exporting-your-annotated-data)

Commands are based on [these instructions](https://cloud.google.com/genomics/docs/how-tos/load-variants).

Use the `-h` flag to print help text for any of the commands.

All arguments that specify BigQuery tables should include both the
dataset and table name, separated by a period. For example, if your
dataset is `dataset_1` and your table is `table_1`, the argument would
be `dataset_1.table_1`.

### 1. Setting up BigQuery and importing your data

1. Create a new project: `./create-project [project-id] [billing-id]`
    1. List billing ids: `./list-billing-ids`
1. Set active project: `./set-active-project [project-id]`.
1. Create a dataset: `./create-dataset [dataset-name]`
1. Create a bucket: `./create-bucket [bucket-name]`
1. Upload VCF to bucket
    1. Local file: `./upload-to-bucket [bucket-name] [path-to-file]`
    1. From URL: `./upload-url-to-bucket [bucket-name] [URL]`
1. Convert VCF to BigQuery table: `./vcf-to-bq [bucket-name] [bucket-vcf-path] [VCF-table]`
    1. Wait for task to finish `./watch-task [task-id]`

### 2. Annotating your data with the Mastermind CVR

1. Set active project:
    ```
    ./set-active-project [project-id]
    ```
1. Annotate VCF BigQuery table with CVR:
    ```
    ./annotate-vcf [VCF-table] [output-table] [assembly-version] [reference-name-type]
    ```
    Example:
    ```
    ./annotate-vcf my_dataset.my_table my_dataset.my_annotated_table GRCh37 chr
    ```

- `[VCF-table]`: The input dataset table you want to annotate.

   To list project datasets: `bq ls`

   Then to list dataset tables: `bq ls [dataset]`
- `[output-table]`: The output dataset table
  you want to create with the annotated variants
- `[assembly-version]`: The assembly version your
  variants are using, either `GRCh37` or `GRCh38`.
- `[reference-name-type]`: The type of data defined in the
  `reference_name` of your input dataset table imported from your VCF
  data. This will be the same as in the original VCF file's `#CHROM`
  column, which is one of the following data types:
    - `contig`: For example, `NC_000014.9`
    - `chr`: For example, `14`
    - `chr_prefix`: For example, `chr14`

You can also list the available Genomenon Mastermind CVR
public datasets available from which to annotate your data. These
consist of a GRCh37 and GRCh38 version for each date the CVR was
released as a BigQuery public dataset:
```
./list-cvr-tables -v [assembly-version]
```

### 3. Exporting your annotated data

This is only needed if you want to export your annotated variant data
from BigQuery to an annotated VCF file.

You will need
[bcftools](https://samtools.github.io/bcftools/bcftools.html) to run
this, which can be installed on Mac with Homebrew:
```
brew install bcftools
```

1. Generate representative header:
    ```
    bcftools merge [vcf-file] [cvr-file] --print-header -O z -o [header.vcf.gz]
    ```
1. Upload header file to bucket:
    ```
    ./upload-to-bucket [bucket-name] [path-to-header-file]
    ```
1. Convert table to VCF:
    ```
    ./bq-to-vcf [bucket-name] [annotated-table] [header-bucket-path]
    ```

