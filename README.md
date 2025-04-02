## 使用方法
(cuda12.4 torch2.4.0 avx2)
```
conda create -n kt python=3.11 -y
conda activate kt
sudo apt-get update -y
sudo apt-get install gcc g++ cmake ninja-build -y
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip install torch==2.4.0 --index-url https://download.pytorch.org/whl/cu124

pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.4cxx11abiFALSE-cp311-cp311-linux_x86_64.whl


pip install torch packaging ninja cpufeature numpy
sudo apt-get install --only-upgrade libstdc++6 -y
conda install -c conda-forge libstdcxx-ng -y
conda install conda-libmamba-solver=24.11.0 -y

strings ~/miniconda3/envs/kt/lib/libstdc++.so.6 | grep GLIBCXX_3.4.32 # 确认输出是否有内容
pip install ktransformers-0.2.4+cu124torch24fancy-cp311-cp311-linux_x86_64.whl
```

## 构建过程
```bash
conda create -n kt python=3.11 -y
conda activate kt
sudo apt-get update -y
sudo apt-get install gcc g++ cmake ninja-build -y
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip install torch==2.4.0 --index-url https://download.pytorch.org/whl/cu124

pip install https://github.com/Dao-AILab/flash-attention/releases/download/v2.7.4.post1/flash_attn-2.7.4.post1+cu12torch2.4cxx11abiFALSE-cp311-cp311-linux_x86_64.whl


pip install torch packaging ninja cpufeature numpy
sudo apt-get install --only-upgrade libstdc++6 -y
conda install -c conda-forge libstdcxx-ng -y
conda install conda-libmamba-solver=24.11.0 -y

strings ~/miniconda3/envs/kt/lib/libstdc++.so.6 | grep GLIBCXX_3.4.32 # 确认输出是否有内容

git clone https://github.com/kvcache-ai/ktransformers.git
cd ktransformers
git submodule init
git submodule update
bash install.sh
```

install.sh
```bash
#!/bin/bash
set -e  

# clear build dirs
rm -rf build
rm -rf *.egg-info
rm -rf csrc/build
rm -rf csrc/ktransformers_ext/build
rm -rf csrc/ktransformers_ext/cuda/build
rm -rf csrc/ktransformers_ext/cuda/dist
rm -rf csrc/ktransformers_ext/cuda/*.egg-info
rm -rf ~/.ktransformers

# 创建用于存储whl的目录
mkdir -p dist

echo "Installing python dependencies from requirements.txt"
pip install -r requirements-local_chat.txt
pip install -r ktransformers/server/requirements.txt

echo "Building ktransformers wheel..."
KTRANSFORMERS_FORCE_BUILD=TRUE pip wheel . --no-build-isolation -w dist

echo "Installing from built wheel..."
pip install --no-index --find-links=dist ktransformers

echo "Installing custom flashinfer..."
pip install third_party/custom_flashinfer/

SITE_PACKAGES=$(python -c "import site; print(site.getsitepackages()[0])")
echo "Copying thirdparty libs to $SITE_PACKAGES"
cp -a csrc/balance_serve/build/third_party/prometheus-cpp/lib/libprometheus-cpp-*.so* $SITE_PACKAGES/
patchelf --set-rpath '$ORIGIN' $SITE_PACKAGES/sched_ext.cpython*

echo "Installation completed successfully"
echo "Wheel package saved in: $(pwd)/dist"
```

