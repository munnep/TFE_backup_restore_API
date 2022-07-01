# TFE_backup_restore_API

This repository describes how to de a backup and restore from your TFE environment using the backup and API. The repository steps are mainly based on the following official documentation found [here](https://www.terraform.io/enterprise/admin/infrastructure/backup-restore)

These same procedures can be used to migrate from a TFE with mounted disk to TFE external services which is in the example of this repo

# Prerequisites

For creating the mounted disk installation in AWS use the following repository
[https://github.com/munnep/TFE_aws_disk](https://github.com/munnep/TFE_aws_disk)

For the external services installation in AWS use the following repository
[https://github.com/munnep/TFE_aws_external](https://github.com/munnep/TFE_aws_external)

# How to

- Clone the repository to your local machine
```
git clone https://github.com/munnep/TFE_backup_restore_API.git
```
- Go to the directory
```
cd TFE_backup_restore_API
```
- Create the TFE mounted disk installation environment using the repo [https://github.com/munnep/TFE_aws_disk](https://github.com/munnep/TFE_aws_disk)
- On the TFE environment generate an organization, workspace, users everything you like and that you can check after a restore. 
- Get the Backup API token from the Terraform Enterprise dashboard under settings
- 
```
4355a63556b400097d4e247fb6366484c197ae86a8994b29c15a60164340116e
```
- create a payload with an encryption password for the file. See `payload.json` for an example

```
{
    "password": "Vewyrubskjdf@#$890werjsdFSFDSdf"
}
```

- create a backup as follow. Change the URL to your own TFE environment

```
export TOKEN=4355a63556b400097d4e247fb6366484c197ae86a8994b29c15a60164340116e
curl \
  --header "Authorization: Bearer $TOKEN" \
  --request POST \
  --data @payload.json \
  --output backup.blob \
  https://patrick-tfe9.bg.hashicorp-success.com/_backup/api/v1/backup
```
output: you don't get any output from the above command. Just a return to your prompt
```
```
- you should now have a `backup.blob` file. This should be used to restore to a different TFE environment
- Destroy the current TFE mounted disk environment. 
- create a new TFE environment **(same version as the mounted disk environment)** that is using external services using the following repo [https://github.com/munnep/TFE_aws_external](https://github.com/munnep/TFE_aws_external) 
- Do not create an organization or anything within TFE itself
- Get the **new** Backup API token from the Terraform Enterprise dashboard under settings

```
4355a63556b400097d4e247fb6366484c197ae86a8994b29c15a60164340116e
```
- Make sure you have a `payload.json` with the encryption password of the `backup.blob` file

```
{
    "password": "Vewyrubskjdf@#$890werjsdFSFDSdf"
}
```

- restore the backup into you new TFE environment

```
export TOKEN=4355a63556b400097d4e247fb6366484c197ae86a8994b29c15a60164340116e
curl \
  --header "Authorization: Bearer $TOKEN" \
  --request POST \
  --form config=@payload.json \
  --form snapshot=@backup.blob \
  https://patrick-tfe9.bg.hashicorp-success.com/_backup/api/v1/restore

```
output:
```
snapshot applied successfully
```

- You should restart the TFE application before you can continue
- ssh into your server and do the following
```
replicatedctl app stop
replicatedctl app start
```
- you should now be able to login and see your data and things you created earlier on the mounted disk environment






