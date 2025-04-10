# flask アプリ

## コマンド

```sh
# ローカルサーバ起動
make run
```

## Azure App Service へのデプロイ

### コマンド

```sh
# Azure にログイン
az login
※ az コマンドがない場合、 brew install azure-cli でインストール

##### Azure App Service にデプロイ #####

# B1: ¥8,266.703/month
az webapp up --runtime PYTHON:3.12 --sku B1 --logs
※ sku: Stock Keeping Unit

# F1: ¥0
az webapp up --runtime PYTHON:3.12 --sku F1 --logs

# F1 + 各種指定(アプリ名, リソースグループ名, リージョン)
※ AppServicePlan は事前に作成していない想定
az webapp up --name {アプリ名} --resource-group {リソースグループ名} --runtime PYTHON:3.12 --sku F1 --location japaneast --logs
## 例
az webapp up --name flaskdemo20250410 --resource-group rg-flaskdemo-dev-jpn-01 --runtime PYTHON:3.12 --sku F1 --location japaneast --logs


※ アプリ名はグローバルでユニークである必要がある
※ 一般的に大文字は使用せず、小文字、数字、ハイフン、アンダースコアを組み合わせる
※ App Service Plan は自動で作成され、Azure 側で命名される
  => 名前を指定したいなら事前に作成しておく必要がある
  [作成順序]
  リソースグループ => AppServicePlan => webapp => ソースコードが入った zip
※ App Service Plan が定義するもの
  - オペレーティング システム (Windows、Linux)
  - リージョン (米国西部、米国東部など)
  - VM インスタンスの数
  - VM インスタンスのサイズ (小、中、大)
  - 価格レベル (Free、Shared、Basic、Standard、Premium、PremiumV2、PremiumV3、Isolated、IsolatedV2)
    ※ デフォルトでは無料の F1 プランが設定される

# リソースのクリーンアップ
az group delete --name {リソースグループ名} --no-wait
## 例
az group delete --name rg-flaskdemo-dev-jpn-01 --no-wait

##### App Service Plan の作成 #####
# リソースグループの作成
※ App Service Plan の作成にはリソースグループが必要
az group create --name {resource-group-name} --location {region}
## 例
az group create --name rg-flaskdemo-dev-jpn-01 --location japaneast

# App Service Plan の作成
az appservice plan create --name {app-service-plan-name} --resource-group {resource-group-name} --sku {sku} --is-linux
## 例
az appservice plan create --name asp-flaskdemo-dev-jpn-01 --resource-group rg-flaskdemo-dev-jpn-01 --sku F1 --is-linux


# 作成した App Service Plan の確認
az appservice plan list --resource-group {resource-group-name} --output table
## 例
az appservice plan list --resource-group rg-flaskdemo-dev-jpn-01 --output table


# webapp の作成
az webapp up --name {アプリ名} --resource-group {リソースグループ名} --plan {AppServicePlan名} --runtime PYTHON:3.12 --sku F1 --location japaneast --logs
## 例
az webapp up --name flaskdemo20250410 --resource-group rg-flaskdemo-dev-jpn-01 --plan  --runtime PYTHON:3.12 --sku F1 --location japaneast --logs



##### その他 #####
# 使用可能な全てのランタイムを一覧表示
az webapp list-runtimes --os linux --output table

```

### 参考資料

- [[公式] App Service の概要](https://learn.microsoft.com/ja-jp/azure/app-service/overview)
- [[公式] クイック スタート: Python (Django、Flask、または FastAPI) Web アプリを Azure App Service にデプロイする](https://learn.microsoft.com/ja-jp/azure/app-service/quickstart-python?tabs=flask%2Cwindows%2Cazure-cli%2Cazure-cli-deploy%2Cdeploy-instructions-azportal%2Cterminal-bash%2Cdeploy-instructions-zip-azcli)
- [[公式]名前付け規則を定義する](名前付け規則を定義する)
  ```sh
  # Azure 推奨命名規則
  {リソースの種類}-{プロジェクト名}-{env}-{region}-{番号/ID/カウンター}
  # 例 (リソースグループ)
  rg-gettingstarteddemo-linuxvm-eus2-dev-01
  ※ eus2: East US 2 (Virginia, United States)
  ※ gettingstarteddemo-linuxvm がプロジェクト名
  ```
- [[公式] Azure regions list](https://learn.microsoft.com/ja-jp/azure/reliability/regions-list)
  ```sh
  # 日本
  Japan East (Tokyo, Saitama): jpn
  Japan West (Osaka): jpnw
  ```
- [[公式] Azure リソースの省略形の例](https://learn.microsoft.com/ja-jp/azure/cloud-adoption-framework/ready/azure-best-practices/resource-abbreviations)
  Virtual machines: vm<br>
  AppServicePlan: asp<br>
  etc...
