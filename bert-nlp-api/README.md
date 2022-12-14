# AzureMchineLearningを使った機械学習(BERT)API

## 環境
- Windows10
- VSCode

## 参考資料
https://docs.microsoft.com/ja-jp/azure/machine-learning/how-to-deploy-managed-online-endpoints

## 前提
- Azureのサブスクリプションとリソースグループと、AzureMLのワークスペースが必要
- AzureCLI（コマンドベース）でエンドポイント作成やデプロイを行うため、AzureCLIとml拡張機能のインストールが必要
- AzureCLIの既定値の設定推奨（何度もパラメータ渡さなくてすむように）
```
az account set --subscription <subscription ID>
az configure --defaults workspace=<Azure Machine Learning workspace name> group=<resource group>
```

## 手順

### AzureCLIとml拡張機能のインストール
https://docs.microsoft.com/ja-jp/azure/machine-learning/how-to-configure-cli?tabs=public

### Dockerのインストール（ローカルの動作確認に必要）
https://docs.docker.com/engine/install/

### ローカルのDockerにデプロイ
#### 既定の設定を確認
az configure -l -o table

#### エンドポイント名の環境変数を設定
export ENDPOINT_NAME=bertendpoint

#### ローカルにエンドポイントを作成
az ml online-endpoint create --local -n $ENDPOINT_NAME -f endpoints/online/managed/endpoint.yml

#### ローカルのエンドポイントにデプロイ
az ml online-deployment create --local -n blue --endpoint $ENDPOINT_NAME -f endpoints/online/managed/nlp-blue-deployment.yml
- デプロイが完了するとinit()が自動で実行される
- 初回は（conda.yamlを更新した場合も）時間がかかる（20~30分ほど）


- デプロイを更新する場合はaz ml online-deployment update
az ml online-deployment update --local -n blue --endpoint $ENDPOINT_NAME -f endpoints/online/managed/nlp-blue-deployment.yml
<br>
エラーになる
```
./run: line 63: exec: gunicorn: not found
2022-08-09T12:48:53,391099900+00:00 - gunicorn/finish 127 0
2022-08-09T12:48:53,394034800+00:00 - Exit code 127 is not normal. Killing image.
```

### ローカル デプロイが成功したかどうかを確認する
az ml online-endpoint show -n $ENDPOINT_NAME --local

### ローカル エンドポイントを呼び出し、推論結果を得る
az ml online-endpoint invoke --local --name $ENDPOINT_NAME --request-file endpoints/online/sample-text-request.json
az ml online-endpoint invoke --local --name $ENDPOINT_NAME --request-file endpoints/online/sample-text-request-multi.json


## Azureにデプロイ
無料のサブスクリプションだと、quota足りなくてデプロイできない・・