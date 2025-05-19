# Windows配置GitHub项目deploy key

---

- 创建项目 ssh key (注意: **一个 deploy key 只能用 1 次**)

    - powershell进入 ~/.ssh

    - ```bash
        ssh-keygen -t ed25519 -C "yourname@example.com"
        ```

    - 询问放置路径,修改复制下面的

        - ```
            C:\Users\Administrator/.ssh/id_project
            ```

    - ls 一下,就会发现有 `id_project.pub` 和 `id_project`两个内容

- 复制 ssh key

    - ```bash
        cat .\id_project.pub	
        ```

    - 复制输出的内容到GitHub->项目仓库->settings->deploy keys-> add deploy key

- 编辑 config 文件

    - 如果安装了 vscode, 使用这个编辑

    - ```bash
        code config
        ```

    - 添加如下内容

    - ```
        Host project.github.com
            HostName github.com
            User git
            IdentityFile ~/.ssh/id_project
            ProxyCommand connect -S 127.0.0.1:7890 %h %p
        ```

    - `ProxyCommand connect -S 127.0.0.1:7890 %h %p` 这行是说使用系统代理

- clone 代码到本地

    - 进入你的 code 目录,修改原始 clone 路径

    - ```bash
        # 原始路径 git@github.com:Aris1024/depin_spark.git
        # 修改成 git@project.github.com:Aris1024/depin_spark.git
        git clone git@project.github.com:Aris1024/depin_spark.git
        ```



enjoy! 😄

