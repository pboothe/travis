# travis
A support library for adding deployment automation in travis.

# Add submodule to a new repository

So travis can automatically checkout the travis submodule, use the `https`
address. Think of this as a read-only link to the travis repo.

From the root directory of the repo you're adding to travis:
```
git submodule add https://github.com/m-lab/travis.git
```

# Creating accounts

From the top level repo (that contains travis as a submodule):

```
mkdir keys
./travis/create_service_account_and_key.sh \
    mlab-sandbox cloud-storage-deployer keys/mlab-sandbox.json
./travis/create_service_account_and_key.sh \
    mlab-staging cloud-storage-deployer keys/mlab-staging.json

# NB: do not include "." in the resulting tar file.
pushd keys
  tar --exclude=*.tar* -cvf service-accounts.tar *.json
popd

cp ./travis/template-travis.yml .travis.yml
travis encrypt-file keys/service-accounts.tar --add
```

Update the .travis.yml template to match your repository and deployment needs.

## Encrypting files for travis

### Recovering if encryption keys are overwritten

Encryption keys may be overwritten by invoking `travis encrypt-file` more than
once for the same repository.

In the event that the encryption keys are lost, there are a few
steps that have to be taken to restore functionality.

 1. If the SA keys are available, skip to step 4.
 2. Create new service accounts or new keys for existing account, for
    mlab-sandbox and mlab-staging, downloading the json key files.
 3. Update GCS ACLs, e.g.
    ```
    gsutil acl ch -u \
       legacy-rpm-writer@mlab-sandbox.iam.gserviceaccount.com:WRITE \
       gs://legacy-rpms-mlab-sandbox
    ```
 4. Tar the SA keys:
    tar cf service-accounts.tar legacy-rpm-writer.mlab*
 5. Encrypt the tar file:
    ```
    travis encrypt-file -f -p service-accounts.tar --repo m-lab/<repo-name>
    ```
    Optionally, if you want to provide the keys for some other repos,
    copy the key and iv values into a command like:
    ```
    travis encrypt-file -f -p service-accounts.tar --key \
      AAA151324478927bbbbbbbbbcccccccccccccdddddddddd53223551235324324 \
      --iv 632451671306d1842843a792250ce707 --repo gfr10598/ndt-support
    ```
 6. Copy the keys printed when you encrypted the tar file,
    and paste them in place of the three occurances in the script
    commands below.
 7. Copy the encrypted tar file to the travis directory (where
    this script is located).
 8. Commit to an appropriate branch, generate PR, and send for review.
