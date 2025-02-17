# 如何获取token

## 1. 打开PC端开发者模式

```bash:terminal
DEBUG=APP* OPEN_DEVTOOLS_IN_PRODUCTION=1 /Applications/AICoin.app/Contents/MacOS/AICoin
```

## 2. 获取token

在`开发者工具`中，选`网络`，选`载荷`，`右键`，`复制值`。

## 3. 解密token

### 3.1 修改测试单元

```go:go-pc-api/test/decrypt_test.go
// 将以下值替换为从PC端开发者工具中复制的值
json := `{
    "p": "4Po7jd5ukqqyBfJZfyXdpxCM49t3hpqD5Ltw9MzP+qhRqob3ILe1dyjqSLxqv5EjOmZjz6J39SQwTw95WzCpRYFxdVf6XqynJZq42jMC1duqgxTNDcjo8eiycInXZUgdLM/vWNq24AgWg0t6PV6uqnPRgjGFEhHtHZOE+k6shUfXOHxFroJvaMphJ6RiGa3+E/zi7UAIP+RI3MfOyXCDME3aP8VTcFkhCTZqd9KRQpf/NQBp3ZV4jrnOQuCbbZBW/5ZvbAki/Ou8K2hZ5NG86bTeUjMfZ7FX2BssDg9LhwUd8qhL8Ut4hfcOzxZlEHu/EXSEbwGq+UNQeXQ4rC+nqB0RY9d+pW+ogJFlTY1kVhtOfmvn7QNFLb2um0xbnr4gvjnSRhEqIxF2o5qr1hT9yLm461HHYtETMMFdaTO38ae4zmQBnMEpAPcPyB0JaFuta73MFwlPp4gEbQUUaECBowQnoP8cOYqbaLNw68feQWs=",
    "k": "DXE1KS2QgZff3rhySqbI8raLfnql8qanBIxePSUcd11/xMlb2N9ihHKC1Ds6d/W+w/FLRqDs2FPdL2hXI9pFIGHi9a+qHInKGc20q389gd2J+FToT82aVKyjcwirm2SoCj6EykEAtY79XI3HJJOwjO1ylVnGMXjKv0NrvaYUK4k=",
    "v": "1535898394",
    "iv": "V1wcaPjzDw+a0MM09X7Wzqap6nYaqsz1mh1jiNeXwj2+qoTwbBMpT79V106iGu/kZLDup566aBtKkmvEicbSsX9EcYowqE+cP6HDUYsVf55d3RUwIpxWGSA0LL+YYozgSkKfcDH405gcSoUYc1RTV2/LHEDLGYR1+9HVKz1VFSA="
}`
```

### 3.2 运行测试单元

```bash:terminal
go test -v test/decrypt_test.go
```
