# SFTP Bulk

<HideInUI>

This page contains the setup guide and reference information for the [SFTP Bulk](https://docs.airbyte.com/integrations/sources/sftp-bulk#file-specific-configuration) source connector.

</HideInUI>

The SFTP Bulk connector offers several features that are not available in the standard SFTP source connector:

- **Bulk ingestion of files**: This connector can consolidate and process multiple files as a single data stream in your destination system.
- **Incremental loading**: This connector supports incremental loading, allowing you to sync files from the SFTP server to your destination based on their creation or last modification time.
- **Load most recent file**: You can choose to load only the most recent file from the designated folder path. This feature is particularly useful when dealing with snapshot files that are regularly added and contain the latest data.

## Prerequisites

- Access to a remote server that supports SFTP
- Host address
- Valid username and password associated with the host server

## Setup guide

### Set up SFTP Bulk

#### Step 1: Set up SFTP authentication

To set up the SFTP connector, you will need to select at least _one_ of the following authentication methods:

- Your username and password credentials associated with the server.
- A private/public key pair.

To set up key pair authentication, follow these steps:

1. Open your terminal or command prompt and use the `ssh-keygen` command to generate a new key pair.
   :::note
   If your operating system does not support the `ssh-keygen` command, you can use a third-party tool like [PuTTYgen](https://www.puttygen.com/) to generate the key pair instead.
   :::

2. You will be prompted for a location to save the keys, and a passphrase to secure the private key. You can press enter to accept the default location and opt out of a passphrase if desired. Your two keys will be generated in the designated location as two separate files. The private key will usually be saved as `id_rsa`, while the public key will be saved with the `.pub` extension (`id_rsa.pub`).

3. Use the `ssh-copy-id` command in your terminal to copy the public key to the server.

```
ssh-copy-id <username>@<server_ip_address>
```

Be sure to replace your specific values for your username and the server's IP address.
:::note
Depending on factors such as your operating system and the specific SSH implementation your remote server uses, you may not be able to use the `ssh-copy-id` command. If so, please consult your server administrator for the appropriate steps to copy the public key to the server.
:::

4. You should now be able to connect to the server via the private key. You can test this by using the `ssh` command:

```
ssh <username>@<server_ip_address>
```

For more information on SSH key pair authentication, please refer to the
[official documentation](https://www.ssh.com/academy/ssh/keygen).

### Set up the SFTP Bulk connector in Airbyte

### For Airbyte Cloud:

1. [Log into your Airbyte Cloud](https://cloud.airbyte.com/workspaces) account.
2. Click Sources and then click + New source.
3. On the Set up the source page, select SFTP Bulk from the Source type dropdown.
4. Enter a name for the SFTP Bulk connector.
5. Choose a [delivery method](../../platform/using-airbyte/delivery-methods) for your data.
6. Enter the **Host Address**.
7. Enter your **Username**
8. Enter your authentication credentials for the SFTP server (**Password** or **Private Key**). If you are authenticating with a private key, you can upload the file containing the private key (usually named `rsa_id`) using the Upload file button.
9. In the section titled "The list of streams to sync", enter a **Stream Name**. This will be the name of the stream that will be created in your destination. Add additional streams by clicking "Add". 
10. For each stream, select in the dropdown menu the **File Type** you wish to sync. Depending on the format chosen, you'll see a set of options specific to the file type. You can read more about specifics to each file type below.
11. (Optional) Provide a **Start Date** using the provided datepicker, or by entering the date in the format `YYYY-MM-DDTHH:mm:ss.SSSSSSZ`. Incremental syncs will only sync files modified/added after this date.
12. (Optional) Specify the **Host Address**. The default port for SFTP is 2​2. If your remote server is using a different port, enter it here.
(Optional) Determine the **Folder Path**. This determines the directory to search for files in, and defaults to "/". If you prefer to specify a specific folder path, specify the directory on the remote server to be synced. For example, given the file structure:

```
Root
| - logs
|   | - 2021
|   | - 2022
|
| - files
|   | - 2021
|   | - 2022
```

An input of `/logs/2022` will only replicate data contained within the specified folder, ignoring the `/files` and `/logs/2021` folders. Leaving this field blank will replicate all applicable files in the remote server's designated entry point. You may choose to enter a [regular expression](https://docs.oracle.com/javase/8/docs/api/java/util/regex/Pattern.html) to specify a naming pattern for the files to be replicated. Consider the following example:

```
log-([0-9]{4})([0-9]{2})([0-9]{2})
```

This pattern will filter for files that match the format `log-YYYYMMDD`, where `YYYY`, `MM`, and `DD` represented four-digit, two-digit, and two-digit numbers, respectively. For example, `log-20230713`. Leaving this field blank will replicate all files not filtered by the previous two fields.

14. Click **Set up source** to complete setup. A test will run to verify the configuration.

### For Airbyte Open Source:

1. Navigate to the Airbyte Open Source dashboard.
2. Click Sources and then click + New source.
3. On the Set up the source page, select SFTP Bulk from the Source type dropdown.
4. Enter a name for the SFTP Bulk connector.

#### Delivery Method

<FieldAnchor field="delivery_method.delivery_type">

Choose a [delivery method](../../platform/using-airbyte/delivery-methods) for your data. 

</FieldAnchor>

##### Preserve Sub-Directories in File Paths

If enabled, sends subdirectory folder structure along with source file names to the destination. Otherwise, files will be synced by their names only. This option is ignored when file-based replication is not enabled.

#### File-specific Configuration

Depending on your **File Type** selection, you will be presented with a few configuration options specific to that file type. 

For JSONL, Parquet, and Document File Type formats, you can specify the **Glob** pattern used to specify which files should be selected from the file system. If your provided Folder Path already ends in a slash, you need to add that double slash to the glob where appropriate.

For example, assuming your folder path is not set in the connector configuration and your files are located in the root folder, use a glob pattern like `//my_prefix_*.csv` to specify your file. If your files are in a folder, include the folder in your glob pattern, like `//my_folder/my_prefix_*.csv`.

If your files are in a folder, include the folder in your glob pattern, like `my_folder/my_prefix_*.csv`.

## Supported sync modes

The SFTP Bulk source connector supports the following [sync modes](https://docs.airbyte.com/cloud/core-concepts/#connection-sync-modes):

| Feature                        | Support | Notes |
|:-------------------------------|:-------:|:------|
| Full Refresh - Overwrite       |    ✅    |       |
| Full Refresh - Append Sync     |    ✅    |       |
| Incremental - Append           |    ✅    |       |
| Incremental - Append + Deduped |    ❌    |       |

## Supported Streams

This source provides a single stream per file with a dynamic schema. The current supported type files are Avro, CSV, JSONL, Parquet, and Document File Type Format. 

## Changelog
<details>
  <summary>Expand to review</summary>

| Version | Date       | Pull Request                                             | Subject                                                     |
|:--------|:-----------|:---------------------------------------------------------|:------------------------------------------------------------|
| 1.8.1 | 2025-05-10 | [58962](https://github.com/airbytehq/airbyte/pull/58962) | Update dependencies |
| 1.8.0 | 2025-05-07 | [57514](https://github.com/airbytehq/airbyte/pull/57514) | Adapt file-transfer records to latest protocol, requires platform >= 1.7.0, destination-s3 >= 1.8.0 |
| 1.7.8 | 2025-04-19 | [58448](https://github.com/airbytehq/airbyte/pull/58448) | Update dependencies |
| 1.7.7 | 2025-04-05 | [57475](https://github.com/airbytehq/airbyte/pull/57475) | Update dependencies |
| 1.7.6 | 2025-03-29 | [56898](https://github.com/airbytehq/airbyte/pull/56898) | Update dependencies |
| 1.7.5 | 2025-03-22 | [54083](https://github.com/airbytehq/airbyte/pull/54083) | Update dependencies |
| 1.7.4 | 2025-02-08 | [53570](https://github.com/airbytehq/airbyte/pull/53570) | Update dependencies |
| 1.7.3 | 2025-02-01 | [52971](https://github.com/airbytehq/airbyte/pull/52971) | Update dependencies |
| 1.7.2 | 2025-01-25 | [52470](https://github.com/airbytehq/airbyte/pull/52470) | Update dependencies |
| 1.7.1 | 2025-01-18 | [43821](https://github.com/airbytehq/airbyte/pull/43821) | Starting with this version, the Docker image is now rootless. Please note that this and future versions will not be compatible with Airbyte versions earlier than 0.64 |
| 1.7.0 | 2025-01-17 | [51611](https://github.com/airbytehq/airbyte/pull/51611) | Promoting release candidate 1.7.0-rc.1 to a main version. |
| 1.7.0-rc.1   | 2025-01-16 | [50972](https://github.com/airbytehq/airbyte/pull/50972) | Include option to not mirroring subdirectory structure.     |
| 1.6.0   | 2024-12-17 | [49826](https://github.com/airbytehq/airbyte/pull/49826) | Increase individual file size limit.                        |
| 1.5.0   | 2024-12-02 | [48434](https://github.com/airbytehq/airbyte/pull/48434) | Add get_file method for file-transfer feature.              |
| 1.4.0   | 2024-10-31 | [46739](https://github.com/airbytehq/airbyte/pull/46739) | make private key an airbyte secret.                         |
| 1.3.0   | 2024-10-31 | [47703](https://github.com/airbytehq/airbyte/pull/47703) | Update dependency to CDK v6 with ability to transfer files. |
| 1.2.0   | 2024-09-03 | [46323](https://github.com/airbytehq/airbyte/pull/46323) | Update dependency to CDK v5                                 |
| 1.1.0   | 2024-08-14 | [44028](https://github.com/airbytehq/airbyte/pull/44028) | Update dependency to CDK v4                                 |
| 1.0.1   | 2024-05-29 | [38703](https://github.com/airbytehq/airbyte/pull/38703) | Avoid error on empty stream when running discover           |
| 1.0.0   | 2024-03-22 | [36256](https://github.com/airbytehq/airbyte/pull/36256) | Migrate to File-Based CDK. Manage dependencies with Poetry. |
| 0.1.2   | 2023-04-19 | [19224](https://github.com/airbytehq/airbyte/pull/19224) | Support custom CSV separators                               |
| 0.1.1   | 2023-03-17 | [24180](https://github.com/airbytehq/airbyte/pull/24180) | Fix field order                                             |
| 0.1.0   | 2021-24-05 | [17691](https://github.com/airbytehq/airbyte/pull/17691) | Initial version                                             |

</details>
