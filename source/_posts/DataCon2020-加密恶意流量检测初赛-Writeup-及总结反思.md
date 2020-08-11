
---
title: DataCon2020 加密恶意流量检测初赛 Writeup 及总结反思
tags: [Network, Security]
categories: [Algorithms]
date: 2020-08-09 23:20:52
---
## 初赛题目

主办方提供了 black/white/test 三个 pcap 文件夹，其中 black 和 white 分别是检测出有/无恶意软件感染的客户端 IP 组，要求选手对 test 数据集进行判定。

![09bda35ff290f2e34d88d50e71d98f1a.png](0.png)

## 解题思路

该题的解题过程应分为以下三个大步骤：
1. **特征选择（Feature Selection）**，选取对于区分正常/恶意流量有明显作用的 Features。
2. **特征提取（Feature Extraction）**，从 pcap 文件中提取上述 Features，并转换为模型训练所需要的格式。我们选择的特征提取工具为 **Zeek**。
3. **模型训练（Model Training）**，选择合适的机器学习模型对三类 pcap 文件进行训练和预测。我们选择的 Python 机器学习库为 **scikit-learn**。

## 解题步骤

### 1. 合并pcap文件

主办方提供的 pcap 文件，其中 white/black 各有 1500 个 pcap，test 2000 个 pcap

```
➜  tree eta_1 
eta_1
├── black
│   ├── 192.168.10.91.pcap
│   ├── 192.168.44.25.pcap
|		├── ...
│   └── 192.168.80.115.pcap
├── test
│   ├── 192.168.150.71.pcap
│   ├── 192.168.150.99.pcap
|		├── ...
│   └── 192.168.210.239.pcap
└── white
    ├── 192.168.119.23.pcap
    ├── 192.168.122.37.pcap
		├── ...
    └── 192.168.96.180.pcap
```

使用 `mergecap` 命令将 pcap 文件合并为三个大的 pcap 文件

```
➜  mergecap -w black.pcap ./eta_1/black/*.pcap
➜  tree -s -h .     
.
├── [418M]  black.pcap
├── [1.0G]  test.pcap
└── [1.7G]  white.pcap
```

### 2. 特征选择

参考了多篇加密恶意流量检测的研究论文，我们初步确定了以下四类需要提取的 Features，包括**TLS客户端指纹信息**、**数据包元数据**、**HTTP头部信息**和**DNS响应信息**。

#### TLS client fingerprinting：

在进行TLS握手时，会进行如下几个步骤：
1. **Client Hello**，客户端提供支持的加密套件数组（cipher suites）；
2. **Server Hello**，由服务器端选择一个加密套件，传回服务器端公钥，并进行认证和签名授权（**Certificate** + Signature）；
3. 客户端传回客户端公钥（**Client Key Exchange**），客户端确立连接；
4. 服务器端确立连接，开始 HTTP 通信。

![5b01483a65bdc17c68873508796bb85e.png](1.png)

以上加粗的四种消息类型可以通过 TLS 握手协议的 **Handshake Type** 做区分：

![bff6a460bcaa7c442014c3eb89deb26f.png](2.png)

基于此，我们选取了以下特征：

- **客户端支持的加密套件数组**（Cipher suites），**服务器端选择的加密套件**。
- **支持的扩展**（TLS extensions），若分别用向量表示客户端提供的密码套件列表和 TLS 扩展列表，可以从服务器发送的确认包中的信息确定两组向量的值。
- **客户端公钥长度**（Client public key length），从密钥交换的数据包中，得到密钥的长度。
- **Client version**，the preferred TLS version for the client
- **是否非CA自签名**，统计数据表示，恶意流量约70%出现非CA认证服务器且自签名的情况，非恶意流量约占0.1%。此项判断的依据是：未出现 `CA: True` 字段（默认非 CA 机构）且 `signedCertificate` 中的 `issuer` 字段等于 `subject` 字段。

#### 数据包元数据：

