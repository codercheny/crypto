# 生成 RSA KEY

```sh
openssl genrsa -out rsa-openssl.pem 2048
```

>   KEY 与 PEM 格式 相同。

# 制作 PFX (P12) 文件

## 1. 生成 KEY

```sh
openssl genrsa -out server.key 2048
```

## 2. 生成证书

```sh
openssl req -new -x509 -key server.key -out server.crt -days 3650
```

## 3. 将 X509 格式的数字证书转换成微软的 PFX 格式

```sh
openssl pkcs12 -export -inkey server.key -in server.crt -out server.pfx
```

# 从 PFX 中提取私钥

将 PFX 证书转换为 PEM 格式文件

```sh
openssl pkcs12 -in server.pfx -out server.pem -nodes
```

从 PFX 格式文件中提取私钥格式文件（`*.key`）

```sh
openssl pkcs12 -in server.pfx -nocerts -nodes -out private.key
```


# 获取私钥
从私钥格式文件中导出私钥：
```sh
openssl rsa -in rsa-openssl.pem -out prikey.pem
```
```sh
# remove the password and convert to PKCS #8
openssl pkcs8 -nocrypt -in rsa-openssl.pem -inform PEM -topk8 -outform DER -out prikey.der
```


# 获取公钥
从私钥格式文件中导出公钥：
```sh
openssl rsa -in prikey.pem -pubout -out pubkey.pem
openssl rsa -in prikey.der -inform DER -pubout -out pubkey.pem
openssl rsa -in prikey.pem -pubout -outform DER -out pubkey.der
# 导出的公钥与前面指令导出的公钥不同
openssl rsa -in prikey.pem -RSAPublicKey_out -out pubkey.pem
```

# PEM 转换成 DER
```sh
openssl rsa -in prikey.pem -outform DER -out prikey.der
openssl rsa -in pubkey.pem -outform DER -pubin -out pubkey.der
```
```sh
openssl base64 -d -in prikey.pem -out prikey.der
openssl base64 -d -in pubkey.pem -out pubkey.der
```


# DER 转换成 PEM
```sh
openssl rsa -in prikey.der -inform DER -out prikey.pem
openssl rsa -in pubkey.der -inform DER -pubin -out pubkey.pem
```


## openssl 签名和验签
RSA 私钥签名、公钥验签
```sh
openssl rsautl -sign -inkey prikey.pem -in origin -out sign
openssl rsautl -verify -inkey pubkey.pem -pubin -in sign -out verify
```

sha1withrsa:
```sh
openssl dgst -sha1 -sign prikey.pem -out sign.sha1 origin
openssl dgst -sha1 -verify pubkey.pem -signature sign.sha1 origin
```
> 对应于 `RSA Signature Scheme PKCS v1.5`


## openssl 加密和解密
RSA 公钥加密、私钥解密
```sh
openssl rsautl -encrypt -in origin -inkey pubkey.pem -pubin -out encrypt
openssl rsautl -decrypt -in encrypt -inkey prikey.pem -out decrypt
```


## 明文输出
以明文形式输出密钥的各个参数值到文件：
```sh
openssl rsa -in pubkey.pem -out public.txt -pubin -text
openssl rsa -in pubkey.pem -out public.txt -RSAPublicKey_in -text
openssl rsa -in prikey.pem -out private.txt -text
openssl rsa -in prikey.pem -out private.txt -pubout -text
```

以明文形式输出密钥的各个参数值到窗口：
```sh
openssl rsa -in pubkey.pem -pubin -noout -text
openssl rsa -in pubkey.pem -RSAPublicKey_in -noout -text
openssl rsa -in prikey.pem -noout -text
```


# 从证书中导出公钥

```sh
openssl x509 -inform der -in cert.cer -pubkey -out public.pem
```
