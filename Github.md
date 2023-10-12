### Github

1. #### 生成SSH Key：[generating-a-new-ssh-key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

   ```
   ssh-keygen -t ed25519 -C "2823688963@qq.com"
   ```

2. #### 将SSH Key添加到 ssh-agent：

   - 启动ssh-agent

     ```
     eval "$(ssh-agent -s)"
     ```

   - 添加

     ```
     ssh-add ~/.ssh/id_ed25519
     ```

   

   

3. #### 将SSH Key加入github账户：[adding-a-new-ssh-key](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account)

   1. 复制SSH key

      ```
      cat ~/.ssh/id_ed25519.pub
      ```

4. #### 查看是否连接到github:

   ```
   ssh -T git@github.com
   ```

5. #### 本地添加邮箱和账户名：

   ```
   git config --global user.name  "JiuPin-2823"  
   git config --global user.email  "2823688963@qq.com"
   ```

   可在~/.gitconfig查看或编辑

6. #### 添加远程路径（仓库SSH地址）：

   ```
   git remote add origin git@github.com:name/repo.git
   ```

   可在.git/config查看或编辑

7. #### 上传：

   - create a new repository on the command line：

     ```
     echo "# test" >> README.md
     git init
     git add README.md
     git commit -m "second commit"
     git branch -M main
     git remote add origin git@github.com:JiuPin-2823/Typora_ubuntu.git
     git push -u origin main
     ```

     

   - or push an existing repository from the command li:

     ```
     git remote add origin git@github.com:JiuPin-2823/test.git
     git branch -M main
     git push -u origin main
     ```

     

   - ##### 上传文件：

     ```
     git add .
     git commit -m "test"
     git push -u origin main
     ```

     

   - ##### 删除文件：[deleting-files](https://docs.github.com/en/repositories/working-with-files/managing-files/deleting-files-in-a-repository)