- **数据包的大小**，数据包的长度受 UDP、TCP 或者 ICMP 协议中数据包的有效载荷大小影响，如果数据包不属于以上协议，则被设置为 IP 数据包的大小。
- **到达时间序列**
- **字节分布**

#### HTTP头部信息：

- **Content-Type**，正常流量 HTTP 头部信息汇总值多为 `image/*`，而恶意流量为 `text/*、text/html、charset=UTF-8` 或者 `text/html;charset=UTF-8`。
- **User-Agent**
- **Accept-Language**
- **Server**
- **HTTP响应码**

#### DNS响应信息：

- **域名的长度**：正常流量的域名长度分布为均值为6或7的高斯分布（正态分布）；而恶意流量的域名（FQDN全称域名）长度多为6（10）。
- **数字字符及非字母数字(non-alphanumeric character)的字符占比**：正常流量的DNS响应中全称域名的数字字符的占比和非字母数字字符的占比要大。
- **DNS解析出的IP数量**：大多数恶意流量和正常流量只返回一个IP地址；其它情况，大部分正常流量返回2-8个IP地址，恶意流量返回4或者11个IP地址。
- **TTL值**：正常流量的TTL值一般为60、300、20、30；而恶意流量多为300，大约22%的DNS响应汇总TTL为100，而这在正常流量中很罕见。
- **域名是否收录在Alexa网站**：恶意流量域名信息很少收录在Alexa top-1,000,000中，而正常流量域名多收录在其中。

### 3. 特征提取

特征提取我们采用的工具是 **Zeek**，它的前身是 Bro，一款网络安全监视（Network Security Monitoring）工具，同时它也支持直接处理 pcap 文件形成各类日志文件，包括 dns、http、smtp 等：

![cce00cf94b14017707eaa55f09ae67e1.png](6.png)

Zeek 网上有一些现成的脚本，我们采用的是 **Zeek FlowMeter**，它基于 OSI 七层协议的网络层和传输层，可以分析并生成一些 Packets 到达时间序列、Packet 字节大小和元数据等新特征。

