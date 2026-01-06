## Clone Existing Git Repository on Storage Server

---

#### Task Overview

The Nautilus application development team requires a local copy of an existing Git repository on the Storage Server in the Stratos DC.

This task involves cloning an already-initialized Git repository from a local path to a target directory without making any modifications to permissions, ownership, or repository contents.

##### Given Details

- Source repository: /opt/apps.git

- Target directory: /usr/src/kodekloudrepos

- User: natasha

- Server: Storage Server (ststor01)

##### Constraints:

- Do not modify repository content

- Do not change permissions or ownership

- Perform actions strictly as the natasha user

---

### Step-by-Step Solution

#### 1. Switch to the natasha User

Ensure all operations are performed using the correct user.

```
sudo su - natasha
```

#### 2. Ensure Target Directory Exists

Verify that the destination directory exists before cloning.

```
ls -ld /usr/src/kodekloudrepos
```

#### 3. Clone the Git Repository

Clone the repository from the local path into the target directory.

```
git clone /opt/apps.git /usr/src/kodekloudrepos/apps
```

#### 4. Verify the Clone

Confirm that the repository was cloned successfully.

```
ls -l /usr/src/kodekloudrepos/apps
```

##### Check Git status to ensure repository integrity.

```
cd /usr/src/kodekloudrepos/apps
git status
```

#### Notes

- This repository was cloned from a local bare Git repository

- No remote URLs or credentials were required

- The task complies strictly with DevOps change control rules
