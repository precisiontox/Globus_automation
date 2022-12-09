
# Set up automatic file transfer from UOB Globus endpoint to UHEI Globus endpoint
- The script `uob-uhei-sync.sh` is edited from an example provided by Globus: 
- https://github.com/globus/automation-examples/blob/master/cli-sync.sh
## Current implementation
ssh to the VM `globus-sdshd-ptox`
```shell
    ssh -i {your ssh key} ubuntu@129.206.7.190
```
output:
```
    Welcome to Ubuntu 20.04 LTS (GNU/Linux 5.4.0-26-generic x86_64)
    
     * Documentation:  https://help.ubuntu.com
     * Management:     https://landscape.canonical.com
     * Support:        https://ubuntu.com/advantage
    
      System information as of Thu Dec  8 09:34:29 UTC 2022
    
      System load:  0.15              Processes:             132
      Usage of /:   36.9% of 9.52GB   Users logged in:       0
      Memory usage: 60%               IPv4 address for ens3: 192.168.0.28
      Swap usage:   0%
    
     * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
       just raised the bar for easy, resilient and secure K8s cluster deployment.
    
       https://ubuntu.com/engage/secure-kubernetes-at-the-edge
    
    33 updates can be applied immediately.
    To see these additional updates run: apt list --upgradable
    
    New release '22.04.1 LTS' available.
    Run 'do-release-upgrade' to upgrade to it.
        
        
    *** System restart required ***
    Last login: Wed Dec  7 14:45:17 2022 from 2.212.146.77   
```
Attempt to create a venv:
```
    python3 -m venv globusCLI
```
Output:
```
    The virtual environment was not created successfully because ensurepip is not
    available.  On Debian/Ubuntu systems, you need to install the python3-venv
    package using the following command.

    apt install python3.8-venv

    You may need to use sudo with that command.  After installing the python3-venv
    package, recreate your virtual environment.

    Failing command: ['/home/ubuntu/globusCLI/bin/python3', '-Im', 'ensurepip', '--upgrade', '--default-pip']
```

Install python3.8-venv: 
```
    sudo apt install python3.8-venv
``` 
Create and activate venv: 
```
    python3 -m venv globusCLI
    source globusCLI/bin/activate
```

