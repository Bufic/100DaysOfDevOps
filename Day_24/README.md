### Create a New Git Branch on Storage Server

---

#### Task Overview

The Nautilus development team requires a new branch in the repository to isolate upcoming features while keeping the master branch intact.

- Repository Path: /usr/src/kodekloudrepos/beta

- Base Branch: master

- New Branch: xfusioncorp_beta

- Server: Storage Server (Stratos DC)

- Task Constraint: Do not make any changes to the code.

#### Step-by-Step Instructions

##### Step 1: Log in to the Storage Server

SSH into the Storage Server (ststor01) where the repository is located.

```
ssh your_user@ststor01
```

##### Step 2: Switch to Root User (Required)

Because the repository permissions prevent the natasha user from modifying Git metadata, switch to root to create the branch safely:

```
sudo su -
```

##### Step 3: Navigate to the Repository

```
cd /usr/src/kodekloudrepos/beta
```

Verify that this is a valid Git repository:

```
git status
```

##### Step 4: Ensure You Are on master

```
git checkout master
```

Confirm branch status:

```
git status
```

Expected output:

```
On branch master
nothing to commit, working tree clean
```

##### Step 5: Create the New Branch

Create the branch xfusioncorp_beta:

```
git branch xfusioncorp_beta
```

Or, to create and switch to it immediately:

```
git checkout -b xfusioncorp_beta
```

##### Step 6: Verify Branch Creation

List all branches:

```
git branch
```

Expected output:

```
* xfusioncorp_beta
  master
```

#### Task completed successfully