在使用时，我们需要在 `local.zeek` 配置文件中加入 `@load flowmeter`，这样 Zeek 在执行时会加载 `flowmeter.zeek` 并生成对应的 `flowmeter.log`，下面列出了 FlowMeter 提取出的一些特征，包括上下行包总数、包负载均值方差等。其他详细的特征请见 [zeek-flowmeter GitHub官方文档](https://github.com/zeek-flowmeter/zeek-flowmeter)。

| Feature Name         | Description                                                                                                                              | exists in FlowMeter |
|----------------------|---------------------------------------------------------------------------------------------------------------------------------|---------------------|
| uid                  | The ID of the flow as given by Zeek                                                                                                      | No                  |
| flow\_duration       | The length of the flow in seconds \(maximal precision ms\)\. If only on packet was seen the duration is 0\.                              | Yes                 |
| fwd\_pkts\_tot       | The number of packets travelling in the forward direction\.                                                                              | Yes                 |
| bwd\_pkts\_tot       | The number of packets travelling in the backwards direction\.                                                                            | Yes                 |
| fwd\_pkts\_per\_sec  | The average number of forward packets transmitted per second during the flow\. If the duration is 0 then this feature is also set to 0\. | Yes                 |
| fwd\_pkts\_payload\.avg | The average payload size, in bytes, seen in the forward direction\.                   | Yes |
| fwd\_pkts\_payload\.std | The standard deviation of the payload size, in bytes, seen in the forward direction\. | Yes |
| ... | ... | ... |

除 `flowmeter.log` 之外我们还需要关注 `conn.log`、`ssl.log` 和 `X509.log`。这几个日志可用共同字段 `uid` 连接，`uid` 字段是 Zeek 根据一次连接的源/目的 IP、源/目的端口生成的唯一 ID。为了方便后续的处理，我们将这几个日志文件统一读入，使用 `uid` 字段连接后转成 csv 格式输出到文件。**最终提取的特征如下：**

![7d1475ca9c4d88e2c3e078a126fdfd52.jpeg](3.jpeg)

### 特征向量化

因为模型训练不支持 `str` 类型的特征，所以需要对 `version` 和 `server_cipher` 等字段进行特征向量化。

| version | cipher                                       |
|---------|----------------------------------------------|
| TLSv10  | TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA          |
| TLSv10  | TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA          |
| TLSv12  | TLS\_ECDHE\_RSA\_WITH\_AES\_128\_GCM\_SHA256 |
| TLSv10  | TLS\_RSA\_WITH\_RC4\_128\_MD5                |
| ... | ... |

`DictVectorizer` 是 scikit-learn 库中用于将 Python `dict` 对象表示的特征数组转换为 scikit-learn 估计器使用的 NumPy/SciPy 表示形式。

```python
def convert_by_dict(src_file, dest_file):
    from sklearn.feature_extraction import DictVectorizer
    vec = DictVectorizer()

    df = pd.read_csv(src_file)
    vs1 = list(map(lambda x: {'version': x}, df['version']))
    vs2 = vec.fit_transform(vs1).toarray()

    for index, name in enumerate(vec.get_feature_names()):
        df[name] = list(map(lambda x: int(x[index]), vs2))

    df.to_csv(dest_file, index=False)
```

向量化后的效果：

| version=SSLv3 | version=TLSv10 | version=TLSv12 | version=TLSv11 | cipher=TLS\_RSA\_WITH\_3DES\_EDE\_CBC\_SHA | ... |
|---------------|----------------|----------------|----------------|--| --|
| 0             | 1              | 0              | 0              | 0 |... |
| 0             | 1              | 0              | 0              |0 |... |
| 0             | 1              | 0              | 0              |0 |...|
| 0             | 0              | 1              | 0              |0 |... |
| ...            | ...              |...              |...             |...|... |

另外需要注意的是，**需要对特征列进行补全**，否则在进行模型训练时会出现特征个数不匹配的问题。例如：white.pcap 文件中 TLS versions 包含 `['TLSv1.0', 'TLSv1.2', 'SSLv3']` 三个版本，而 black 和 test 的 pcap 文件中除这三个版本外还包含 `'TLSv1.1'` 版本，所以需要在 white 中加入 `version=TLSv1.1` 全为 0 的列。

### 4. 模型选取及参数

需要注意的是：white 已明确没有被恶意软件感染，所以产生的流量可以全部标注为正常流量，而 black 明确的只是客户端感染了恶意软件，但产生的流量不一定全为恶意流量。所以实际上是**对不平衡样本数据进行训练和预测**。遵循这个思路，可以采用 **Anomaly Detector** + **Misuse Detector** 的联合分类器进行训练。

![fe7a1c020a03492fd8b465e8541a410e.png](4.png)

基于全正常流量的 white.pcap 文件进行 **one-class classification** 训练异常检测器 Anomaly Detector；再用该分类器对 black.pcap 文件进行推理预测恶意流量；结合 black 中检测的恶意流量和 white 正常流量训练二分类器；最终采用二分类器对 test 中的流量进行检测。

#### 数据处理

数据集是我们之前经过 Zeek 特征提取、特征向量化后产生的 `white.csv`、`black.csv` 和 `test.csv` 三个 csv 文件，使用 pandas 读入后定义一个列名数组 `data_f` 来获取相应特征：

```python
white_df = pd.read_csv("white.csv",index_col=0)
black_df = pd.read_csv("black.csv",index_col=0)
test_df = pd.read_csv("test.csv",index_col=0)

white_flow = white_df[data_f]
black_flow = black_df[data_f]
test_flow = test_df[data_f]

# 源 IP 用于生成结果 result.txt 文件时使用
test_ip = test_df['id.oirg_h']
```

将 white_flow 和 black_flow 两个数据集合并做归一化处理，得到total_data：

```python
merge_flow = white_flow.append(black_flow, ignore_index=True)

min_max_scaler = preprocessing.MinMaxScaler()
total_data = min_max_scaler.fit_transform(merge_flow.values)
```

然后我们从 total_data 中提取 white 的训练集和测试集，同时需要提取 black 的测试集：

```python
# white 训练集和测试集(标签1表示正常流量)
x_train,x_valid,y_train,y_valid=train_test_split(total_data[0:4834],np.ones(4834,np.int),random_state=30)

# black 测试集
black_data = total_data[4834:]
```

#### Anomaly Detector

在 black 数据集中同时存在恶意流量和正常流量，没有明确的标注，无法直接用于训练分类器。而 white 数据集中都是正常流量，可以先用 white 数据集来训练一个 Anomaly Detector 分类器。然后用这个分类器在 black 数据集中推理得到哪些是恶意流量。我们的模型选取的隔离森林 `IsolationForest`。

```python
clf = IsolationForest(max_samples=len(x_train), contamination=0.3)
clf.fit(x_train)

# 验证集正确率
y_pred_valid = clf.predict(x_valid)
print("valid_accuracy：" + str(np.mean(y_valid == y_pred_valid)))

# 由 black 测试集得到的恶意流量占比
y_black_test = clf.predict(black_data)
print("black_anomaly_ratio：" + str(1 - np.mean(y_black_test==np.ones(3980, np.int))))
```

#### 训练误用检测器 Misuse Detector

假设这些由异常检测器识别的可疑流量是恶意流量，我们就有了恶意流量的标注。接下来我们用这些恶意流量 labels，结合 white 数据集中的正常流量 labels，来训练一个 Misuse Detector。我们选取了 XGBoost 基于树的模型，目标选取为多分类问题（分类数为2）：

```python
# XGbooster Model     gbtree    multi:softmax   2

# label标签  1:white  0:black
white_flow['label'] = 1
black_flow['label'] = y_black_test
black_flow['label'] = black_flow['label'].map(lambda x: 1 if x > 0 else 0)

# 保留 black 中的恶意流量数据
balack_anomaly_flow = black_flow[black_flow['label'] == 0]

# 得到模型训练数据集
model_data = white_flow.append(balack_anomaly_flow, ignore_index=True) 
```

下一步对于数据做 Max/Min 归一化，然后分割出训练集、验证集、测试集：

```python
# Max/Min 归一化
X_flow = model_data[data_f]
label_flow = model_data['label'].values
model_total_flow = X_flow.append(test_flow, ignore_index=True)
data_normal = preprocessing.scale(model_total_flow.values)

# 分出训练集和测试集
model_train = data_normal[:model_data.shape[0]]
model_test = data_normal[model_data.shape[0]:]

# 训练集中分割出验证集(根据 label_flow 来层次化分割)
X_train, X_valid, Y_train, Y_valid = train_test_split(model_train, label_flow, random_state=50,stratify = label_flow)

xgboost_train = xgb.DMatrix(X_train, label=Y_train)
xgboost_valid = xgb.DMatrix(X_valid, label=Y_valid)
```

进行 XGBoost 模型训练：

```python
def evalerror(preds, dtrain):    
    labels = dtrain.get_label()
    return 'error', math.sqrt(metrics.mean_squared_error(preds,labels))

params = {
    'booster': 'gbtree',  # 树模型
    'objective': 'multi:softmax',  # 多分类
    'num_class': 2,  # 类别数
    
    'gamma': 0.1,  # 指定了节点分裂所需的最小损失函数下降值，越大越不易分裂
    'max_depth': 7,  # 树的深度
    'subsample': 0.75,  # 每棵树随机采样的占比（训练集中）
    'colsample_bytree': 0.75,  # 每棵树随机采样的列数的占比（每一列是一个特征）
    'min_child_weight': 0.06, # 决定最小叶子节点样本权重和
    
    'eta': 0.05,  # 学习率
    'seed': 100, # 随机种子
    'nthread': 6  # cpu 线程数
}
rounds = 200 # 迭代次数
watchlist = [(xgboost_train, 'train'), (xgboost_valid, 'valid')]
bst = xgb.train(params, xgboost_train, rounds, watchlist,feval=evalerror)
```

用模型去预测结果：

```python
# test 数据集上使用模型进行预测
xgboost_test = xgb.DMatrix(model_test)
y_predicted = bst.predict(xgboost_test)


# test_ip.txt 是 test 文件夹中的所有文件名
ip = pd.read_table('test_ip.txt', header=None)

# 文件名去掉对应的 .pcap 就是要预测的 IP
ip[0] = ip[0].map(lambda x: x.replace('.pcap', ''))
ip_list = list(ip[0])

# 先将所有客户端 IP 对应的类型标记为 white
ip[1] = ['white' for i in range(ip.shape[0])]

# 遍历预测结果
for i in range(len(y_predicted)):
    # 如果第 i 条流量为 black
    if y_predicted[i] == 0:
        # 读取第 i 条流量的源 IP
        tmp_ip = list(test_ip[i:i+1])[0]
        
        # 遍历所有要预测的客户端 IP
        for j in range(len(ip_list)):
            # 如果是第 i 条流量（恶意）的源 IP
            if tmp_ip == ip_list[j]:
            # 将该客户端 IP 标记为 black
                ip.loc[j,1] = 'black'

# test 数据集恶意 IP 计数
malware_ip_count = 0
for i in range(ip.shape[0]):
    if ip.loc[i,1] == 'black':
        malware_ip_count += 1
print("test数据集恶意ip个数" + str(malware_ip_count) )

# 将最终结果写入 result.txt
ip.to_csv('result.txt',sep=',',index=False,header=None)
```

## 总结与反思

这次比赛几次提交点的最好成绩是 72.5 分，距离复赛要求的名次差几名，很遗憾无缘复赛。总结了失利的几点原因：

1. 对恶意流量的特征不熟悉，导致花了很多时间去确定需要提取哪些 Features，浪费了前面的检查点。其实我们最后提取的 Features 还有很大的提升空间，比如数据包的时间序列特征，我们只提取了均值、方差等特征，还有字节分布等重要特征未进行有效提取。
![e67c9834b783a0c2b314e2dd35719671.png](5.png)
2. 对模型的选择和参数认识不够，只是盲目的更换模型和调参，一开始发现模型和参数对结果的影响很大，实际上还是特征提取的不够好。当提取了能有效区分正常/恶意流量的特征后，模型的选择和参数对结果影响就较小了，所以最重要的依然是有效特征的提取。

## 参考

- Shekhawat, A. S. (2018). *Analysis of Encrypted Malicious Traffic.*
- Jenseg, O. (2019). *A machine learning approach to detecting malware in TLS traffic using resilient network features (Master's thesis, NTNU).*
- Shekhawat, A. S., Di Troia, F., & Stamp, M. (2019). *Feature analysis of encrypted malicious traffic. Expert Systems with Applications, 125, 130-141.*
- [Analysing PCAPs with Bro/Zeek](https://medium.com/@melanijan93/https-medium-com-melanijan93-analysing-pcaps-with-bro-zeek-33340e710012)
- [Machine Learning Applications in Misuse and Anomaly Detection](https://www.intechopen.com/online-first/machine-learning-applications-in-misuse-and-anomaly-detection)
- [imbalanced-classification-with-python](https://machinelearningmastery.com/imbalanced-classification-with-python/)
- [scikit-learn GitHub doc: Feature extraction](https://github.com/scikit-learn/scikit-learn/blob/master/doc/modules/feature_extraction.rst)
- [基于机器学习的TLS恶意加密流量检测方案](https://www.secrss.com/articles/18679)
- [利用背景流量数据（contexual flow data）识别TLS加密恶意流量](https://www.cnblogs.com/bonelee/p/9604530.html)