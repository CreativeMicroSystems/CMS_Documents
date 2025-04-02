#################################
#### Updated on : 31/03/2025 ####
#################################

Linux server IP : 192.168.10.193
Username : yocto_imx6
Password : Redhat$321


export REPO_URL='https://mirrors.tuna.tsinghua.edu.cn/git/git-repo/'

repo init -u https://github.com/MYiR-Dev/myir-imx-manifest.git --no-clone-bundle --depth=1 -m myir-i.mx6ul-5.4.3-2.0.0.xml -b i.MX6UL-5.4-zeus 
----OR---
repo init -u https://github.com/MYiR-Dev/myir-imx-manifest.git --no-clone-bundle --depth=1 -m myir-i.mx6ul-5.10.9-1.0.0.xml -b i.MX6UL-5.10-gatesgarth
repo sync

DISTRO=fsl-imx-fb MACHINE=myd-y6ull14x14 source myir-setup-release.sh -b build_imx6ull
bitbake myir-image-full



*********** fixes **********
I am not sure about the Torizon Core Builder but I was facing the same issue while compiling Yocto using Bitbake based on BSP 5.7.2.

I have added these lines to the conf/local.conf file and it solved the problem:

MIRRORS += " \
    git://source.codeaurora.org/external/imx/ git://github.com/nxp-imx-support/;protocol=https \n \
    https://source.codeaurora.org/external/imx/ https://github.com/nxp-imx-support/ \n \
    http://source.codeaurora.org/external/imx/ http://github.com/nxp-imx-support/ \n \
    gitsm://source.codeaurora.org/external/imx/ gitsm://github.com/nxp-imx-support/;protocol=https \n \
"
Hope it works for you, too.

Regards,
Aditya

wget https://downloads.sourceforge.net/expat/expat-2.2.8.tar.bz2 -P /mnt/newdiskpart1/yocto_imx6/sanjay_tmp/

