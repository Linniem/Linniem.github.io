---
title: "[python] 為 protoc 產生的檔案加入型別定義檔"
date: 2022-11-27 13:51:16
tags: 
    - python
    - protobuf
    - grpc
---

先說方法，使用版本至少`3.20.0`的 protoc 產生檔案時帶參數 `--pyi_out` 可以在指定路徑(跟產生出的pb.py檔放一起)產生對應的 pyi 檔案供編輯器讀取，這樣開發時就可以由編輯器帶出對應的 class 以及 object 的 fields。

# 問題

工作上要使用到 grpc，卻發現 protoc 預設產生的檔案沒有辦法讓編輯器讀取出型別，除了在 import 時無法自動帶出有哪些 class 可用，連 object 都無法帶出對應欄位，開發體驗非常糟糕。

> 如果你是在 `2022/11/04` 之後照著 [grpc 的官方教學](https://grpc.io/docs/languages/python/quickstart/)來使用 protobuf 的話應該就不會遇到這個問題
> [官方文件更新的 pr](https://github.com/grpc/grpc.io/pull/1068)

{% asset_img no-type-hints.jpg %}

> 上圖是官方的範例 然後是還沒有型別定義檔的情況

如果在 vscode 中開啟 type checking 會警告說 `helloworld_pb2` 沒有 `HelloRequest` 這個 member，自然也不知道初始化要帶哪些值進去，`helloworld_pb2` 也無法自動提示有哪些 class 可用。

開發時要不斷去跟 proto 檔比對型別，還有錯字的風險。

# 解法

確保使用的 protoc 版本至少要 `3.20.0` 以上

在呼叫 protoc 的指令加上 `--pyi_out` 參數 這裡直接引用[官方的教學](https://grpc.io/docs/languages/python/quickstart/#generate-grpc-code)

``` bash
$ python -m grpc_tools.protoc -I../../protos --python_out=. --pyi_out=. --grpc_python_out=. ../../protos/helloworld.proto
```

然後就可以看到編輯器提示 `helloworld_pb2` 有 `HelloRequest` 跟 `HelloReply`

{% asset_img after2.jpg %}

`HelloRequest` 也會告訴你初始化需要帶什麼值了

{% asset_img after1.jpg %}

# 官方 grpc 的限制

雖然 import 進來的 protobuf class 有型別了，但 grpc stub 的 service 卻還是 `Unknown`，導致編輯器沒辦法顯示 service 的 input 需要什麼型別，output 也是 `Unknown`，這部分還是麻煩一點。

{% asset_img still-no-type.jpg %}

如果使用 `grpclib` 取代官方的 `grpc` ，那麼就不會有上述問題，輸入輸出都有對應的型別而非`Unknown`。

{% asset_img grpclib.jpg %}

# 相關 issue/pr

新增 type hints 的 [issue](https://github.com/protocolbuffers/protobuf/issues/2638) 是在 2017年初提出的。

直到 2022 年 4 月於 v3.20.0 新增 `--pyi_out` 參數產生型別檔案 ([ 相關 comment ](https://github.com/protocolbuffers/protobuf/issues/2638#issuecomment-1106725624))

到可以用這個參數的 [release](https://github.com/protocolbuffers/protobuf/releases/tag/v3.20.3) 已經是 2022/09/30 的事了 

~~這些人開發時一直看 proto 都沒問題 太強了~~

這個 issue 實在是等了很久

# 補充

如果團隊將 protoc 輸出的檔案放到獨立的 repo 變成 package 供其他專案 import，會發現 pyi 檔案不會包含在已經 install 的 package 內，導致 type hints 沒有效果。

> pyi 檔案是不影響程式的功能的，要視為 data file。

將 data file 也包含進 package 的方式請參考  [setuptools 的說明](https://setuptools.pypa.io/en/latest/userguide/datafiles.html)
