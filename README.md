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


