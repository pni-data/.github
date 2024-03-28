# Research Data Repositories
*A collection of GitHub siblings of DataLad datasets. Some of the included datasets require access credentials.*

## Quick Contents
- [TRR289 datasets](https://github.com/search?q=topic%3Atrr289+org%3Apni-data&type=Repositories)
- [Pain sensitivity datasets (RPN)](https://github.com/search?q=topic%3Arpn+org%3Apni-data&type=Repositories)
- [SFB1280 datasets](https://github.com/search?q=topic%3Asfb1280+org%3Apni-data&type=Repositories)
- [Derivative datasets](https://github.com/search?q=topic%3Aderivatives+org%3Apni-data&type=Repositories)
- [BIDS-MEGA (BEP-035) example datasets](https://github.com/search?q=topic%3Abep035+org%3Apni-data&type=Repositories) ([see also](https://bids.neuroimaging.io/get_involved.html#extending-the-bids-specification))
- [PUMI example and test datasets](https://github.com/search?q=topic%3Apumi+org%3Apni-data&type=Repositories)

[**Browse all datasets**](https://github.com/orgs/pni-data/repositories)


----------------------
## Quickstart Guide
- ✅ [Requirements](#requirements-for-all-datasets)
- ⬇️ [Dataset installation](#i-would-like-to-clone-a-dataset)
- ⬆️ [Uploading changes](#i-want-to-upload-my-changes)
- 🆕 [Add new dataset](#i-want-to-add-a-new-dataset)

----------------------

## Requirements for all datasets in the lab's RDM system
- dataset must be a DataLad dataset
- dataset must be in BIDS format (for derivative and non-imaging data: at least a dataset_description.json file must be there)
- datasets must have a GitHub sibling in this GH organization
- datasets must have a reliable special remote (preferred: Coscine RDS-S3 or Amazon S3 for public datasets (e.g. clones of openneuro datasets)
- datasets must have an [unannexed](https://handbook.datalad.org/en/latest/basics/101-136-filesystem.html) readme.md and dataset_description.json
- GitHub repo description must be set to a short description of the dataset, ending with the sample size when possible (n=xy)

----------------------

## ⬇️ I would like to clone a dataset

#### 1. Install it with datalad based on the github handle
This does not download the actual data, only the "skeleton".
After the install command, you have to explictly tell datalad that you would like your github sibling (origin) to depend on the S3-sibling.
```bash
datalad install -s git@github.com:pni-data/<dataset_name>.git <dataset_name>
datalad siblings configure -s origin --publish-depends coscine-rds-s3
```
*If the dataset you are about to donwload is in a private github repo, you'll need to authenticate, as usual (e.g. with a Personal Access Token or a key).*

####  1.a Set up Proxy (optional)
*If your connection goes trough a proxy server, you'll need to allow its IP, to be able to communicatee with the S3 sibling.*
```bash
git config --add annex.security.allowed-ip-addresses <proxy-server-ip>
```

#### 2. Change to the dataset directory and download the file(s) you want
You can selectively download what you need (e.g. derivatives only).
```bash
cd <dataset_name>
datalad get <path/to/file*>
```
*Depending on the dataset, you will be prompted for the S3 credentials to access the files. In this case, contact the dataset owner to obtain the (read or write) credentials and set them uplikee this:*
```bash
export AWS_ACCESS_KEY_ID="XXXXX-XXXX-XXXX-XXXX-XXXX"
export AWS_SECRET_ACCESS_KEY="XXXXXXXX"
```
Now you should be able to get the data.

#### 3. Check the locations a file is stored at
```bash
git-annex whereis <path/to/file>
```
#### 4. Free up space, without deleting the dataset
As all datasets here are guaranteed to be also stored on an s3 remote, you can always safely drop any file from your local dataset. Datalad only drops the actual data, but not the annexed links. That is the "dataset skeleton" never has to be removed. You will still able to browse and search the dataset skeleton (and the metadata) and download a file again, if you need it.

```bash
datalad drop <path/to/file*>
```

----------------------

## ⬆️ I want to upload my changes 
Just save your changes and push/publish it to the ggithub sibling. As the github sibling depends on the coscine-rds-s3 special remote, the following command will upload the actual data to thee s3 storage. 
```bash
datalad save .
datalad push --to origin
```
----------------------

## 🆕 I want to add a new dataset

#### 1. Turn your folder a DataLad dataset
```bash
cd <my_dataset>
datalad create -f .
datalad save .
```

#### 2. Unannex small, public meta-data files
E.g. readme.md and dataset_description.json (this way these will be directly visible in github)
```bash
datalad no-annex --pattern readme.md
datalad save .
```

#### Create Special Remote sibling
Here we create a Coscine RDS-S3 sibling. 

You will need the following info about the S3 resource:
- Host name (e.g. coscine-s3-01.data.fds.uni-due.de)
- Port (e.g. 443)
- Access Key for Writing
- Secret Key for Writing
- Bucket Name

See the DataLad [docs](https://handbook.datalad.org/en/latest/basics/101-139-s3.html) for more detail.
```bash
export AWS_ACCESS_KEY_ID="your-access-key-for-writing"
export AWS_SECRET_ACCESS_KEY="your-secret-key-for-writing"
git-annex initremote coscine-rds-s3 type=S3 host=<your_host> port=<your_port> encryption=none bucket=<your_bucket_name> signature=v4 chunk=50mb autoenable=true
```

#### Create a github sibling
This is for RDM-purposes (listing, sharing, using issues, PRs, etc). The github repo will only contain data that is unannexed. It will disclose the directory tree and the filenames, though.
If that's not what you want, make the repo private with `--private`. See the DataLad [docs](https://docs.datalad.org/en/stable/generated/man/datalad-create-sibling-github.html) for details.

Requirements:
- you must be a member of the GitHub organization "pni-data" (or swap it your own profile or organization)
- you need a valid GitHub [Personal Access Token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- the gitHub repo must not yet exist (the command creates it)

```bash
datalad create-sibling-github -d . --github-organization -s origin pni-data <dataset_name> --publish-depends coscine-rds-s3 --access-protocol ssh
# here, your github token is needed
```

##### Check if you have both siblings ready
```bash
datalad siblings
```

> .: here(+) [git] \
> .: coscine-rds-s3(+) [git] \
> .: github(-) [git@github.com:pni-data/datalad_test2.git (git)]

#### Ready! Now you just need to push your data.
We push/publish the unannexed data and the annexed "dataset skeleton" to github. As the github sibling (origin) depends on the coscine-rds-s3 special remote, the following command will upload the actual data to thee s3 storage (in machine-readable chunks and, if requested, in an encrypted format). 

```bash
datalad push --to origin
```







