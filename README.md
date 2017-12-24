## Day 6 - 與 k8s 溝通: APIs

### 本日共賞

* APIs
* API 存取規則

### 希望你知道
* [安裝 k8s](https://ithelp.ithome.com.tw/articles/10192748)
* [運行 minikube](https://ithelp.ithome.com.tw/articles/10193237)

[Day 3 - k8s 安裝須知](https://ithelp.ithome.com.tw/articles/10192448) 曾經提到有三種方式可以與 k8s 溝通

1. APIs
2. 儀表板 (Dashboard UI)
3. kubectl

今天讓我們花點時間，了解一下如何運用 APIs 與 k8s 溝通。

#### APIs

在 k8s 中，使用者必須將命令傳達給 master node 中運作的 `api-server`，當 `api-server` 接收到使用者的要求後，會先驗證使用者是否有足夠的權限進行操作，而後才會執行使用者的命令。使用者可以透過 k8s 提供的 RESTful APIs 對物件進行新增 (POST)、查詢 (GET) 或刪除 (DELETE) 等等動作。想透過這種方式與 k8s 溝通必須

* 取得認證 token
* 知道 master node ip，因為 api-server 運行在 master node
* 知道 api 存取規則

<br/>
<strong>*取得認證 token*</strong>

經過 [Day 5 - 運行 minikube](https://ithelp.ithome.com.tw/articles/10193237) 後，minikube 已經順利運行，因此可以透過下列指令取得 token

```bash
$ TOKEN=$(kubectl describe secret $(kubectl get secrets | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d ' ')
``` 

>細心的讀者應該有發現，這裡有使用 kubectl 這個工具。因為啟動 minikube 時已經自動幫我們建立認證，因此我們可以透過 kubectl 取得與 api-server 溝通的 token。如果想自行定義認證方式可參考 [Authenticating](https://kubernetes.io/docs/admin/authentication/)

上述指令會將 token 資訊儲存在 `$TOKEN` 中，可以透過下列指令查看

```bash
$ echo $TOKEN
eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6ImRlZmF1bHQtdG9rZW4tbGQ5anQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGVmYXVsdCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjA0ZDc5MzAxLWQwMWMtMTFlNy1hZThkLTA4MDAyNzlhODM3OSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OmRlZmF1bHQifQ.qF6kSTdzdBuus4y601u29bbLpWlbw-EooDqRwVFc2k9mZLxN05GagyBYIjORp4QRp9PbK-z_4Si3Nwc4Zii94Wv2XGfmZfm4v8eaZJnD_aX6r6r1dE5f8c8GMMzRQ5F17m0kxA6u62AZ8tCQX62uZLDWYwPT6R0f0IfEh1XZsBGvRsRAlGUcftLL-DIHFzYTazEZm35QnWVP36FaKvyP_Awhlq9OqZRrKTUCux5Mh5Wv_X5vMsf-WwS1E1OFTekg-w5AFKcsLeY5qYq-GkkDg_OlUiqI3H8WZRC2DJzgezQhtiVV5vol_oLTl-__ms-0UaJOB5Fked3Gi1KmZA8FXw
```

> token 內容並不會一樣，所以請不要直接複製使用喔！

<br/>

**master node ip**

```bash
$ APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
```

我們可以透過上面指令將 config 中描述 node ip 的內容擷取下來，master node ip 會被存在變數 `$APISERVER` 中。同樣地，可以透過指令查看

```bash
$ echo $APISERVER
https://192.168.99.100:8443
```

> 如果操作多個叢集，上述指令可能會將多個 ip 儲存在 `$APISERVER` 中，請記得修正。另外，也可使用 `kubectl proxy` 代理，將 ip 轉到本機端。關於 `kubectl proxy` 我們會在之後的文章再詳細說明。

<br/>

#### API 存取規則

基本上，k8s 中所有的物件都可以透過 APIs 操作。因此 k8s 提供了很 [完整的文件說明](https://kubernetes.io/docs/concepts/overview/kubernetes-api/)。你可以找到 [對應版本的文件](https://kubernetes.io/docs/reference/)，例如 [v1.8 版可用參數與如何存取](https://kubernetes.io/docs/api-reference/v1.8/)

> 目前 k8s 相當活躍，平均三個月更新版本一次。這裡建議把基本觀念以及需要用到的物件弄清楚，剩餘部分再慢慢閱讀。而本次鐵人賽分享的主旨便是基本觀念以及常用到的物件。

*如何閱讀文件*

![](https://ithelp.ithome.com.tw/upload/images/20171224/20107062vqXEHB2NKU.png)

上圖是官方 v1.8 文件中讀取 Pod 的說明。可以分成三個部分來看

* <span style="color:#ffffff;background:red; padding: 2px">紅色</span>：API 概觀，標示可操作物件。這邊看到的 `Deployment`、`Job` 或 `Pod` 等等指的是可以部署到 k8s 的物件名稱。
* <span style="color:#ffffff;background:blue; padding: 2px">藍色</span>：操作說明，詳述 Http url 以及可帶參數。
* <span style="color:#ffffff;background:green; padding: 2px">綠色</span>：提供 kubectl 與 curl 操作範例。

因此，我們可以從紅色區域中挑選想要操作的物件，根據藍色區域中的說明，對想要變更的物件進行操作

了解上述三個步驟後，我們就可以嘗試利用 curl 與 k8s 溝通

*範例一 查看 default 命名空間內正在運行的 Pod*

根據 [說明文件](https://v1-8.docs.kubernetes.io/docs/api-reference/v1.8/#-strong-read-operations-strong--60) 格式如下

```
GET /api/v1/namespaces/{namespace}/pods/{name}
```

因此，如果要查詢 `default` 命名空間下的 Pod

> 因為還沒有部署任何 Pod 到 k8s，所以會查不到東西，可以先利用範例二新增一個 Pod 到 k8s 後再來試試看查詢。

```bash
$ curl $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure
{
  "kind": "PodList",
  "apiVersion": "v1",
  "metadata": {
    "selfLink": "/api/v1/namespaces/default/pods",
    "resourceVersion": "51161"
  },
  "items": [
    {
      "metadata": {
        "name": "nginx-75f4785b7-dxp2h", <== 試試查詢這個 Pod
...
```

* `$APISERVER/api/v1/namespaces/default/pods`：查詢 Pod 物件的 API Url
* `--header "Authorization: Bearer $TOKEN"`：帶入認證 Token

結果會將 `default` 中所有的 Pod 以 JSON 的方式回傳。如果對應到原本的格式來看，`{namespace}` 就是 `default`，而 `{name}` 未指定則表示查詢 `default` 底下所有的 Pod。

因此，如果要查詢單一個 Pod，你會需要知道 Pod 的名稱。例如上面的 `nginx-75f4785b7-dxp2h` 

```bash
$ curl $APISERVER/api/v1/namespaces/default/pods/nginx-75f4785b7-dxp2h --header "Authorization: Bearer $TOKEN" --insecure
```

*範例二 在 default 中新增一個 Pod*

根據 [說明文件](https://v1-8.docs.kubernetes.io/docs/api-reference/v1.8/#-strong-write-operations-strong--54)，如果要透過 API 部署一個 Pod 物件到 k8s 可參考下面格式

```
POST /api/v1/namespace/{namespace}/pods
```

而 Post Body 的部分就是填入 Pod 的相關設定，例如

```vim
{"apiVersion": "v1",
 "kind": "Pod",   <=== 這裡指定部署 Pod 物件，其他設定內容之後我們會再詳細說明
 "metadata": {"name":"nginx"},
 "spec": {
   "containers": [
     {"image": "nginx",
      "name": "frontend",
      "ports": [{"containerPort": 80,"protocol": "TCP"}]
     }
   ]
 }
}
```

底下我們透過 curl 呼叫 API 來部署一個 Pod，基本上就是把上面的內容帶給 k8s

```bash
$ curl -X POST $APISERVER/api/v1/namespaces/default/pods --header "Authorization: Bearer $TOKEN" --insecure -H "Content-Type:application/json" -d '{"apiVersion": "v1","kind": "Pod","metadata": {"name":"nginx"},"spec": {"containers": [{"image": "nginx","name": "frontend","ports": [{"containerPort": 80,"protocol": "TCP"}]}]}}'
{
  "kind": "Pod",
  "apiVersion": "v1",
  "metadata": {
    "name": "nginx",
    "namespace": "default",
    "selfLink": "/api/v1/namespaces/default/pods/nginx",
    "uid": "6aeb1863-e85e-11e7-b5bb-080027641f6a",
    "resourceVersion": "46479",
    "creationTimestamp": "2017-12-24T03:56:27Z"
  },
  ...
```

這裡指令比較長我們把它分開來看：

* `curl -X POST`：指定使用 POST 方式傳輸
* `$APISERVER/api/v1/namespaces/default/pods`：新增 Pod API 位置
* `--header "Authorization: Bearer $TOKEN"`：帶入認證 Token
* `-H "Content-Type:application/json"`：指定傳輸內容格式為 JSON
* `-d {pod settings}`：POST body, 即描述部署到 k8s 的 Pod 的內容

> 可以用範例一的方法查看新增加的內容

寫到這裡，大家應該都可以感覺到雖然可以用 API 的方式跟 k8s 溝通，不過似乎不是很方便容易操作，別擔心！我們明天會介紹其他存取方式。

> API 的方式一般來說會用在程式內，如果是人工操作大多不會使用這種方式。不過，如果你喜歡也是可以喔！


本文同步發表於 [https://jlptf.github.io/ironman2018-day6/](https://jlptf.github.io/ironman2018-day6/)