Install pipx (Globus CLI doc recommend to install globus cli using pipx: https://docs.globus.org/cli/)
```
    sudo apt install pipx
```

Install Globus CLI: 
```
    pipx install globus-cli
```
Output:
```
    installed package globus-cli 3.10.1, Python 3.8.10
    These binaries are now globally available
        - globus
        ⚠️  Note: '/home/ubuntu/.local/bin' is not on your PATH environment variable. These binaries will not be globally accessible until your PATH is updated. Run `pipx ensurepath` to automatically add it, or manually modify your PATH in your shell's config file (i.e. ~/.bashrc).
        done! ✨ 🌟 ✨
```
Run `pipx ensurepath` to automatically add '/home/ubuntu/.local/bin' to PATH environment variable
```
    pipx ensurepath
```
output:
```
    Added /home/ubuntu/.local/bin to the PATH environment variable in /home/ubuntu/.bashrc

    Open a new terminal to use pipx ✨ 🌟 ✨
```
Setting Up Tab Completion by adding following lines to `~/.bashrc` (ref: https://docs.globus.org/cli/#tab-completion)
```shell
  if type globus > /dev/null 2>&1; then
      eval "$(globus --bash-completer)"
  fi
```
```
    source ~/.bashrc
```
Now we follow the instructions from the Globus Automation Examples Repository provided by Globus (https://github.com/globus/automation-examples).

Environment Setup
```
    sudo apt-get update
    sudo apt-get install git
    git clone https://github.com/globus/automation-examples
    cd automation-examples
    python3 -m venv globus_auto_venv
    source globus_auto_venv/bin/activate
```
Install requirements:
```
    pip install -r requirements.txt
```
output:
```
    Collecting globus-sdk<=2.0.0,>=1.1.0
      Downloading globus_sdk-1.11.0-py2.py3-none-any.whl (85 kB)
         |████████████████████████████████| 85 kB 694 kB/s 
    Collecting fair_research_login
      Downloading fair_research_login-0.3.0-py3-none-any.whl (27 kB)
    Collecting six<2.0.0,>=1.10.0
      Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
    Collecting pyjwt[crypto]<2.0.0,>=1.5.3
      Downloading PyJWT-1.7.1-py2.py3-none-any.whl (18 kB)
    Collecting requests<3.0.0,>=2.9.2
      Using cached requests-2.28.1-py3-none-any.whl (62 kB)
    Collecting cryptography>=1.4; extra == "crypto"
      Downloading cryptography-38.0.4-cp36-abi3-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (4.1 MB)
         |████████████████████████████████| 4.1 MB 28.4 MB/s 
    Collecting idna<4,>=2.5
      Using cached idna-3.4-py3-none-any.whl (61 kB)
    Collecting urllib3<1.27,>=1.21.1
      Using cached urllib3-1.26.13-py2.py3-none-any.whl (140 kB)
    Collecting certifi>=2017.4.17
      Using cached certifi-2022.12.7-py3-none-any.whl (155 kB)
    Collecting charset-normalizer<3,>=2
      Using cached charset_normalizer-2.1.1-py3-none-any.whl (39 kB)
    Collecting cffi>=1.12
      Using cached cffi-1.15.1-cp38-cp38-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (442 kB)
    Collecting pycparser
      Using cached pycparser-2.21-py2.py3-none-any.whl (118 kB)
    Installing collected packages: six, pycparser, cffi, cryptography, pyjwt, idna, urllib3, certifi, charset-normalizer, requests, globus-sdk, fair-research-login
    Successfully installed certifi-2022.12.7 cffi-1.15.1 charset-normalizer-2.1.1 cryptography-38.0.4 fair-research-login-0.3.0 globus-sdk-1.11.0 idna-3.4 pycparser-2.21 pyjwt-1.7.1 requests-2.28.1 six-1.16.0 urllib3-1.26.13
```
login to Globus
```
    globus login
```
output
```
    Please authenticate with Globus here:
    ------------------------------------
    https://auth.globus.org/v2/oauth2/authorize?client_id=835051c3-acf3-4291-90ec-ffb7a7dac030&redirect_uri=https%3A%2F%2Fauth.globus.org%2Fv2%2Fweb%2Fauth-code&scope=openid+profile+email+urn%3Aglobus%3Aauth%3Ascope%3Aauth.globus.org%3Aview_identity_set+urn%3Aglobus%3Aauth%3Ascope%3Atransfer.api.globus.org%3Aall+urn%3Aglobus%3Aauth%3Ascope%3Agroups.api.globus.org%3Aall+urn%3Aglobus%3Aauth%3Ascope%3Asearch.api.globus.org%3Aall+https%3A%2F%2Fauth.globus.org%2Fscopes%2F524230d7-ea86-4a52-8312-86065a9e0417%2Ftimer+https%3A%2F%2Fauth.globus.org%2Fscopes%2Feec9b274-0c81-4334-bdc2-54e90e689b9a%2Fmanage_flows+https%3A%2F%2Fauth.globus.org%2Fscopes%2Feec9b274-0c81-4334-bdc2-54e90e689b9a%2Fview_flows+https%3A%2F%2Fauth.globus.org%2Fscopes%2Feec9b274-0c81-4334-bdc2-54e90e689b9a%2Frun+https%3A%2F%2Fauth.globus.org%2Fscopes%2Feec9b274-0c81-4334-bdc2-54e90e689b9a%2Frun_status+https%3A%2F%2Fauth.globus.org%2Fscopes%2Feec9b274-0c81-4334-bdc2-54e90e689b9a%2Frun_manage&state=_default&response_type=code&access_type=offline&prompt=login
    ------------------------------------
    
    Enter the resulting Authorization Code here: QQCvoHKhgBNVsEHXJ1H9XRdBKtndoH
    
    You have successfully logged in to the Globus CLI!
    
    You can check your primary identity with
      globus whoami
    
    For information on which of your identities are in session use
      globus session show
    
    Logout of the Globus CLI with
      globus logout
    
```

Execute the example folder sync script:
```
  ./cli-sync.sh
```
output:
```
    Checking for a previous transfer
    Verified that source is a directory
    Submitted sync from ddb59aef-6d04-11e5-ba46-22000b92c6ec:/share/godata/ to ddb59af0-6d04-11e5-ba46-22000b92c6ec:/~/sync-demo/
    Link:
    https://app.globus.org/activity/53618988-77af-11ed-92e5-d578f8325bc7/overview
    Saving sync transfer ID to last-transfer-id.txt
```

Now, create our folder sync script using `./cli-sync.sh` as a template. 
```shell
  mkdir  ~/globus_automation
  cp cli-sync.sh ~/globus_automation/uob-uhei-sync.sh
```
In ` ~/globus_automation/uob-uhei-sync.sh`, assign 
- `SOURCE_ENDPOINT='d57770c0-538b-11ec-8fd4-e7402f1d930f'` 
- `DESTINATION_ENDPOINT='d2c74310-718e-11ed-92db-d578f8325bc7'`
- `SOURCE_PATH='/~/CDSI/'`
- `DESTINATION_PATH='/home/ubuntu/sdshd/sd22i001/CDSI/'`
- Remove the option `--delete` in `globus transfer` to avoid deleting files in the destination path which don't exist in the source path.
- Document of `globus transfer` can be found here: https://docs.globus.org/cli/reference/transfer/

Now run `uob-uhei-sync.sh`
```
     ~/globus_automation/uob-uhei-sync.sh
```
output:
```
    Checking for a previous transfer
    Verified that source is a directory
    Submitted sync from d57770c0-538b-11ec-8fd4-e7402f1d930f:/~/CDSI/ to d2c74310-718e-11ed-92db-d578f8325bc7:/home/ubuntu/sdshd/sd22i001/CDSI/
    Link:
    https://app.globus.org/activity/0a48eaac-77b4-11ed-86a5-7d4ee7d812ee/overview
    Saving sync transfer ID to last-transfer-id-test.txt
```
now set up a chrontab which runs `uob-uhei-sync.sh` periodically.
```
  crontab -e
```
in the crontab, add following lines:
```shell
  # Automatic copy data from UOB Globus endpoint to the Globus endpoint in the heiCloud VM globus-sdshd-ptox
  0 0,12 * * *  ~/globus_automation/uob-uhei-sync.sh
```
in the beginning of `~/globus_automation/uob-uhei-sync.sh` export the $PATH of the user `ububtu`. 
To avoid having to type the absolute path to a command, shells introduced the $PATH environment variable, each directory is separated by a : and searches are done from left to right. cron often clears the whole environment, including this $PATH variable. Therefore, the script may behave differently in your cron compared to the behavior in the shell.
```
    export PATH="/home/ubuntu/.local/bin:/home/ubuntu/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin"
```
Install a post service to enable Cron to send email
```
    sudo apt-get install postfix
```

## Notes for future improvements
### Periodically delete files which are older than x days in the UOB Globus endpoint in the path `~/CDSI`

From Globus support Ada Nikolaidis:
- it is possible to get a listing of files that were modified after a particular time (not the same as created time, 
  but perhaps could work for your scenario). So for instance, you could do something like: 

```shell
  COLLECTION_ID="00000000-0000-0000-0000-000000000000"
  globus api transfer GET /operation/endpoint/${COLLECTION_ID}/ls -Q "filter=last_modified:$(date -v-1d -Iseconds)
```
(Please note my example is using a FreeBSD version of date but you can construct a similar date using other variants.

Mu-En tested the following, and were able to list files in an endpoint, but was not yet able to list files in a 
sub-directory of an endpoint using `globus api transfer GET`
```shell
  COLLECTION_ID="d57770c0-538b-11ec-8fd4-e7402f1d930f"
  globus api transfer GET /operation/endpoint/d57770c0-538b-11ec-8fd4-e7402f1d930f/ls
```

output:
```shell
{
  "DATA": [
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-09 11:19:18+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "CDSI",
      "permissions": "2700",
      "size": 4096,
      "type": "dir",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-04 23:25:39+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "CRG",
      "permissions": "2700",
      "size": 4096,
      "type": "dir",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-01 16:40:31+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "UHEI",
      "permissions": "2700",
      "size": 4096,
      "type": "dir",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-08 09:50:06+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "heiCLOUD",
      "permissions": "2700",
      "size": 4096,
      "type": "dir",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-10-25 09:29:21+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "11107993304-TS_TUBE.fasta",
      "permissions": "0700",
      "size": 33673,
      "type": "file",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-01-25 16:11:58+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "Alfredo_DevelopmentalData.list.txt",
      "permissions": "0700",
      "size": 2971709,
      "type": "file",
      "user": "dhandapv"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-01-25 17:03:45+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "Alfredo_DevelopmentalData.listt.txt",
      "permissions": "0700",
      "size": 6755182,
      "type": "file",
      "user": "dhandapv"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2021-11-30 04:22:20+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "Alfredo_DevelopmentalData.tar.gz",
      "permissions": "0700",
      "size": 510787979533,
      "type": "file",
      "user": "dhandapv"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2021-12-02 01:16:08+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "human_dataset.tar.gz",
      "permissions": "0700",
      "size": 1063411745403,
      "type": "file",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_CASTLES_2017-orsinil-01",
      "last_modified": "2021-11-25 07:00:27+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "phylotox_data_analysis_rawData.tar.gz",
      "permissions": "0700",
      "size": 1895833865629,
      "type": "file",
      "user": "dhandapv"
    }
  ],
  "DATA_TYPE": "file_list",
  "absolute_path": null,
  "endpoint": "d57770c0-538b-11ec-8fd4-e7402f1d930f",
  "length": 10,
  "path": "/~/",
  "rename_supported": true,
  "symlink_supported": false,
  "total": 10
}

```

Another way to get the timestamps of the files in a specific sub-directory is using `globus ls`:
```shell
globus ls -l -r -F json d57770c0-538b-11ec-8fd4-e7402f1d930f:heiCLOUD
```
output:
```shell
{
  "DATA": [
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-05 10:44:34+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "rna_seq_contrast.tsv",
      "permissions": "0700",
      "size": 14686932,
      "type": "file",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-05 10:44:34+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "rna_seq_gene_count.tsv",
      "permissions": "0700",
      "size": 17785550,
      "type": "file",
      "user": "zhoujz"
    },
    {
      "DATA_TYPE": "file",
      "group": "gITS_BEAR_2019-colboujk-phylotoxicology",
      "last_modified": "2022-12-08 09:50:06+00:00",
      "link_group": null,
      "link_last_modified": null,
      "link_size": null,
      "link_target": null,
      "link_user": null,
      "name": "~$PrecisionTox-WP5-hash-identifiers_IUB_Drosophilas_Pilot.xlsx",
      "permissions": "0700",
      "size": 165,
      "type": "file",
      "user": "zhoujz"
    }
  ]
}

```
```shell
globus ls -l -r -F unix d57770c0-538b-11ec-8fd4-e7402f1d930f:heiCLOUD
```
output
```
DATA    file    gITS_BEAR_2019-colboujk-phylotoxicology 2022-12-05 10:44:34+00:00       None    None    None    None    None    rna_seq_contrast.tsv    0700    14686932file     zhoujz
DATA    file    gITS_BEAR_2019-colboujk-phylotoxicology 2022-12-05 10:44:34+00:00       None    None    None    None    None    rna_seq_gene_count.tsv  0700    17785550file     zhoujz
DATA    file    gITS_BEAR_2019-colboujk-phylotoxicology 2022-12-08 09:50:06+00:00       None    None    None    None    None    ~$PrecisionTox-WP5-hash-identifiers_IUB_Drosophilas_Pilot.xlsx   0700    165     file    zhoujz
```

