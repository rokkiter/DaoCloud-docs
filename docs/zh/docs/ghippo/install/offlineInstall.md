# GHippo 离线安装包使用文档

## 解压 tar 压缩包

```sh
tar zxvf ghippo.bundle.tar
```


## 加载镜像文件

解压成功后会得到 ghippo.bundle 文件，其中包含 hints.yaml、images.tar、original-chart 3个子文件。

### 通过chart-syncer同步镜像到指定harbor

- 首先请确认本地是否安装chart-syncer，如果已安装，则跳过当前安装步骤

```shell
tmp_dir=$(mktemp -d)

git clone https://github.com/DaoCloud/charts-syncer.git ${tmp_dir}

cp ${tmp_dir}/charts-syncer /usr/local/bin/charts-syncer

chmod +x /usr/local/bin/charts-syncer

rm -rf ${tmp_dir}
```

- 创建load-image.yaml，具体yaml参考：
https://github.com/DaoCloud/charts-syncer/blob/master/examples/test-load-dao-2048.yaml
并根据私有harbor修改相关配置

- 执行同步镜像命令
```shell
charts-syncer sync --config load-image.yaml
```

### 本地加载镜像到docker

```sh
cd ghippo.bundle

docker load -i images.tar
```

## 开始升级

- 备份 --set 参数

在升级 ghippo 版本之前，我们建议您执行如下命令，备份上一个版本的 --set 参数。

```shell
helm get values ghippo -n ghippo-system -o yaml > bak.yaml
```

- 执行 helm upgrade

```shell
cd original-chart

helm upgrade ghippo . \
-n ghippo-system \
-f ./bak.yaml
```
