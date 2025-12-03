# [Tips] Speed up JupyterHub startup in air-gapped environment

Created: 2025-12-03
Tags: #jupyterhub #offline #performance #docker #troubleshooting


## Configuration / Code

### 1. Disable AWS Extensions (AWS拡張の無効化)
SageMakerコンテナに含まれるAWS連携機能（Glue/Athena用エディタ等）は、起動時にAWSエンドポイントへ接続を試みるため、閉域網ではこれがタイムアウトするまで画面が表示されません。これを明示的に無効化します。

```sh

# Disable SageMaker SQL Editor extension to prevent timeout
jupyter server extension disable amazon_sagemaker_sql_editor || true

```



### 2 Docker Pull Policy (イメージ確認の無効化)

JupyterHubがユーザーコンテナを起動する際、デフォルトではRegistryに最新イメージを確認しに行きます。これを「Never」に設定し、ローカルにあるイメージだけを使用させます。

**File: `jupyterhub_config.py`**

```python
# Prevent checking Docker Registry for updates
c.DockerSpawner.pull_policy = 'Never'
```

### 3 Force Offline for Package Managers (パッケージ管理のオフライン化)

pipやcondaがバックグラウンドでインデックス更新や通知確認を行うのを防ぎます。

**Environment Variables**

```python
# Set in jupyterhub_config.py or Dockerfile
os.environ['PIP_NO_INDEX'] = 'true'
os.environ['CONDA_OFFLINE'] = 'true'
os.environ['NO_UPDATE_NOTIFIER'] = 'true'
```

