# Install Percona XtraBackup Pro

--8<--- "pro-build-announcement.md"

This document provides guidelines how to install Pro packages of Percona XtraBackup from Percona repositories. [Check files in packages built for Percona XtraBackup Pro :material-arrow-right:](pro-files.md){.md-button}

## Procedure

1. Request the access to the pro repository from Percona Support. You will receive the client ID and the access token which you use when downloading the packages.

3. Configure the repository and install Percona XtraBackup packages

    === "On Debian or Ubuntu"

        1. Download a `DEB` package for *percona-release* the repository packages from Percona web:

            ```{.bash data-prompt="$"}
            $ wget https://repo.percona.com/apt/percona-release_latest.$(lsb_release -sc)_all.deb
            ```

        2. Install the downloaded package with **dpkg**. To do that, run the following commands as root or with **sudo**: `dpkg -i percona-release_latest.$(lsb_release -sc)_all.deb`
   
            Once you install this package the Percona repositories should be added. You
            can check the repository setup in the `/etc/apt/sources.list.d/percona-release.list` file.

        3. Enable the repository: `percona-release enable-only tools release`

            If Percona XtraBackup is intended to be used in combination with the upstream MySQL Server, you only need to enable the `tools` repository: `percona-release enable-only tools`.

        4. Refresh the local cache to update the package information:

            ```{.bash data-prompt="$"}
            $ sudo apt update
            ```

        5. Install the `percona-xtrabackup-pro` package:

            ```{.bash data-prompt="$"}
            $ sudo apt install percona-xtrabackup-pro
            ```

        6. To decompress backups made using `LZ4` or `ZSTD` compression algorithm, install the corresponding package:

            === "Install the `lz4` package"

                ```{.bash data-prompt="$"}
                $ sudo apt install lz4
                ```

            === "Install the `zstd` package"

                ```{.bash data-prompt="$"}
                $ sudo apt install zstd
                ```

    === "On RHEL or derivatives"

        1. Install the Percona yum repository by running the following command as the `root` user or with **sudo**: 

            ```{.bash data-prompt="$"}
            $ sudo yum install \
            https://repo.percona.com/yum/percona-release-latest.\
            noarch.rpm
            ```

        2. Enable the repository: 

            ```{.bash data-prompt="$"}
            $ sudo percona-release enable-only tools release
            ```

            If *Percona XtraBackup* is intended to be used in combination with
            the upstream MySQL Server, you only need to enable the `tools repository: 

            ```{.bash data-prompt="$"}
            $ sudo percona-release enable-only tools
            ```

        3. Install *Percona XtraBackup* by running:

            ```{.bash data-prompt="$"}
            $ sudo yum install percona-xtrabackup-pro
            ```

        4. To decompress backups made using `LZ4` or `ZSTD` compression algorithm, install the corresponding package:

            === "Install the `lz4` package"

                ```{.bash data-prompt="$"}
                $ sudo yum install lz4
                ```

            === "Install the `zstd` package"

                ```{.bash data-prompt="$"}
                $ sudo yum install zstd
                ```
