# 安装依赖，根据系统（Ubuntu 24.04、HCE2.0）不同选择相应的脚本
```shell
# install gcc 12.3 & numa in Ubuntu 24.04
apt-get update  -y
apt-get install -y gcc-12 g++-12 libnuma-dev
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 10 --slave /usr/bin/g++ g++ /usr/bin/g++-12


# install gcc-12.3 & numa in HCE2.0
cd ~
wget https://ftp.gnu.org/gnu/gcc/gcc-12.3.0/gcc-12.3.0.tar.gz
tar -xzf gcc-12.3.0.tar.gz && cd gcc-12.3.0
./contrib/download_prerequisites   
mkdir build && cd build
../configure --enable-languages=c,c++ --disable-multilib
make -j$(nproc)                    
make install
gcc -v
yum install numactl-devel -y
```


# 创建虚拟环境，基于源码编译vLLM
``` shell
# download vllm
cd ${HOME}
git clone https://github.com/vllm-project/vllm.git --branch v0.8.3 vllm_source


# install Conda
conda_home=${HOME}/miniconda3
mkdir -p ${conda_home}
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-aarch64.sh -O ${conda_home}/miniconda.sh
bash ${conda_home}/miniconda.sh -b -u -p ${conda_home}
rm -f ${conda_home}/miniconda.sh
source ${conda_home}/bin/activate
conda init --all


# set python mirror
mkdir ~/.pip
cat > ~/.pip/pip.conf << 'EOF'
[global]
index-url = https://repo.huaweicloud.com/repository/pypi/simple
trusted-host = repo.huaweicloud.com
timeout = 120
EOF


# create env
conda create -n vllm python=3.12 -y
conda activate vllm

cd ${HOME}/vllm_source
pip install "cmake>=3.26" wheel packaging ninja "setuptools-scm>=8" numpy
pip install -v -r requirements/cpu.txt --extra-index-url https://download.pytorch.org/whl/cpu
VLLM_TARGET_DEVICE=cpu python setup.py install
```


# 安装加速库，根据系统（Ubuntu 24.04、HCE2.0）不同选择相应的脚本
```shell
# install tcmalloc in Ubuntu 24.04
apt-get install libtcmalloc-minimal4

# install tcmalloc in HCE2.0
yum install gperftools -y
```

# 根据系统（Ubuntu 24.04、HCE2.0）不同设置加速库和虚拟环境
**编辑~/.bashrc文件追加下面内容**
## Ubuntu 24.04
```shell
conda activate vllm
export LD_PRELOAD=/usr/lib/aarch64-linux-gnu/libtcmalloc_minimal.so.4:$LD_PRELOAD
```

## HCE2.0
```shell
conda activate vllm
export LD_PRELOAD=/usr/lib64/libtcmalloc_minimal.so.4:$LD_PRELOAD
```
