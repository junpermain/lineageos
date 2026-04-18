一、基础版：自动构建 LineageOS（核心模板）
 
yaml
  
name: Build LineageOS

# 触发条件：手动触发 + 定时触发（可选）
on:
  workflow_dispatch:  # 允许在 GitHub 页面手动点击运行
  schedule:
    - cron: '0 0 * * 0'  # 每周日 0 点自动构建（可选）

jobs:
  build:
    runs-on: ubuntu-latest  # 使用 Ubuntu 环境
    container:
      image: ubuntu:20.04  # 和官方编译指南一致的系统版本

    steps:
      - name: Install dependencies
        run: |
          apt update && apt install -y bc bison build-essential ccache curl flex g++-multilib gcc-multilib git git-lfs gnupg gperf imagemagick lib32readline-dev lib32z1-dev libelf-dev liblz4-tool libsdl1.2-dev libssl-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev

      - name: Set up repo
        run: |
          curl https://storage.googleapis.com/git-repo-downloads/repo > /usr/bin/repo
          chmod a+x /usr/bin/repo

      - name: Configure git
        run: |
          git config --global user.name "你的GitHub用户名"
          git config --global user.email "你的GitHub邮箱"

      - name: Initialize LineageOS repo
        run: |
          mkdir -p ~/android/lineage
          cd ~/android/lineage
          repo init -u https://github.com/LineageOS/android.git -b lineage-22.2 --git-lfs

      - name: Sync source code
        run: |
          cd ~/android/lineage
          repo sync -j$(nproc) -c

      - name: Set up environment
        run: |
          cd ~/android/lineage
          source build/envsetup.sh

      - name: Download device tree
        run: |
          cd ~/android/lineage
          breakfast earth  # 替换成你的设备代号（比如 earth）

      - name: Extract proprietary blobs
        run: |
          cd device/xiaomi/earth  # 替换成你的设备路径
          ./extract-files.py

      - name: Build LineageOS
        run: |
          cd ~/android/lineage
          brunch earth  # 替换成你的设备代号

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: lineage-build
          path: ~/android/lineage/out/target/product/earth/*.zip
 
 
 
 
二、关键部分说明（你需要修改的地方）
 
1. 触发条件-  workflow_dispatch ：开启后，你可以在 GitHub 仓库的  Actions  页面，手动点击按钮运行构建，非常方便调试。
-  schedule ：可选，设置定时自动构建，比如每周一次。
2. 设备代号与分支-  lineage-22.2 ：替换成你要构建的 LineageOS 分支（比如  lineage-23.2 ）。
-  earth ：替换成你的设备代号（比如你之前的 Redmi 12C 就是  earth ）。
-  device/xiaomi/earth ：替换成你的设备树路径，根据你的设备厂商和代号修改。
3. 依赖安装- 模板里的依赖包是 LineageOS 官方推荐的，直接复制即可，不需要修改。
4. 上传构建产物- 最后一步会把生成的  .zip  刷机包上传到 GitHub Actions 的 Artifacts 里，构建完成后你可以直接下载使用。
 
 
 
三、进阶优化（可选）
 
1. 开启 ccache 加速构建
在  Install dependencies  步骤后添加：yaml
  
- name: Set up ccache
  run: |
    apt install -y ccache
    echo "export USE_CCACHE=1" >> ~/.bashrc
    echo "export CCACHE_DIR=~/.ccache" >> ~/.bashrc
    source ~/.bashrc
    ccache -M 50G
 
2. 缓存源码和构建文件
使用  actions/cache  缓存  repo  源码和  ccache ，大幅减少后续构建时间：yaml
  
- name: Cache ccache
  uses: actions/cache@v4
  with:
    path: ~/.ccache
    key: ${{ runner.os }}-ccache-${{ hashFiles('**/*.mk') }}
    restore-keys: |
      ${{ runner.os }}-ccache-
 
 
 
 
四、使用步骤
 
1. 把上面的模板复制到  main.yml  文件里。
2. 修改以下内容：- 你的 GitHub 用户名和邮箱
- 你的 LineageOS 分支（比如  lineage-22.2 ）
- 你的设备代号（比如  earth ）
- 你的设备树路径
3. 点击  Commit changes...  保存文件。
4. 进入仓库的  Actions  页面，找到  Build LineageOS  工作流，点击  Run workflow  即可开始构建。
 
 
 
