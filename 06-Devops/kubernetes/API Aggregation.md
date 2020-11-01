#### 在master中启动API的聚合功能

◎　--requestheader-client-ca-file=/etc/kubernetes/ssl_keys/ca.crt：客户端CA证书。
◎　--requestheader-allowed-names=：允许访问的客户端common names列表，通过header中--requestheader-username-headers参数指定的字段获取。客户端common names的名称需要在client-ca-file中进行设置，将其设置为空值时，表示任意客户端都可访问。
◎　--requestheader-extra-headers-prefix=X-Remote-Extra-:：请求头中需要检查的前缀名。
◎　--requestheader-group-headers=X-Remote-Group：请求头中需要检查的组名。
◎　--requestheader-username-headers=X-Remote-User：请求头中需要检查的用户名。
◎　--proxy-client-cert-file=/etc/kubernetes/ssl_keys/kubelet_client.crt：在请求期间验证Aggregator的客户端CA证书。
◎　--proxy-client-key-file=/etc/kubernetes/ssl_keys/kubelet_client.key：在请求期间验证Aggregator的客户端私钥。

如果kube-apiserver所在的主机上没有运行kube-proxy，即无法通过服务的ClusterIP进行访问，那么还需要设置以下启动参数：

--enable-aggregator-routing=true

#### 注册自定义APIService资源

kubectl create -f   xxx.yaml

#### 实现和部署自定义的API Server

1. 下载[Releases · kubernetes-sigs/apiserver-builder-alpha · GitHub](https://github.com/kubernetes-sigs/apiserver-builder-alpha/releases)放到 $GOPATH/bin

2. ```
   $ cd GOPATH/src/github.com/my-org/my-project
   $ apiserver-boot init repo --domain <your-domain>
   $ apiserver-boot init glide
   ```

3.     编译不通过！！！！ 谁能救我。
