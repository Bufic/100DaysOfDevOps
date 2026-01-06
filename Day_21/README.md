## Git Repository Setup on Nautilus Storage Server

---

#### Overview

This document describes the procedure used to set up a centralized Git repository for a new application development project on the Nautilus infrastructure in the Stratos Data Center.

The objective is to install Git on the Storage Server and create a bare Git repository that will serve as the central source control location for the development team.

This setup follows enterprise DevOps best practices and mirrors how Git servers are provisioned in real-world environments.

#### Requirements

- Git must be installed using yum

- A bare Git repository must be created

- Repository path must be exactly:

  ```
  /opt/demo.git
  ```

  ***

  ### Step-by-Step Implementation

#### 1. Access the Storage Server

Log in to the Storage Server from the jump host:

```
ssh natasha@ststor01
```

##### Expected result:

You gain shell access to the Storage Server.

#### 2. Install Git Using yum

Install Git using the system package manager:

```
sudo yum install -y git
```

#### Why this is required:

- Ensures Git is installed from a trusted, vendor-supported repository

- Automatically resolves dependencies

- Aligns with enterprise system management standards

##### Verification:

```
git --version
```

Expected output:

```
git version 2.x.x
```

#### 3. Create the Bare Git Repository

Create the repository directory and initialize it as a bare repository:

```
sudo mkdir -p /opt/demo.git
sudo git init --bare /opt/demo.git
```

##### Why a bare repository?

- Contains only Git metadata (no working files)

- Prevents direct editing on the server

- Designed for shared access by developers and CI/CD systems

##### Why /opt?

- Standard Linux directory for application-related data

- Keeps repositories separate from OS and user files

- Common practice in enterprise environments

#### 4. Verify Repository Creation

List the repository contents:

```
ls -l /opt/demo.git
```

You should see files and directories such as:

```
HEAD
config
objects/
refs/
hooks/
```

##### Conclusion

The Git repository on the Nautilus Storage Server is now ready for use.
This configuration provides a secure, scalable, and industry-standard foundation for collaborative development and DevOps automation.
