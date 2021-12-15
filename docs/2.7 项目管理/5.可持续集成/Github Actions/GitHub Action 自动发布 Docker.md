[GitHub Action 自动发布 Docker](https://segmentfault.com/a/1190000039715679)



## 自动测试

当然以上流程完全可以利用 `Actions` 自动化搞定。

首选我们需要在项目根路径创建一个 *`.github/workflows/\*.yml`* 的配置文件，新增如下内容：

```
name: go-docker
on: push
jobs:
  test:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' || startsWith(github.ref, 'refs/tags')
    steps:
      - uses: actions/checkout@v2
      - name: Run Unit Tests
        run: go test
```

简单解释下：

- `name` 不必多说，是为当前工作流创建一个名词。
- `on` 指在什么事件下触发，这里指代码发生 `push` 时触发，更多事件定义可以参考官方文档：

[Events that trigger workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)

- `jobs` 则是定义任务，这里只有一个名为 `test` 的任务。

该任务是运行在 `ubuntu-latest` 的环境下，只有在 `main` 分支有推送或是有 `tag` 推送时运行。

运行时会使用 `actions/checkout@v2` 这个由他人封装好的 `Action`，当然这里使用的是由官方提供的拉取代码 `Action`。

- 基于这个逻辑，我们可以灵活的分享和使用他人的 `Action` 来简化流程，这点也是 `GitHub Action`扩展性非常强的地方。

最后的 `run` 则是运行自己命令，这里自然就是触发单元测试了。

- 如果是 Java 便可改为  `mvn test`.

之后一旦我们在 `main` 分支上推送代码，或者有其他分支的代码合并过来时都会自动运行单元测试，非常方便。

![img](https://segmentfault.com/img/remote/1460000039715689)

![img](https://segmentfault.com/img/remote/1460000039715685)

与我们本地运行效果一致。

## 自动发布

接下来考虑自动打包 `Docker` 镜像，同时上传到 `Docker Hub`；为此首先创建 `Dockerfile` ：

```
FROM golang:1.15 AS builder
ARG VERSION=0.0.10
WORKDIR /go/src/app
COPY main.go .
RUN go build -o main -ldflags="-X 'main.version=${VERSION}'" main.go

FROM debian:stable-slim
COPY --from=builder /go/src/app/main /go/bin/main
ENV PATH="/go/bin:${PATH}"
CMD ["main"]
```

这里利用 `ldflags` 可在编译期间将一些参数传递进打包程序中，比如打包时间、go 版本、git 版本等。

这里只是将 `VERSION` 传入了  `main.version` 变量中，这样在运行时就便能取到了。

```
docker build -t go-docker:last .
docker run --rm go-docker:0.0.10
0.0.10
```

接着继续编写 `docker.yml` 新增自动打包 `Docker` 以及推送到 `docker hub` 中。

```
deploy:
    runs-on: ubuntu-latest
    needs: test
    if: startsWith(github.ref, 'refs/tags')
    steps:
      - name: Extract Version
        id: version_step
        run: |
          echo "##[set-output name=version;]VERSION=${GITHUB_REF#$"refs/tags/v"}"
          echo "##[set-output name=version_tag;]$GITHUB_REPOSITORY:${GITHUB_REF#$"refs/tags/v"}"
          echo "##[set-output name=latest_tag;]$GITHUB_REPOSITORY:latest"

      - name: Set up QEMU
        uses: docker/setup-qemu-action@v1

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v1

      - name: Login to DockerHub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKER_USER_NAME }}
          password: ${{ secrets.DOCKER_ACCESS_TOKEN }}

      - name: PrepareReg Names
        id: read-docker-image-identifiers
        run: |
          echo VERSION_TAG=$(echo ${{ steps.version_step.outputs.version_tag }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV
          echo LASTEST_TAG=$(echo ${{ steps.version_step.outputs.latest_tag  }} | tr '[:upper:]' '[:lower:]') >> $GITHUB_ENV

      - name: Build and push Docker images
        id: docker_build
        uses: docker/build-push-action@v2.3.0
        with:
          push: true
          tags: |
            ${{env.VERSION_TAG}}
            ${{env.LASTEST_TAG}}
          build-args: |
            ${{steps.version_step.outputs.version}}
```

新增了一个 `deploy` 的 job。

```
    needs: test
    if: startsWith(github.ref, 'refs/tags')
```

运行的条件是上一步的单测流程跑通，同时有新的 `tag` 生成时才会触发后续的 `steps`。

```
name: Login to DockerHub
```

在这一步中我们需要登录到 `DockerHub`，所以首先需要在 GitHub 项目中配置 hub 的 `user_name` 以及 `access_token`.

![img](https://segmentfault.com/img/remote/1460000039715681)

![img](https://segmentfault.com/img/remote/1460000039715684)

配置好后便能在 action 中使用该变量了。

![img](https://segmentfault.com/img/remote/1460000039715686)

这里使用的是由 docker 官方提供的登录 action(`docker/login-action`)。

有一点要非常注意，我们需要将镜像名称改为小写，不然会上传失败，比如我的名称中 `J` 字母是大写的，直接上传时就会报错。

![img](https://segmentfault.com/img/remote/1460000039715688)

所以在上传之前先要执行该步骤转换为小写。

![img](https://segmentfault.com/img/remote/1460000039715683)

最后再用这两个变量上传到 Docker Hub。

![img](https://segmentfault.com/img/remote/1460000039715687)

![img](https://segmentfault.com/img/remote/1460000039715687)

今后只要我们打上 `tag` 时，`Action` 就会自动执行单测、构建、上传的流程。