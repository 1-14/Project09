# Project9

## SM4

### SM4简介

最初作为我国自主无线局域网安全标准WAPI的专用密码算法于2006年公开发布

2012年3月成为国家密码行业标准，2016年8月成为我国商用密码标准算法中的分组密码算法。 2021年6月纳入ISO/IEC国际标准ISO/IEC 18033-3《加密算法 第3部分：分组密码 补篇2》

支持SM4算法的WAPI无线局域网芯片已超过350多个型号，全球累计出货量超过70亿颗。在金融领域，仅统计支持 SM4 算法的智能密码钥匙出货量已超过 1.5 亿个（2021年，数据来源于网络）

SM4的密钥长度和分组长度均为128比特，加密算法与密钥扩展算法都采用 32 轮非平衡 Feistel 结构。SM4算法的解密算法和加密算法的结构相同，除了轮密钥的使用顺序是加密轮密钥的逆序。

### SM4流程

#### 加密流程

![image](https://github.com/1-14/Project9/blob/main/sm4/3.png)

SM4 的加密算法和 DES、AES 的结构一样，均采用了基本函数迭代，但 SM4 也有一些不同的加密迭代处理特点。由图可以看出，SM4 的加密法代处理方式具有密文反馈连接和流密码的某一些特点，前一轮加密的结果与前一轮的加密数据拼接起来供下一轮加密处理。一次加密处理四个字，然后产生一个字的中间密文，这个中间密文和前三个字拼接起来后再供下一次加密处理，一共会迭代加密处理32 轮，产生出四个字的密文。这个加密处理的整个过程就像一个宽度为 4 个字的窗口在滑动，加密处理完一轮，窗口就滑动一个字，窗口一共滑动 32 次后加密迭代就结束了。


#### 轮函数

![image](https://github.com/1-14/Project9/blob/main/sm4/1.png)

#### 非线性部件

![image](https://github.com/1-14/Project9/blob/main/sm4/2.png)

#### 解密算法

![image](https://github.com/1-14/Project9/blob/main/sm4/1.png)

由于 SM4 密码算法的运算是对合运算，所以它的解密算法结构是和加密算法的结构一样的，不同的是轮密钥的使用顺序，解密和加密的是相反的，也就是说解密的轮密钥是加密的轮密钥的逆序。设输入的密文为(X0，X1，X2, X3)，输入轮密钥为 rki,i=31,30,…，1，0，输出的明文为(Y0，Y1，Y2, Y3)。

### 密钥扩展算法

SM4 密码算法采用 32 轮的迭代加密结构，拥有 128 位加密密钥，一共使用 32轮密钥，每一轮的加密使用 32 位的一个轮密钥。SM4 算法的特点使得它需要使用一个密钥扩展算法，在加密密钥当中产生 32 个轮密钥。在这个密钥的扩展算法当中有常数 FK、固定参数 CK 这两个数值，利用这两个数值便可完成它的这一个扩展算法。

1.常数 FK

密钥扩展当中使用的常数为以下几个：
FK0=(A3B1BAC6)，FK1=(56AA3350),FK2=(677D9197), FK3=(B27022DC)。

2.固定参数 CK

一共使用有 32 个固定参数 CKi，这个 CKi，是一个字，它的产生规则是：
设 cki ， j 为 CKi 的 第 j 字 节 (i=0 ， 1 ， … ， 31;j=0,1,2,3) ， 即CKj=(cki,0,cki,1,cki,2,cki,3),
则 cki，j=(4i+j)×7(mod 256)

这 32 个固定参数如下(十六进制) :

00070e15， 1c232a31， 383f64d， 545b6269，
70777e85， 8c939aa1， a8afb6bd， c4cbd2d9，
e0e7eef5， fc030a11， 181f262d， 343b4249，
50575e65， 6c737a81， 888f969d， a4abb2b9，
c0c7ced5， dce3eafl， f8ff060d， 141b2229，
30373e45， 4c535a61， 686f767d， 848b9299，
a0a7aeb5， bcc3cad1， d8dfe6ed， f4fb0209，
10171e25， 2c333a41， 484f565d， 646b7279。

3.密钥扩展算法

假设输入的加密密钥为 MK= (MK0，MK1，MK2，MK3 )，输出的轮密钥为 rki, i=0,1…,30,31，中间的数据为 Ki=0,1,…,34,35。则密钥扩展算法可描述如下。

密钥扩展算法：

①(K0，K1，K2，K3)=(MK0⊕FK0，MK1⊕FK1,MK2⊕FK2,MK3⊕FK3)
②For i=0,1,…,30,31 Do
rki=K(i+4)=Ki⊕T’（K(i+1)⊕K(i+2)⊕K(i+3)⊕CKi）

说明:其中的 T’变换与加密算法轮函数中的 T 基本相同，只将其中的线性变换L 修改为以下的 L’:

L’(B)=B⊕(B<<<13)⊕(B<<<23)

从密钥扩展算法中我们分析后可以发现，密钥扩展算法与加密算法在算法结构方面类似，同样都是采用了 32 轮类似的迭代处理。

需要特别注意的是密钥扩展算法采用了非线性变换 T，这个措施将会使密钥扩展的安全性大大地加强了。在这方面上 SM4 和 AES 密码类似，而 DES 的子密钥生产算法并没有采用这种类似措施。

## 关键代码展示

```
string encode(string plain, string key) {//加密函数实现
	cout << "轮密钥与每轮输出状态：" << endl;
	cout << endl;
	string cipher[36] = { plain.substr(0,8),plain.substr(8,8),plain.substr(16,8),plain.substr(24) };
	string rks = KeyExtension(key);
	for (int i = 0; i < 32; i++) {
		cipher[i + 4] = XOR(cipher[i], T(XOR(XOR(XOR(cipher[i + 1], cipher[i + 2]), cipher[i + 3]), rks.substr(8 * i, 8))));
		cout << "rk[" + to_string(i) + "] = " + rks.substr(8 * i, 8) + "    X[" + to_string(i) + "] = " + cipher[i + 4] << endl;
	}
	cout << endl;
	return cipher[35] + cipher[34] + cipher[33] + cipher[32];
}

string decode(string cipher, string key) {//解密函数实现
	cout << "轮密钥与每轮输出状态：" << endl;
	cout << endl;
	string plain[36] = { cipher.substr(0,8),cipher.substr(8,8), cipher.substr(16,8), cipher.substr(24,8) };
	string rks = KeyExtension(key);
	for (int i = 0; i < 32; i++) {
		plain[i + 4] = XOR(plain[i], T(XOR(XOR(XOR(plain[i + 1], plain[i + 2]), plain[i + 3]), rks.substr(8 * (31 - i), 8))));
		cout << "rk[" + to_string(i) + "] = " + rks.substr(8 * (31 - i), 8) + "    X[" + to_string(i) + "] = " + plain[i + 4] << endl;
	}
	cout << endl;
	return plain[35] + plain[34] + plain[33] + plain[32];
}
```

## 运行结果展示

### 加密

![image](https://github.com/1-14/Project9/blob/main/sm4/4.png)

### 解密

![image](https://github.com/1-14/Project9/blob/main/sm4/5.png)
