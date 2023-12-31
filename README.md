# concourse-semver-test
Concourse CI での SemVer - セマンティックバージョニング のテスト

- 以下のREADMEの作業で、Concourse と Vault が動作していることが前提

    https://github.com/moriyamaES/vault-install

- 以下のサイトの記事を参考にして実施


    [公式のチュートリアル](https://concoursetutorial-ja.site.lkj.io/miscellaneous/versions-and-buildnumbers)


    https://stackoverflow.com/questions/55402179/automate-semver-in-concourseci


## 環境

- 環境は以下

    ```sh
    uname -a
    Linux control-plane.minikube.internal 6.4.12-1.el7.elrepo.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Aug 23 13:48:54 EDT 2023 x86_64 x86_64 x86_64 GNU/Linux
    ```


- vaulte を起動

    ```sh
    $ systemctl status vault.service 
    ● vault.service - "HashiCorp Vault - A tool for managing secrets"
    Loaded: loaded (/usr/lib/systemd/system/vault.service; disabled; vendor preset: disabled)
    Active: inactive (dead)
        Docs: https://www.vaultproject.io/docs/

    9月 10 13:30:27 control-plane.minikube.internal systemd[1]: [/usr/lib/systemd/system/vault.service:7] Un...t'
    9月 10 13:30:27 control-plane.minikube.internal systemd[1]: [/usr/lib/systemd/system/vault.service:8] Un...t'
    Hint: Some lines were ellipsized, use -l to show in full.
    ```

    ```sh
    systemctl start vault.service 
    ```

    ```sh
    $ systemctl status vault.service 
    ● vault.service - "HashiCorp Vault - A tool for managing secrets"
    Loaded: loaded (/usr/lib/systemd/system/vault.service; disabled; vendor preset: disabled)
    Active: active (running) since 日 2023-09-10 13:31:14 JST; 43s ago
        Docs: https://www.vaultproject.io/docs/
    Main PID: 4258 (vault)
        Tasks: 8
    Memory: 252.2M
    CGroup: /system.slice/vault.service
            └─4258 /usr/bin/vault server -config=/etc/vault.d/vault.hcl

    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Log Level:
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Mlock: supported: true, enabled: false
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Recovery Mode: false
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Storage: file
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Version: Vault v1.14.2, built 2023-08-24T13...2Z
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: Version Sha: 16a7033a0686eca50ee650880d5c55...89
    9月 10 13:31:14 control-plane.minikube.internal systemd[1]: Started "HashiCorp Vault - A tool for managi...".
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: ==> Vault server started! Log data will str...w:
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: 2023-09-10T13:31:14.178+0900 [INFO]  proxy ...""
    9月 10 13:31:14 control-plane.minikube.internal vault[4258]: 2023-09-10T13:31:14.181+0900 [INFO]  core: ...re
    Hint: Some lines were ellipsized, use -l to show in full.
    ```

    ```sh
    $ netstat -tuln | grep 8200
    tcp        0      0 10.1.1.200:8200         0.0.0.0:*               LISTEN 
    ```

<del>

- concourse を起動

    ```sh
    $ docker ps -a | grep conco
    367e9bba35ad   concourse/concourse                  "dumb-init /usr/loca…"   3 hours ago   Exited (137) 13 minutes ago                                                concourse-install-worker-1
    0465769c26d9   concourse/concourse                  "dumb-init /usr/loca…"   3 hours ago   Exited (137) 13 minutes ago                                                concourse-install-web-1
    f615ac05d83e   postgres                             "docker-entrypoint.s…"   3 hours ago   Exited (0) 14 minutes ago                                                  concourse-install-db-1
    ```

    ```sh
    $ cd ~/concourse-install/
    ```

    ```sh
    $ docker-compose down
    [+] Running 4/4
    ✔ Container concourse-install-worker-1  Removed                                                          0.0s 
    ✔ Container concourse-install-web-1     Removed                                                          0.0s 
    ✔ Container concourse-install-db-1      Removed                                                          0.0s 
    ✔ Network concourse-install_default     Removed                                                          0.5s 
    ```

    ```sh
    $ docker-compose up -d
    [+] Running 4/4
    ✔ Network concourse-install_default     Created                                                          0.5s 
    ✔ Container concourse-install-db-1      Started                                                          0.4s 
    ✔ Container concourse-install-web-1     Started                                                          0.1s 
    ✔ Container concourse-install-worker-1  Started                                                          0.2s
    ```

</del>

## Concourse と Vault　の接続テスト

<del>

- 以下のコマンドを実行

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p push-docker-image -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-tutorial/tutorials/miscellaneous/docker-images/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p push-docker-image -c pipeline.yml -n
    ```

    - 結果

        ```
        resources:
        resource tutorial has been added:
        + name: tutorial
        + source:
        +   branch: develop
        +   uri: https://github.com/drnic/concourse-tutorial.git
        + type: git
        
        resource hello-world-docker-image has been added:
        + name: hello-world-docker-image
        + source:
        +   email: ((docker-hub-email))
        +   password: ((docker-hub-password))
        +   repository: ((docker-hub-username))/concourse-tutorial-hello-world
        +   username: ((docker-hub-username))
        + type: docker-image
        
        jobs:
        job publish has been added:
        + name: publish
        + plan:
        + - get: tutorial
        + - params:
        +     build: tutorial/tutorials/miscellaneous/docker-images/docker
        +   put: hello-world-docker-image
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: ((docker-hub-username))/concourse-tutorial-hello-world
        +       type: docker-image
        +     params:
        +       NAME: ((docker-hub-username))
        +     platform: linux
        +     run:
        +       path: /bin/hello-world
        +   task: run
        + public: true
        
        pipeline name: push-docker-image

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/push-docker-image

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p push-docker-image
        - click play next to the pipeline in the web ui
        ```

- パイプラインのリソースをチェック


    ```
    $ fly -t tutorial check-resource -r push-docker-image/tutorial
    ```


    ```sh
    $ fly -t tutorial check-resource -r push-docker-image/hello-world-docker-image
    ```

    ```
    checking push-docker-image/hello-world-docker-image in build 1
initializing check: hello-world-docker-image
resource config creds evaluation: timed out to login to vault
errored
    
    ```

</del>


## vault unsealする

- vault サーバを指定

    ```sh
    $ export VAULT_ADDR='http://10.1.1.200:8200'
    ```

- ちなみに、前の手順より、Unseal Key と Root Token　は以下

    ```sh
    Unseal Key 1: tYN2ZXR6UOJKiMZzhQvu1nZ+5bymI/B8nSM0zQBlP+cH
    Unseal Key 2: zRm7x5T4yjtLWTwwwWaY5K/YL/dplOd+KQ6VyykVeCFH
    Unseal Key 3: 7xXvhns+T4hp8YT4Pd38EqmURNIU20o92itb8PTNmlt4
    Unseal Key 4: Y7iHvD3EVISub6uSjqt4aVxtnC+0B8OF7m6TXmYL5f9+
    Unseal Key 5: K9/xHj95ermVqZTnQlk8ZHJ4xtu4e6d0x+ylMKB90G4r

    Initial Root Token: hvs.AkzMQJ2dOofPjOjsLK7pzmWz
    ```


- 以下のコマンドを実行（5つのUnseal Keyの内、3つのUnseal Keyを3回に分け入力）

    ```sh
    $ vault operator unseal
    ```

## approle認証バックエンドの設定

- 以下のコマンドを実行

    ```sh
    $ vault auth enable approle
    ```

    - 結果　（エラー）

        ```sh
        Error enabling approle auth: Error making API request.

        URL: POST http://10.1.1.200:8200/v1/sys/auth/approle
        Code: 400. Errors:

        * path is already in use at approle/
        ```

    ```sh
    $ vault write auth/approle/role/concourse policies=concourse period=1h 
    ```

    - 結果

        ```sh
        Success! Data written to: auth/approle/role/concourse
        ```

    ```sh
    $ vault read auth/approle/role/concourse/role-id
    ```

    - 結果　（role_idの値は変わらず）

        ```sh
        Key        Value
        ---        -----
        role_id    2bf6997c-c123-5e25-3e22-ebe3c539ff16 
        ```

    ```sh
    $ vault write -f auth/approle/role/concourse/secret-id
    ```

    - 結果　（secret_i が変わった）

        ```sh
        Key                   Value
        ---                   -----
        secret_id             18cbe4e2-27da-bb78-18d2-9b0cd47f8f6c
        secret_id_accessor    b372ad41-6fd7-6777-e028-10bda03c8e82
        secret_id_num_uses    0
        secret_id_ttl         0s 
        ```

## Concourse を起動

- 先程のトークンをconcourseCIに設定します！docker-composeファイルに以下を追記するだけです。
- To configure this, first configure the URL of your Vault server by setting the following env on the web node

    ```sh
    $ cd ~/concourse-install
    ```

 - `docker-compose.yml`の修正内容

    ```diff
    $ diff -u docker-compose.yml_old docker-compose.yml
    --- docker-compose.yml_old      2023-09-10 14:11:31.649333867 +0900
    +++ docker-compose.yml  2023-09-10 14:12:28.547639661 +0900
    @@ -30,7 +30,7 @@
        CONCOURSE_MAIN_TEAM_LOCAL_USER: test
        CONCOURSE_VAULT_URL: http://10.1.1.200:8200
        CONCOURSE_VAULT_AUTH_BACKEND: "approle"
    -      CONCOURSE_VAULT_AUTH_PARAM: "role_id:2bf6997c-c123-5e25-3e22-ebe3c539ff16,secret_id:f9189289-baf8-368f-088a-c8c14bd484ea"
    +      CONCOURSE_VAULT_AUTH_PARAM: "role_id:2bf6997c-c123-5e25-3e22-ebe3c539ff16,secret_id:18cbe4e2-27da-bb78-18d2-9b0cd47f8f6c"
    
        logging:
        driver: "json-file"
    ```

    ```sh
    version: '3'

    services:
    db:
        image: postgres
        environment:
        POSTGRES_DB: concourse
        POSTGRES_USER: concourse_user
        POSTGRES_PASSWORD: concourse_pass
        logging:
        driver: "json-file"
        options:
            max-file: "5"
            max-size: "10m"

    web:
        image: concourse/concourse
        command: web
        links: [db]
        depends_on: [db]
        ports: ["8080:8080"]
        volumes: ["./keys/web:/concourse-keys"]
        environment:
        CONCOURSE_EXTERNAL_URL: http://localhost:8080
        CONCOURSE_POSTGRES_HOST: db
        CONCOURSE_POSTGRES_USER: concourse_user
        CONCOURSE_POSTGRES_PASSWORD: concourse_pass
        CONCOURSE_POSTGRES_DATABASE: concourse
        CONCOURSE_ADD_LOCAL_USER: test:test
        CONCOURSE_MAIN_TEAM_LOCAL_USER: test
        CONCOURSE_VAULT_URL: http://10.1.1.200:8200
        CONCOURSE_VAULT_AUTH_BACKEND: "approle"
        CONCOURSE_VAULT_AUTH_PARAM: "role_id:2bf6997c-c123-5e25-3e22-ebe3c539ff16,secret_id:18cbe4e2-27da-bb78-18d2-9b0cd47f8f6c"

        logging:
        driver: "json-file"
        options:
            max-file: "5"
            max-size: "10m"

    worker:
        image: concourse/concourse
        command: worker
        privileged: true
        depends_on: [web]
        volumes: ["./keys/worker:/concourse-keys"]
        links: [web]
        stop_signal: SIGUSR2
        environment:
        CONCOURSE_TSA_HOST: web:2222
        # enable DNS proxy to support Docker's 127.x.x.x DNS server
        CONCOURSE_GARDEN_DNS_PROXY_ENABLE: "true"
        logging:
        driver: "json-file"
        options:
            max-file: "5"
            max-size: "10m"
       ```

    ```sh
    $ cd ~/concourse-install/
    ```

    ```sh
    $ docker-compose down
    [+] Running 4/4
    ✔ Container concourse-install-worker-1  Removed                                                         10.3s 
    ✔ Container concourse-install-web-1     Removed                                                         10.4s 
    ✔ Container concourse-install-db-1      Removed                                                          0.3s 
    ✔ Network concourse-install_default     Removed                                                          0.3s 
    ```

    ```sh
    $ docker-compose up -d
    [+] Running 4/4
    ✔ Network concourse-install_default     Created                                                          0.2s 
    ✔ Container concourse-install-db-1      Started                                                          0.1s 
    ✔ Container concourse-install-web-1     Started                                                          0.2s 
    ✔ Container concourse-install-worker-1  Started                                                          0.1s 
    ```

## Concourse と Vault　の接続テスト

- 以下のコマンドを実行

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p push-docker-image -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-tutorial/tutorials/miscellaneous/docker-images/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p push-docker-image -c pipeline.yml -n
    ```

    - 結果

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=46535

        or enter token manually (input hidden): 
        target saved
        [root@control-plane ~/concourse-install (main *)]
        # fly -t tutorial destroy-pipeline -p push-docker-image -n
        !!! this will remove all data for pipeline `push-docker-image`

        `push-docker-image` does not exist
        [root@control-plane ~/concourse-install (main *)]
        # cd ~/concourse-tutorial/tutorials/miscellaneous/docker-images/
        [root@control-plane ~/concourse-tutorial/tutorials/miscellaneous/docker-images (master)]
        # fly -t tutorial set-pipeline -p push-docker-image -c pipeline.yml -n
        resources:
        resource tutorial has been added:
        + name: tutorial
        + source:
        +   branch: develop
        +   uri: https://github.com/drnic/concourse-tutorial.git
        + type: git
        
        resource hello-world-docker-image has been added:
        + name: hello-world-docker-image
        + source:
        +   email: ((docker-hub-email))
        +   password: ((docker-hub-password))
        +   repository: ((docker-hub-username))/concourse-tutorial-hello-world
        +   username: ((docker-hub-username))
        + type: docker-image
        
        jobs:
        job publish has been added:
        + name: publish
        + plan:
        + - get: tutorial
        + - params:
        +     build: tutorial/tutorials/miscellaneous/docker-images/docker
        +   put: hello-world-docker-image
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: ((docker-hub-username))/concourse-tutorial-hello-world
        +       type: docker-image
        +     params:
        +       NAME: ((docker-hub-username))
        +     platform: linux
        +     run:
        +       path: /bin/hello-world
        +   task: run
        + public: true
        
        pipeline name: push-docker-image

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/push-docker-image

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p push-docker-image
        - click play next to the pipeline in the web ui
        ```

- パイプラインのリソースをチェック


    ```
    $ fly -t tutorial check-resource -r push-docker-image/tutorial
    ```


    ```sh
    $ fly -t tutorial check-resource -r push-docker-image/hello-world-docker-image
    ```

    - 結果（成功）
    
        ```
        checking push-docker-image/hello-world-docker-image in build 1
        initializing check: hello-world-docker-image
        selected worker: 97ad8c1afbe6
        succeeded
        ```

    - <span style="color: red; ">つまり、Vault を再起動した場合、`secret_id`を再取得する必要がある</span>

- パイプラインを実行

    ```sh
    $ fly -t tutorial unpause-pipeline -p push-docker-image
    ```

    - 結果

        ```sh
        unpaused 'push-docker-image'
        ```

    ```sh
    $ fly -t tutorial trigger-job -j push-docker-image/publish -w
    ```

    - 結果（成功）


        ```
        # fly -t tutorial trigger-job -j push-docker-image/publish -w
        started push-docker-image/publish #1

        selected worker: 97ad8c1afbe6
        Cloning into '/tmp/build/get'...
        a3edcb3 restrict mkdocs packages until can make time to upgrade https://ci2.starkandwayne.com/teams/starkandwayne/pipelines/concourse-tutorial/jobs/website-master/builds/32
        selected worker: 97ad8c1afbe6
        waiting for docker to come up...
        WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
        Configure a credential helper to remove this warning. See
        https://docs.docker.com/engine/reference/commandline/login/#credentials-store

        Login Succeeded
        DEPRECATED: The legacy builder is deprecated and will be removed in a future release.
                    BuildKit is currently disabled; enable it by removing the DOCKER_BUILDKIT=0
                    environment-variable.

        Sending build context to Docker daemon  3.072kB
        Step 1/4 : FROM busybox
        latest: Pulling from library/busybox
        3f4d90098f5b: Pulling fs layer
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        3f4d90098f5b: Pull complete
        Digest: sha256:3fbc632167424a6d997e74f52b878d7cc478225cffac6bc977eedfe51c7f4e79
        Status: Downloaded newer image for busybox:latest
        ---> a416a98b71e2
        Step 2/4 : ADD hello-world /bin/hello-world
        ---> 423d982bc7fc
        Step 3/4 : ENV NAME=world
        ---> Running in 7603a5f6e8c7
        Removing intermediate container 7603a5f6e8c7
        ---> bdff54222c21
        Step 4/4 : ENTRYPOINT ["/bin/hello-world"]
        ---> Running in e06dfb8d77d1
        Removing intermediate container e06dfb8d77d1
        ---> f79bba1e27cc
        Successfully built f79bba1e27cc
        Successfully tagged kuzumusen/concourse-tutorial-hello-world:latest
        The push refers to repository [docker.io/kuzumusen/concourse-tutorial-hello-world]
        32dbb34f221d: Preparing
        3d24ee258efc: Preparing
        3d24ee258efc: Layer already exists
        32dbb34f221d: Pushed
        latest: digest: sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c size: 735
        selected worker: 97ad8c1afbe6
        waiting for docker to come up...
        WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
        Configure a credential helper to remove this warning. See
        https://docs.docker.com/engine/reference/commandline/login/#credentials-store

        Login Succeeded
        Pulling kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c...
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c: Pulling from kuzumusen/concourse-tutorial-hello-world
        3f4d90098f5b: Pulling fs layer
        a102f9d70bf2: Pulling fs layer
        a102f9d70bf2: Verifying Checksum
        a102f9d70bf2: Download complete
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        3f4d90098f5b: Pull complete
        a102f9d70bf2: Pull complete
        Digest: sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c
        Status: Downloaded newer image for kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c

        Successfully pulled kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c.

        initializing
        initializing check: image
        selected worker: 97ad8c1afbe6
        selected worker: 97ad8c1afbe6
        waiting for docker to come up...
        Pulling kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c...
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c: Pulling from kuzumusen/concourse-tutorial-hello-world
        3f4d90098f5b: Pulling fs layer
        a102f9d70bf2: Pulling fs layer
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        a102f9d70bf2: Verifying Checksum
        a102f9d70bf2: Download complete
        3f4d90098f5b: Pull complete
        a102f9d70bf2: Pull complete
        Digest: sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c
        Status: Downloaded newer image for kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c
        docker.io/kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c

        Successfully pulled kuzumusen/concourse-tutorial-hello-world@sha256:410db690245040304d96d6fe1b7c5cfba9d587951af1473945e5fd33b4f7af0c.

        selected worker: 97ad8c1afbe6
        running /bin/hello-world
        hello kuzumusen
        succeeded
        ```

## ビルドの成果物をアップロードする

- [公式のチュートリアルサイト](https://concoursetutorial-ja.site.lkj.io/basics/publishing-outputs)では、`pipeline.yml`に秘密鍵を直書きしているが、秘密鍵をvaultで管理するようにする

- 参考にしたサイトは以下

    - [公式のチュートリアルサイト](https://concoursetutorial-ja.site.lkj.io/basics/publishing-outputs)

    - https://gammalab.net/blog/3f7pudgk4zbcr/

### `pipeline.yml` の修正

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/basic/publishing-outputs/    
    ```

    ```diff
    $ diff -u pipeline.yml_old pipeline.yml
    --- pipeline.yml_old    2023-09-10 15:07:50.284951802 +0900
    +++ pipeline.yml        2023-09-10 15:17:59.549318976 +0900
    @@ -10,35 +10,8 @@
        source:
        # branch: master
        branch: main
    -      uri: git@gist.github.com:5e40d8df48fa30baf9abee4b35a2241a.git
    -      private_key: |
    -        -----BEGIN RSA PRIVATE KEY-----
    -        MIIEowIBAAKCAQEAp/Tcj0IhF9PG27xq6dH4G72Nqns0KjaFnBABLEZty8DtFiaS
    -        oESMZgrsrEJe36l1kaE9i/3UWKYrU5LZsDvn8DVc4Y5U0IE/QcmsplH5G0zSA0VJ
    -        VRLB4higtJV4JPXKyjS1MBG5pLAD7pUbQ+jBj3a6icsHEuSGAfssY2lpSj8UOUwd
    -        fmnPaaXmyY3x1K1KuaggQ8mLOJNPAJ4x2jar+bap0URlggbu2kFQ4/XakDZVCRxT
    -        JDybRM37RwrSqLYspnOsaJ18z2EJCuIaiVcySyz7DjVcPTdXQzjkWw8nYPX5IykN
    -        zHfHC6qgCcN4AJlgFZl1F9xisXnD+B5X5R4LWwIDAQABAoIBAAQc/g3QG8lemV8m
    -        RSQGzWG4ibCkJcnm3ezNg4nXC7dSuTuypCKiqyGQoO0zDunBV6zCWySDieDF6Qe5
    -        7/Td8rcyR10KxE7661asHrtQBJ7Did0kpEAeHntwCPeDNZcKIfZDxjAwLvC2ktIT
    -        +r/2Ak+GI9leDIVM7W88/IBOw5Ja4POYZUpJ+v6ym3kEA+Xgrw5ej0+pOO2NmTFL
    -        pujQd0P1MnK9kaftRPBg9OlFpq0wjViLDPZKcb9XzHBx6LYHqE2IG7P93b8hMNqP
    -        A8HAcH8S1h7k3nWT20x/eTVWFTNtGoAsO31nz6QH+E6JbFs6ApNpiON30HkDSFX5
    -        wnhbsikCgYEA2gCzC01cJSdzp/yZexelM99ivpwEmytE6R4qBemiNAcRwmKTYSeA
    -        ct3AnxPBj1vIaTgwtq9P/JBEPQMonhTWq3bj1v+70HdBz50njd0Rl58zC9rx6nF4
    -        2zNOTga3EcmQ2EyeZjxFVSs8v1YtHgT1OUjkcFnS2paBrKkiLhd38WcCgYEAxTsY
    -        NkoNTQBitORS5wtp39Dr87k6tyVOmRn8MZuBeaKAJ2cIXFHDwRtNm5so+jQS63td
    -        QVOu4wF9fxBPHrxT4z/D9ASfrmsc1Z7gTRGu/sHGYmdDOomAI6RKBBjNtfgBIekW
    -        HNrLt+xLXxr8xL7QC8bJtKwPmigWVzo8DWSrme0CgYACrKeFp/lNa2J72Rl47R1V
    -        uZPYislzreA2i+wwDmGzCbMqE1ODiZyFzDqkuPVS8OlQgSP32ca9bnen1/YTmmXX
    -        zKmW5aRENnJUPbVShDfHCGjz6Ee3fJTi+4omYua0DSj9vlLjJjIjjVg9cK01BRKN
    -        FVvYFQIFNHt6xshokFkkWQKBgQCYVS48MDHZuWSDhp4paX1aqwiy8+vPrPbp9VH+
    -        FreH9OS6ii/A7j4dljL47nxV04aRbnT2keXP20TMsRILETZRnNyCSlfy5TQeIlnn
    -        7LKWfZ/2PP+F5NGdtbSdOXMZCvYE9PxpSOxzoAQO7s8wPph9oAoGi6Z5UGEA+i+L
    -        wKdxeQKBgDqFNCOKd50GzA5tavw06AzzsHzYc1PoJBIw3NphYkJimxPbI8RBkkCa
    -        HsMd73mr7LrLgzZWj7gBD5GXzBqHDdm+/8u4ev3dhquGeBprFJI8E7K7TEi2Spwz
    -        rm8Hl2L077bV+VOTb177RetIFLt/dY8KREWC72vvqxVIzalhjDnG
    -        -----END RSA PRIVATE KEY-----
    +      uri: git@gist.github.com:8becd8bea76a7442d6a6c2d46a49f0b1.git
    +      private_key: ((private-key))
    
    jobs:
      - name: job-bump-date    
    ```

## 秘密鍵をvaultに登録

- 以下のコマンドを実行

    ```sh
    $ cat ~/.ssh/id_rsa | vault kv put concourse/main/private-key value=-
    ```

    - 結果

        ```sh
        Success! Data written to: concourse/main/private-key
        ```

## パイプラインの作衛

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/basic/publishing-outputs
    ```

- パイプラインの作成
    ```sh
    fly -t tutorial set-pipeline -p publishing-outputs -c pipeline.yml -n
    ```

    - 結果

        ```sh
        resources:
        resource resource-tutorial has been added:
        + name: resource-tutorial
        + source:
        +   branch: develop
        +   uri: https://github.com/starkandwayne/concourse-tutorial.git
        + type: git
        
        resource resource-gist has been added:
        + name: resource-gist
        + source:
        +   branch: main
        +   private_key: ((private-key))
        +   uri: git@gist.github.com:8becd8bea76a7442d6a6c2d46a49f0b1.git
        + type: git
        
        jobs:
        job job-bump-date has been added:
        + name: job-bump-date
        + plan:
        + - get: resource-tutorial
        + - get: resource-gist
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: getourneau/alpine-bash-git
        +       type: docker-image
        +     inputs:
        +     - name: resource-tutorial
        +     - name: resource-gist
        +     outputs:
        +     - name: updated-gist
        +     platform: linux
        +     run:
        +       path: resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        +   task: bump-timestamp-file
        + - params:
        +     repository: updated-gist
        +   put: resource-gist
        + serial: true
        
        pipeline name: publishing-outputs

        apply configuration? [yN]: y
        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/publishing-outputs

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p publishing-outputs
        - click play next to the pipeline in the web ui
        ```

- リソース（秘密鍵）チェック → 成功

    ```sh
    $ fly -t tutorial check-resource -r publishing-outputs/resource-gist
    ```

    - 結果

        ```
        checking publishing-outputs/resource-gist in build 60
        initializing check: resource-gist
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/git-resource-repo-cache'...
        succeeded
        ```

    ```sh
    $ fly -t tutorial unpause-pipeline -p publishing-outputs
    ```
    
    ```sh
    $ fly -t tutorial trigger-job -j publishing-outputs/job-bump-date -w
    ```

    - 結果（成功）

        ```sh
        started publishing-outputs/job-bump-date #1

        selected worker: 97ad8c1afbe6
        Cloning into '/tmp/build/get'...
        a3edcb3 restrict mkdocs packages until can make time to upgrade https://ci2.starkandwayne.com/teams/starkandwayne/pipelines/concourse-tutorial/jobs/website-master/builds/32
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        43e49c9 
        initializing
        initializing check: image
        selected worker: 97ad8c1afbe6
        selected worker: 97ad8c1afbe6
        waiting for docker to come up...
        Pulling getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7...
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7: Pulling from getourneau/alpine-bash-git
        4fe2ade4980c: Pulling fs layer
        03c196859ec8: Pulling fs layer
        720d2de11875: Pulling fs layer
        4fe2ade4980c: Verifying Checksum
        4fe2ade4980c: Download complete
        720d2de11875: Verifying Checksum
        720d2de11875: Download complete
        4fe2ade4980c: Pull complete
        03c196859ec8: Verifying Checksum
        03c196859ec8: Download complete
        03c196859ec8: Pull complete
        720d2de11875: Pull complete
        Digest: sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        Status: Downloaded newer image for getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7
        docker.io/getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7

        Successfully pulled getourneau/alpine-bash-git@sha256:246ebea4839401a027da43e406a0ceaf0f763997a516cf85c344425eb913ffe7.

        selected worker: 97ad8c1afbe6
        running resource-tutorial/tutorials/basic/publishing-outputs/bump-timestamp-file.sh
        + git clone resource-gist updated-gist
        Cloning into 'updated-gist'...
        done.
        + cd updated-gist
        + date
        + echo Sun Sep 10 06:58:57 UTC 2023
        + git config --global user.email nobody@concourse-ci.org
        + git config --global user.name Concourse
        + git add .
        + git commit -m 'Bumped date'
        [main 1811c86] Bumped date
        1 file changed, 1 insertion(+), 1 deletion(-)
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        To gist.github.com:8becd8bea76a7442d6a6c2d46a49f0b1.git
        43e49c9..1811c86  HEAD -> main
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        1811c86 Bumped date
        succeeded
        ```


## Concourse CI での SemVer - セマンティックバージョニング(GtiHub上で管理のバージョンファイルをConcourse CIに更新させる)

- 以下のサイトを参考とした

     [公式のチュートリアル](https://concoursetutorial-ja.site.lkj.io/miscellaneous/versions-and-buildnumbers)

     [公式サイト（concourse のgit の SemVer )](https://github.com/concourse/semver-resource)

     [gitドライバを使用した例](https://stackoverflow.com/questions/55402179/automate-semver-in-concourseci)

## バージョンを表示するパイプラインの作成

- 以下のコマンドを作成

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

- `pipeline-display-version.yml`を作成

    ```sh
    $ cat pipeline-display-version.yml
    ---
    resources:
    - name: version
        type: semver
        source:
        driver: git
        initial_version: 0.0.1
        uri: https://github.com/moriyamaES/semver-test.git
        branch: main
        file: number
        private_key: ((private-key))

    jobs:
    - name: display-version
        public: true
        serial: true
        plan:
        - get: version
        - task: display-version
            config:
            platform: linux
            image_resource:
                type: docker-image
                source: {repository: busybox}
            inputs:
                - name: version
            run:
                path: cat
                args: [version/number]
    ```

- パイプラインの削除

    ```
    $ fly -t tutorial destroy-pipeline -p versions-and-buildnumbers -n
    ```

- パイプラインの作成

    ```
    $ fly -t tutorial set-pipeline -p versions-and-buildnumbers -c pipeline-display-version.yml -n
    ```

    - 結果

        ```sh
        resources:
        resource version has been added:
        + name: version
        + source:
        +   branch: main
        +   driver: git
        +   file: number
        +   initial_version: 0.0.1
        +   private_key: ((private-key))
        +   uri: https://github.com/moriyamaES/semver-test.git
        + type: semver
        
        jobs:
        job display-version has been added:
        + name: display-version
        + plan:
        + - get: version
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: busybox
        +       type: docker-image
        +     inputs:
        +     - name: version
        +     platform: linux
        +     run:
        +       args:
        +       - version/number
        +       path: cat
        +   task: display-version
        + public: true
        + serial: true
        
        pipeline name: versions-and-buildnumbers

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/versions-and-buildnumbers

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
        - click play next to the pipeline in the web ui
        ```
ｑ
- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r versions-and-buildnumbers/version
    ```

    - 結果

        ```sh
        checking versions-and-buildnumbers/version in build 230
        initializing check: version
        selected worker: 97ad8c1afbe6
        Cloning into '/tmp/semver-git-repo'...
        HEAD is now at b231816 Add number file
        succeeded
        ```

- パイプラインの実行
    ```sh
    fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
    ```

    
    ```sh
    fly -t tutorial trigger-job -j versions-and-buildnumbers/display-version -w
    ```

    - 結果（成功）

        ```sh
        checking versions-and-buildnumbers/version in build 230
        initializing check: version
        selected worker: 97ad8c1afbe6
        Cloning into '/tmp/semver-git-repo'...
        HEAD is now at b231816 Add number file
        succeeded
        [root@control-plane ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers (main *+)]
        # fly -t bucc trigger-job -j versions-and-buildnumbers/display-version -w
        error: unknown target: bucc
        [root@control-plane ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers (main *+)]
        # fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
        unpaused 'versions-and-buildnumbers'
        [root@control-plane ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers (main *+)]
        # fly -t tutorial trigger-job -j versions-and-buildnumbers/display-version -w
        started versions-and-buildnumbers/display-version #1

        selected worker: 97ad8c1afbe6
        initializing
        initializing check: image
        selected worker: 97ad8c1afbe6
        selected worker: 97ad8c1afbe6
        waiting for docker to come up...
        Pulling busybox@sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee...
        docker.io/library/busybox@sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee: Pulling from library/busybox
        3f4d90098f5b: Pulling fs layer
        3f4d90098f5b: Verifying Checksum
        3f4d90098f5b: Download complete
        3f4d90098f5b: Pull complete
        Digest: sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee
        Status: Downloaded newer image for busybox@sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee
        docker.io/library/busybox@sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee

        Successfully pulled busybox@sha256:023917ec6a886d0e8e15f28fb543515a5fcd8d938edb091e8147db4efed388ee.

        selected worker: 97ad8c1afbe6
        running cat version/number
        0.0.1succeeded
        ```

- 【結論】`type: semver` のリソースを使えば、GitHub上のファイルで管理するバージョン番号が参照できることが分かった。しかし、バージョン番号の更新方法が分からない。

## バージョンを更新するパイプラインの作成

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

- `pipeline-bump-then-save.yml`の作成

    ```sh
    $ cat pipeline-bump-then-save.yml
    ---
    resources:
    - name: version
        type: semver
        source:
        driver: git
        initial_version: 0.0.1
        uri: https://github.com/moriyamaES/semver-test.git
        branch: main
        file: number
        private_key: ((private_key))

    jobs:
    - name: bump-version
        public: true
        serial: true
        plan:
        - get: semver-test
            trigger: true
        - get: version
            params: {bump: patch}
        - put: version
            params: {file: version/number}
        - task: display-version
            config:
            platform: linux
            image_resource:
                type: docker-image
                source: {repository: busybox}
            inputs:
                - name: version
            run:
                path: cat
                args: [version/number]
    ```

- パイプラインの作成

    ```
    $ fly -t tutorial set-pipeline -p versions-and-buildnumbers -c pipeline-bump-then-save.yml -n
    ```

    - 結果

        ```sh
        jobs:
        job display-version has been removed:
        - name: display-version
        - plan:
        - - get: version
        - - config:
        -     image_resource:
        -       name: ""
        -       source:
        -         repository: busybox
        -       type: docker-image
        -     inputs:
        -     - name: version
        -     platform: linux
        -     run:
        -       args:
        -       - version/number
        -       path: cat
        -   task: display-version
        - public: true
        - serial: true
        
        job bump-version has been added:
        + name: bump-version
        + plan:
        + - get: semver-test
        +   trigger: true
        + - get: version
        +   params:
        +     bump: patch
        + - params:
        +     file: version/number
        +   put: version
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: busybox
        +       type: docker-image
        +     inputs:
        +     - name: version
        +     platform: linux
        +     run:
        +       args:
        +       - version/number
        +       path: cat
        +   task: display-version
        + public: true
        + serial: true
        
        pipeline name: versions-and-buildnumbers

        error: invalid pipeline config:
        invalid jobs:
                jobs.bump-version.plan.do[0].get(semver-test): unknown resource 'semver-test'
        ```
ｑ
- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r versions-and-buildnumbers/version
    ```

    - 結果

        ```sh
        checking versions-and-buildnumbers/version in build 302
        initializing check: version
        selected worker: 97ad8c1afbe6
        From https://github.com/moriyamaES/semver-test
        * branch            main       -> FETCH_HEAD
        HEAD is now at b231816 Add number file
        succeeded
        ```

- パイプラインの実行
    ```sh
    fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
    ```
    
    ```sh
    fly -t tutorial trigger-job -j versions-and-buildnumbers/bump-version -w
    ```

    - 結果（成功）

        ```sh
        ```

- 【問題発生！！】`type: semver` のリソースでは、GitHub上のファイルで管理するバージョン番号が更新できない！！


## 上手くいかないので、ChatGPTに聞いてみた


- ChatGPTは2つの案を示してくれた

- （案１）`pipeline.yml` とし、`fly set-pipeline`コマンドを以下とする

    ```sh
    resources:
      - name: source-code
        type: git
        source:
        uri: https://github.com/your/repo.git
        branch: main
        private_key: ((private-key))  # GitHub認証などが必要な場合

    jobs:
    - name: bump-version
    plan:
    - get: source-code
    - task: bump-version-task
        config:
        platform: linux
        image_resource:
            type: docker-image
            source:
            repository: busybox
        inputs:
            - name: source-code
        run:
            path: /bin/sh
            args:
            - -exc
            - |
                # 現在のバージョンを取得
                current_version=$(cat source-code/version/number)
                
                # バージョンをバンプするタイプを指定
                bump_type=${bump_type:-"major"}  # デフォルトはメジャーバージョンをバンプ
                
                # current_version が空文字列の場合、適切な初期バージョンを設定
                if [ -z "$current_version" ]; then
                if [ "$bump_type" = "major" ]; then
                    new_version="1.0.0"
                elif [ "$bump_type" = "minor" ]; then
                    new_version="0.1.0"
                elif [ "$bump_type" = "patch" ]; then
                    new_version="0.0.1"
                else
                    echo "Invalid bump_type specified."
                    exit 1
                fi
                else
                # バージョンをバンプするロジック
                IFS='.' read -r -a version_parts <<< "$current_version"
                major="${version_parts[0]}"
                minor="${version_parts[1]}"
                patch="${version_parts[2]}"
                
                if [ "$bump_type" = "major" ]; then
                    major=$((major + 1))
                    minor=0
                    patch=0
                elif [ "$bump_type" = "minor" ]; then
                    minor=$((minor + 1))
                    patch=0
                elif [ "$bump_type" = "patch" ]; then
                    patch=$((patch + 1))
                fi
                
                new_version="$major.$minor.$patch"
                fi
                
                # 新しいバージョンをファイルに書き込む
                echo "$new_version" > source-code/version/number
        outputs:
            - name: source-code
    ```

    ```sh
    $ fly -t your-target set-pipeline -p your-pipeline-name -c pipeline.yml -v bump_type=minor
    ```

    ```
    jobs:
    - name: bump-version
    plan:
    - get: source-code
    - task: bump-version-task
        config:
        platform: linux
        image_resource:
            type: docker-image
            source:
            repository: busybox
        inputs:
            - name: source-code
        run:
            path: /bin/sh
            args:
            - -exc
            - |
                # 現在のバージョンを取得
                current_version=$(cat source-code/version/number)
                
                # バージョンをバンプするタイプを指定
                bump_type=${bump_type:-"major"}  # デフォルトはメジャーバージョンをバンプ
                
                # current_version が空文字列の場合、適切な初期バージョンを設定
                if [ -z "$current_version" ]; then
                if [ "$bump_type" = "major" ]; then
                    new_version="1.0.0"
                elif [ "$bump_type" = "minor" ]; then
                    new_version="0.1.0"
                elif [ "$bump_type" = "patch" ]; then
                    new_version="0.0.1"
                else
                    echo "Invalid bump_type specified."
                    exit 1
                fi
                else
                # バージョンをバンプするロジック
                IFS='.' read -r -a version_parts <<< "$current_version"
                major="${version_parts[0]}"
                minor="${version_parts[1]}"
                patch="${version_parts[2]}"
                
                if [ "$bump_type" = "major" ]; then
                    major=$((major + 1))
                    minor=0
                    patch=0
                elif [ "$bump_type" = "minor" ]; then
                    minor=$((minor + 1))
                    patch=0
                elif [ "$bump_type" = "patch" ]; then
                    patch=$((patch + 1))
                fi
                
                new_version="$major.$minor.$patch"
                fi
                
                # 新しいバージョンをファイルに書き込む
                echo "$new_version" > source-code/version/number
                
                # 変更をコミット
                cd source-code
                git config --global user.email "you@example.com"
                git config --global user.name "Your Name"
                git add version/number
                git commit -m "Bump version to $new_version"
                git push origin main
        outputs:
            - name: source-code
    
    ```

- 上記をヒントに パイプラインのYAMLを作成することにした。

- 上手くいかない記事は取消線を引く

<del>

## GitHub用のバージョンを更新するパイプラインの作成

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

- `pipeline-bump-then-save-at-git.yml`の作成

    ```sh
    $ cat pipeline-bump-then-save-at-git.yml
    ---
    resources:
      - name: source-code
        type: git
        source:
        uri: https://github.com/moriyamaES/semver-test.git
        branch: main
        private_key: ((private_key))

    jobs:
      - name: bump-version
        public: true
        serial: true
        plan:
          - get: source-code
            trigger: true
          - task: bump-version-task
            config:
              platform: linux
              image_resource:
                type: docker-image
                source:
                  repository: busybox
                 inputs:
                - name: source-code
                run:
                  path: /bin/sh
                  args:
                    - -exc
                    - |
                      # 現在のバージョンを取得
                      current_version=$(cat source-code/version/number)
                      
                      # バージョンをバンプするタイプを指定
                      bump_type=${bump_type:-"major"}  # デフォルトはメジャーバージョンをバンプ
                      
                      # current_version が空文字列の場合、適切な初期バージョンを設定
                      if [ -z "$current_version" ]; then
                         if [ "$bump_type" = "major" ]; then
                           new_version="1.0.0"
                         elif [ "$bump_type" = "minor" ]; then
                           new_version="0.1.0"
                         elif [ "$bump_type" = "patch" ]; then
                           new_version="0.0.1"
                         else
                           echo "Invalid bump_type specified."
                           exit 1
                         fi
                      else
                        # バージョンをバンプするロジック
                        IFS='.' read -r -a version_parts <<< "$current_version"
                        major="${version_parts[0]}"
                        minor="${version_parts[1]}"
                        patch="${version_parts[2]}"
                        
                        if [ "$bump_type" = "major" ]; then
                          major=$((major + 1))
                          minor=0
                          patch=0
                        elif [ "$bump_type" = "minor" ]; then
                          minor=$((minor + 1))
                          patch=0
                        elif [ "$bump_type" = "patch" ]; then
                          patch=$((patch + 1))
                        fi
                
                        new_version="$major.$minor.$patch"
                      fi
                
                      # 新しいバージョンをファイルに書き込む
                      echo "$new_version" > source-code/version/number
                
                      # 変更をコミット
                      cd source-code
                      git config --global user.email "you@example.com"
                      git config --global user.name "Your Name"
                      git add version/number
                      git commit -m "Bump version to $new_version"
                      git push origin main
          - task: display-version
            config:
              platform: linux
              image_resource:
                source:
                  repository: busybox
                inputs:
                  - name: source-code
                run:
                  path: cat
                  args: [version/number]
    ```


- concourse Web UIにログインする

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p versions-and-buildnumbers -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p versions-and-buildnumbers -c pipeline-bump-then-save-at-git.yml -v bump_type=minor -n
    ```

- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r versions-and-buildnumbers/source-code
    ```

- パイプラインの実行
    ```sh
    fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
    ```
    
    ```sh
    fly -t tutorial trigger-job -j versions-and-buildnumbers/bump-version -w
    ```

## 上手くいかないのでやり直し

- 以下のコマンドを実行

    ```sh
    # pwd
    /root/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers
    ```

- パイプライン用のyamlは、以下とする

    ```sh
    $ cat pipeline.yml
    ---
    resources:
      - name: resource-gist
          type: git
          source:
            branch: main
            uri: https://github.com/moriyamaES/semver-test.git
            private_key: ((private-key))

    jobs:
    - name: job-bump-date
        serial: true
        plan:
        - get: resource-gist
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                source: {repository: getourneau/alpine-bash-git}

            inputs:
                - name: resource-gist
            outputs:
                - name: updated-gist
            run:
                path: /bin/sh
                args:
                - -exc
                - |
                    git clone resource-gist updated-gist
                    cd updated-gist
                    date > version/number
                    git config --global user.email "nobody@concourse-ci.org"
                    git config --global user.name "Concourse"
                    git add .
                    git commit -m "Bumped date"
        - put: resource-gist
            params:
            repository: updated-gist
    ```

- concourse Web UIにログインする

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作
    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p versions-and-buildnumbers -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p versions-and-buildnumbers -c  pipeline.yml -n
    ```

    - 結果

        ```sh
        resources:
        resource resource-gist has been added:
        + name: resource-gist
        + source:
        +   branch: main
        +   private_key: ((private-key))
        +   uri: https://github.com/moriyamaES/semver-test.git
        + type: git
        
        jobs:
        job job-bump-date has been added:
        + name: job-bump-date
        + plan:
        + - get: resource-gist
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: getourneau/alpine-bash-git
        +       type: docker-image
        +     inputs:
        +     - name: resource-gist
        +     outputs:
        +     - name: updated-gist
        +     platform: linux
        +     run:
        +       args:
        +       - -exc
        +       - |
        +         git clone resource-gist updated-gist
        +         cd updated-gist
        +         date > version/number
        +         git config --global user.email "nobody@concourse-ci.org"
        +         git config --global user.name "Concourse"
        +         git add .
        +         git commit -m "Bumped date"
        +       path: /bin/sh
        +   task: bump-timestamp-file
        + - params:
        +     repository: updated-gist
        +   put: resource-gist
        + serial: true
        
        pipeline name: versions-and-buildnumbers

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/versions-and-buildnumbers

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
        - click play next to the pipeline in the web ui

        ```


- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r versions-and-buildnumbers/resource-gist
    ```

    - 結果

        ```sh
        $ fly -t tutorial check-resource -r versions-and-buildnumbers/resource-gist
        checking versions-and-buildnumbers/resource-gist in build 12691
        initializing check: resource-gist
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/git-resource-repo-cache'...
        succeeded
        ```

- パイプラインの実行
    ```sh
    $ fly -t tutorial unpause-pipeline -p versions-and-buildnumbers
    ```
    
    ```sh
    $ fly -t tutorial trigger-job -j versions-and-buildnumbers/job-bump-date -w
    ```




started versions-and-buildnumbers/job-bump-date #14

selected worker: 97ad8c1afbe6
INFO: found existing resource cache

initializing
initializing check: image
selected worker: 97ad8c1afbe6
selected worker: 97ad8c1afbe6
INFO: found existing resource cache

selected worker: 97ad8c1afbe6
running /bin/sh -exc git clone resource-gist updated-gist
cd updated-gist
# git clone resource-gist
# cd semver-test
# pwd
date > ./version/number
cat ./version/number
git config --global user.email "moriyama.kazuhiro@earthsys-lab.co.jp"
git config --global user.name "moriyamaES"
git status
# git add .
git add ./version/number
git status
git commit -m "Bumped date"
git push origin main

+ git clone resource-gist updated-gist
Cloning into 'updated-gist'...
done.
+ cd updated-gist
+ date
+ cat ./version/number
Fri Sep 15 13:11:33 UTC 2023
+ git config --global user.email moriyama.kazuhiro@earthsys-lab.co.jp
+ git config --global user.name moriyamaES
+ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

        modified:   version/number

no changes added to commit (use "git add" and/or "git commit -a")
+ git add ./version/number
+ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

        modified:   version/number

+ git commit -m 'Bumped date'
[main 60735d5] Bumped date
 1 file changed, 1 insertion(+)
+ git push origin main
Enumerating objects: 7, done.
Counting objects: 100% (7/7), done.
Delta compression using up to 2 threads.
Compressing objects: 100% (2/2), done.
Writing objects: 100% (4/4), 363 bytes | 72.00 KiB/s, done.
Total 4 (delta 0), reused 0 (delta 0)
To /tmp/build/cc4f7a9f/resource-gist
   b231816..60735d5  main -> main

</del>

## 上手くいった

- 以下のサイトを参考にした（ただし、記載ないようは中途半端だった）

    - [誰かのブログ](https://blog.lespaulstudioplus.info/posts/26/)

    - [公式サイト](https://concourse-ci.org/basic-git-operations.html)


## Concourse CI で GtiHub上のファイルに日付を記入する(上手くいった)

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

- パイプラインYAMLの内容は以下

    ```sh
    $ cat pipeline-push-to-git-repository-basic.yml
    ---
    resources:
    - name: git-repository
        type: git
        source:
        branch: main
        # sshで認証するため、uriはsshのuriに設定する必要がある？(httpsだと正常動作しない）
        # uri: https://github.com/moriyamaES/semver-test.git
        uri: git@github.com:moriyamaES/semver-test.git
        private_key: ((private-key))

    jobs:
    - name: job-bump-date
        # serial: true
        plan:
        - get: git-repository
            # trigger: true
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                source: {repository: getourneau/alpine-bash-git}

            # outputsをinputsと同じ名前にしないと正常動作しない模様
            inputs:
                - name: git-repository
            outputs:
                - name: git-repository
            run:
                path: /bin/sh
                args:
                - -exc
                - |
                    # git cloneは不要。この時点ではconcorseがgit cloneを実行してる模様。
                    cd git-repository
                    date > ./version/number
                    cat ./version/number
                    # メールアドレスは必須（ユーザ名は任意）
                    git config --global user.email "concourse@local"
                    git add ./version/number
                    git commit -m "Bumped date"
                    # git tag v0.1.6
        # このPUTで、ローカルリポジトリからリポジトリへのpushが実行されるため、git pushコマンドは不要。
        - put: git-repository
            params:
            repository: git-repository
        ```

- concourse Web UIにログインする

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作

    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p push-to-git-repository -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p push-to-git-repository -c pipeline-push-to-git-repository-basic.yml -n
    ```

    - 結果

        ```sh
        resources:
          resource git-repository has been added:
        + name: git-repository
        + source:
        +   branch: main
        +   private_key: ((private-key))
        +   uri: git@github.com:moriyamaES/semver-test.git
        + type: git
        
        jobs:
        job job-bump-date has been added:
        + name: job-bump-date
        + plan:
        + - get: git-repository
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: getourneau/alpine-bash-git
        +       type: docker-image
        +     inputs:
        +     - name: git-repository
        +     outputs:
        +     - name: git-repository
        +     platform: linux
        +     run:
        +       args:
        +       - -exc
        +       - |
        +         # git cloneは不要。この時点ではconcorseがgit cloneを実行してる模様。
        +         cd git-repository
        +         date > ./version/number
        +         cat ./version/number
        +         # メールアドレスは必須（ユーザ名は任意）
        +         git config --global user.email "concourse@local"
        +         git add ./version/number
        +         git commit -m "Bumped date"
        +         # git tag v0.1.6
        +       path: /bin/sh
        +   task: bump-timestamp-file
        + - params:
        +     repository: git-repository
        +   put: git-repository
        
        pipeline name: push-to-git-repository

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/push-to-git-repository

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p push-to-git-repository
        - click play next to the pipeline in the web ui
        
        ```

- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r push-to-git-repository/git-repository
    ```

    - 結果

        ```sh
        checking push-to-git-repository/git-repository in build 14230
        initializing check: git-repository
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        HEAD is now at 0215a39 Bump version to v4.3.0
        succeeded
        ```

- パイプラインの実行
    ```sh
    $ fly -t tutorial unpause-pipeline -p push-to-git-repository
    ```
    
    ```sh
    $ fly -t tutorial trigger-job -j push-to-git-repository/job-bump-date -w
    ```

    - 結果

        ```sh
        started push-to-git-repository/job-bump-date #1

        selected worker: 97ad8c1afbe6
        INFO: found existing resource cache

        initializing
        initializing check: image
        selected worker: 97ad8c1afbe6
        selected worker: 97ad8c1afbe6
        INFO: found existing resource cache

        selected worker: 97ad8c1afbe6
        running /bin/sh -exc # git cloneは不要。この時点ではconcorseがgit cloneを実行してる模様。
        cd git-repository
        date > ./version/number
        cat ./version/number
        # メールアドレスは必須（ユーザ名は任意）
        git config --global user.email "concourse@local"
        git add ./version/number
        git commit -m "Bumped date"
        # git tag v0.1.6

        + cd git-repository
        + date
        + cat ./version/number
        Fri Sep 15 22:40:47 UTC 2023
        + git config --global user.email concourse@local
        + git add ./version/number
        + git commit -m 'Bumped date'
        [detached HEAD e9ec4c4] Bumped date
        1 file changed, 1 insertion(+), 1 deletion(-)
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        To github.com:moriyamaES/semver-test.git
        0215a39..e9ec4c4  HEAD -> main
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        e9ec4c4 Bumped date
        succeeded
        ```

## Concourse CI で GtiHub上のファイルのバージョン番号を更新する(上手くいった)

- 以下のサイトを参考にした（ただし、記載ないようは中途半端だった）

    - [誰かのブログ](https://blog.lespaulstudioplus.info/posts/26/)

    - [公式サイト](https://concourse-ci.org/basic-git-operations.html)

- 以下のコマンドを実行

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

- パイプラインYAMLの内容は以下

    ```sh
    # cat pipeline-bump-soft-version.yml 
    ---
    resources:
    - name: git-repository
        type: git
        source:
        branch: main
        # sshで認証するため、uriはsshのuriに設定する必要がある(httpsだと正常動作しない）
        uri: git@github.com:moriyamaES/semver-test.git
        # GitHubに接続するための秘密鍵はVaultで管理
        private_key: ((private-key))
    jobs:
    - name: bump-version
        plan:
        - get: git-repository
        - task: bump-timestamp-file
            config:
            platform: linux
            image_resource:
                type: docker-image
                # bashとgitが実行できるコンテナをpull
                source: {repository: getourneau/alpine-bash-git}
            # nputsとoutputsは、同じ名前にしないと正常動作しない模様
            inputs:
                - name: git-repository
            outputs:
                - name: git-repository
            run:
                path: /bin/sh
                args:
                - -exc
                - |

                    # この時点ではconcorseがgit cloneを実行してるため、git cloneの実行は不要。
                    # gitのローカルリポジトリのディレクトリに移動
                    cd git-repository

                    # 現在のバージョンを取得
                    current_version=$(cat ./version/number)
                    
                    # バージョンをバンプするタイプを指定
                    BUMP_TYPE=${BUMP_TYPE:-"major"}  # デフォルトはメジャーバージョンをバンプ
                    
                    # current_version が空文字列の場合
                    if [ "$current_version" = "" ]; then
                        # 適切な初期バージョンを設定
                        if [ "$BUMP_TYPE" = "major" ]; then
                        new_version="1.0.0"
                        elif [ "$BUMP_TYPE" = "minor" ]; then
                        new_version="0.1.0"
                        elif [ "$BUMP_TYPE" = "patch" ]; then
                        new_version="0.0.1"
                        else
                        echo "Invalid bump_type specified."
                        exit 1
                        fi
                    else
                    # バージョンをバンプする
                    IFS_SAVE=$IFS
                    IFS='.'
                    set $current_version
                    IFS=$IFS_SAVE
                    major="$1"
                    minor="$2"
                    patch="$3"
                    if [ "$BUMP_TYPE" = "major" ]; then
                        major=$((major + 1))
                        minor=0
                        patch=0
                    elif [ "$BUMP_TYPE" = "minor" ]; then
                        minor=$((minor + 1))
                        patch=0
                    elif [ "$BUMP_TYPE" = "patch" ]; then
                        patch=$((patch + 1))
                    fi
                    new_version="$major.$minor.$patch"
                    fi
            
                    # 新しいバージョンをファイルに書き込む
                    echo "$new_version" > ./version/number
                    cat ./version/number

                    # コミットしタグを付加する
                    git config --global user.email "concourse@local" # メールアドレスは必須（ユーザ名は任意）
                    git add ./version/number
                    git commit -m "Bump version to v$new_version"
                    git tag v$new_version
            params:
                BUMP_TYPE: ((bump-type))

        # このPUTで、ローカルリポジトリからリポジトリへのpushを実行するため、git pushの実行は不要。
        - put: git-repository
            params:
            repository: git-repository
    ```

- concourse Web UIにログインする

    ```sh
    $ fly --target tutorial login --concourse-url http://localhost:8080
    ```

    - 操作

    - `http://localhost:8080/login?fly_port=43269` でホストOSのブラウザにアクセスし、表示されたtokenを貼り付ける
    - ユーザID: `test`、パスワード: `test` とする
    - 上記操作をすると、Concourse CI のWeb UIにログインできる

        ```
        logging in to team 'main'

        navigate to the following URL in your browser:

        http://localhost:8080/login?fly_port=43269

        or enter token manually (input hidden): 
        target saved
        ```

- パイプラインを削除する

    ```sh
    $ fly -t tutorial destroy-pipeline -p bump-soft-minor-version -n
    ```

- パイプラインを作成

    ```sh
    $ cd ~/concourse-semver-test/tutorials/miscellaneous/versions-and-buildnumbers/
    ```

    ```sh
    $ fly -t tutorial set-pipeline -p bump-soft-minor-version -c pipeline-bump-soft-version.yml -v bump-type=minor -n
    ```

    - 結果

        ```sh
        resources:
        resource git-repository has been added:
        + name: git-repository
        + source:
        +   branch: main
        +   private_key: ((private-key))
        +   uri: git@github.com:moriyamaES/semver-test.git
        + type: git
        
        jobs:
        job bump-version has been added:
        + name: bump-version
        + plan:
        + - get: git-repository
        + - config:
        +     image_resource:
        +       name: ""
        +       source:
        +         repository: getourneau/alpine-bash-git
        +       type: docker-image
        +     inputs:
        +     - name: git-repository
        +     outputs:
        +     - name: git-repository
        +     params:
        +       BUMP_TYPE: minor
        +     platform: linux
        +     run:
        +       args:
        +       - -exc
        +       - |2
        + 
        +         # この時点ではconcorseがgit cloneを実行してるため、git cloneの実行は不要。
        +         # gitのローカルリポジトリのディレクトリに移動
        +         cd git-repository
        + 
        +         # 現在のバージョンを取得
        +         current_version=$(cat ./version/number)
        + 
        +         # バージョンをバンプするタイプを指定
        +         BUMP_TYPE=${BUMP_TYPE:-"major"}  # デフォルトはメジャーバージョンをバンプ
        + 
        +         # current_version が空文字列の場合
        +         if [ "$current_version" = "" ]; then
        +             # 適切な初期バージョンを設定
        +             if [ "$BUMP_TYPE" = "major" ]; then
        +               new_version="1.0.0"
        +             elif [ "$BUMP_TYPE" = "minor" ]; then
        +               new_version="0.1.0"
        +             elif [ "$BUMP_TYPE" = "patch" ]; then
        +               new_version="0.0.1"
        +             else
        +               echo "Invalid bump_type specified."
        +               exit 1
        +             fi
        +         else
        +           # バージョンをバンプする
        +           IFS_SAVE=$IFS
        +           IFS='.'
        +           set $current_version
        +           IFS=$IFS_SAVE
        +           major="$1"
        +           minor="$2"
        +           patch="$3"
        +           if [ "$BUMP_TYPE" = "major" ]; then
        +             major=$((major + 1))
        +             minor=0
        +             patch=0
        +           elif [ "$BUMP_TYPE" = "minor" ]; then
        +             minor=$((minor + 1))
        +             patch=0
        +           elif [ "$BUMP_TYPE" = "patch" ]; then
        +             patch=$((patch + 1))
        +           fi
        +           new_version="$major.$minor.$patch"
        +         fi
        + 
        +         # 新しいバージョンをファイルに書き込む
        +         echo "$new_version" > ./version/number
        +         cat ./version/number
        + 
        +         # コミットしタグを付加する
        +         git config --global user.email "concourse@local" # メールアドレスは必須（ユーザ名は任意）
        +         git add ./version/number
        +         git commit -m "Bump version to v$new_version"
        +         git tag v$new_version
        +       path: /bin/sh
        +   task: bump-timestamp-file
        + - params:
        +     repository: git-repository
        +   put: git-repository
        
        pipeline name: bump-soft-minor-version

        pipeline created!
        you can view your pipeline here: http://localhost:8080/teams/main/pipelines/bump-soft-minor-version

        the pipeline is currently paused. to unpause, either:
        - run the unpause-pipeline command:
            fly -t tutorial unpause-pipeline -p bump-soft-minor-version
        - click play next to the pipeline in the web ui
        ```

- リソースのチェク

    ```sh
    $ fly -t tutorial check-resource -r bump-soft-minor-version/git-repository
    ```

    - 結果

        ```sh
        checking bump-soft-minor-version/git-repository in build 14313
        initializing check: git-repository
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        HEAD is now at e9ec4c4 Bumped date
        succeeded
        ```

- パイプラインの実行
    ```sh
    $ fly -t tutorial unpause-pipeline -p bump-soft-minor-version
    ```

    ```sh
    $ fly -t tutorial trigger-job -j bump-soft-minor-version/bump-version -w
    ```

    - 結果

        ```sh
        # fly -t tutorial trigger-job -j bump-soft-minor-version/bump-version -w
        started bump-soft-minor-version/bump-version #2

        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        d1e237f Update number
        initializing
        initializing check: image
        selected worker: 97ad8c1afbe6
        selected worker: 97ad8c1afbe6
        INFO: found existing resource cache

        selected worker: 97ad8c1afbe6
        running /bin/sh -exc 
        # この時点ではconcorseがgit cloneを実行してるため、git cloneの実行は不要。
        # gitのローカルリポジトリのディレクトリに移動
        cd git-repository

        # 現在のバージョンを取得
        current_version=$(cat ./version/number)

        # バージョンをバンプするタイプを指定
        BUMP_TYPE=${BUMP_TYPE:-"major"}  # デフォルトはメジャーバージョンをバンプ

        # current_version が空文字列の場合
        if [ "$current_version" = "" ]; then
            # 適切な初期バージョンを設定
            if [ "$BUMP_TYPE" = "major" ]; then
            new_version="1.0.0"
            elif [ "$BUMP_TYPE" = "minor" ]; then
            new_version="0.1.0"
            elif [ "$BUMP_TYPE" = "patch" ]; then
            new_version="0.0.1"
            else
            echo "Invalid bump_type specified."
            exit 1
            fi
        else
        # バージョンをバンプする
        IFS_SAVE=$IFS
        IFS='.'
        set $current_version
        IFS=$IFS_SAVE
        major="$1"
        minor="$2"
        patch="$3"
        if [ "$BUMP_TYPE" = "major" ]; then
            major=$((major + 1))
            minor=0
            patch=0
        elif [ "$BUMP_TYPE" = "minor" ]; then
            minor=$((minor + 1))
            patch=0
        elif [ "$BUMP_TYPE" = "patch" ]; then
            patch=$((patch + 1))
        fi
        new_version="$major.$minor.$patch"
        fi

        # 新しいバージョンをファイルに書き込む
        echo "$new_version" > ./version/number
        cat ./version/number

        # コミットしタグを付加する
        git config --global user.email "concourse@local" # メールアドレスは必須（ユーザ名は任意）
        git add ./version/number
        git commit -m "Bump version to v$new_version"
        git tag v$new_version

        + cd git-repository
        + cat ./version/number
        + current_version=
        + BUMP_TYPE=minor
        + '['  '='  ]
        + '[' minor '=' major ]
        + '[' minor '=' minor ]
        + new_version=0.1.0
        + echo 0.1.0
        + cat ./version/number
        0.1.0
        + git config --global user.email concourse@local
        + git add ./version/number
        + git commit -m 'Bump version to v0.1.0'
        [detached HEAD 3817200] Bump version to v0.1.0
        1 file changed, 1 insertion(+), 1 deletion(-)
        + git tag v0.1.0
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        To github.com:moriyamaES/semver-test.git
        d1e237f..3817200  HEAD -> main
        * [new tag]         v0.1.0 -> v0.1.0
        selected worker: 97ad8c1afbe6
        Identity added: /tmp/git-resource-private-key (/tmp/git-resource-private-key)
        Cloning into '/tmp/build/get'...
        3817200 Bump version to v0.1.0
        succeeded
        ```

- 成功！！