20260409
在 Update feeds 步骤中添加了强制更新 helloworld 的逻辑
--------
    - name: Update feeds
      run: |
        cd openwrt
        ./scripts/feeds update -a
        
        # 强制更新 helloworld 到最新版本
        echo "=== Updating helloworld to latest ==="
        rm -rf feeds/helloworld package/feeds/helloworld
        git clone --depth=1 https://github.com/fw876/helloworld.git feeds/helloworld
        
        # 验证版本
        echo "=== Helloworld latest commit ==="
        cd feeds/helloworld && git log --oneline -1 && cd ../..

20260401
解决GitHub Actions OpenWrt 编译磁盘空间错误
--------

使用 jlumbroso/free-disk-space
    
    - name: Free Disk Space
      uses: jlumbroso/free-disk-space@main
      with:
        tool-cache: true        # 清理工具缓存
        android: true           # 清理 Android SDK
        dotnet: true            # 清理 .NET
        haskell: true           # 清理 Haskell
        large-packages: true    # 清理大软件包
        docker-images: true     # 清理 Docker 镜像
        swap-storage: true      # 清理 swap

    - name: Maximize Build Space
      uses: easimon/maximize-build-space@master
      with:
        root-reserve-mb: 30720   # 30GB 给根分区
        swap-size-mb: 10240
        build-mount-path: '/workdir'

应用 Deep cleanup before build
   
    - name: Deep cleanup before build
      run: |
        echo "=== Before cleanup ==="
        df -h
        
        # 清理大型目录
        sudo rm -rf /usr/share/dotnet
        sudo rm -rf /usr/local/lib/android
        sudo rm -rf /opt/ghc
        sudo rm -rf /opt/hostedtoolcache
        sudo rm -rf /usr/local/share/chromedriver-linux64
        sudo rm -rf /usr/local/share/chromium
        sudo rm -rf /usr/share/swift
        sudo rm -rf /usr/local/julia*
        sudo rm -rf /usr/local/graalvm*
        
        # 清理 Docker
        sudo docker rmi -f $(docker images -q) 2>/dev/null || true
        
        # 清理 apt
        sudo apt-get clean
        sudo apt-get autoremove -y
        sudo rm -rf /var/lib/apt/lists/*
        
        # 清理日志
        sudo journalctl --vacuum-time=1d
        sudo find /var/log -type f -name "*.gz" -delete
        sudo find /var/log -type f -name "*.old" -delete
        
        echo "=== After cleanup ==="
        df -h
