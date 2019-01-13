# node.js安装

1. GitHub上找到nvm-windows，下载nvm-setup

2. 安装，选好nvm目录和nodejs目录，它会自动创建环境变量

3. 设置镜像

   ```shekk
   nvm node_mirror  https://npm.taobao.org/mirrors/node/
   nvm npm_mirror https://npm.taobao.org/mirrors/npm/
   ```

4. 安装某一版本

   ```shell
   nvm install v10.15.0
   ```

5. 使用某一版本

   ```shell
   nvm use 10.15.0
   ```

6. 可查看npm和node版本

   ```shell
   npm -v
   node -v
   ```

7. 设置缓存目录和全局模块目录

   ```shell
   npm config set cache "D:\devtools\nodejs\node_cache"
   npm config set prefix "D:\devtools\nodejs"
   ```

8. 设置npm的淘宝源

   ```shell
   npm config set registry https://registry.npm.taobao.org --global
   npm config set disturl https://npm.taobao.org/dist --global
   ```

9. 愉快的开始吧，比如安装vue-cli

   ```shell
   npm install -g @vue/cli
   vue --version
   ```

10. 