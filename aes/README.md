# Projiect9

## AES简介

高级加密标准（英语：Advanced Encryption Standard，缩写：AES），在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。经过五年的甄选流程，高级加密标准由美国国家标准与技术研究院（NIST）于2001年11月26日发布于FIPS PUB 197，并在2002年5月26日成为有效的标准。2006年，高级加密标准已然成为对称密钥加密中最流行的算法之一。

## AES流程

### 加解密流程

AES的整体结构如下图所示，其中的W[0,3]是指W[0]、W[1]、W[2]和W[3]串联组成的128位密钥。加密的第1轮到第9轮的轮函数一样，包括4个操作：字节代换、行位移、列混合和轮密钥加。最后一轮迭代不执行列混合。另外，在第一轮迭代之前，先将明文和原始密钥进行一次异或加密操作。

![image](https://github.com/1-14/Project9/blob/main/aes/png/11.png)

上图也展示了AES解密过程，解密过程仍为10轮，每一轮的操作是加密操作的逆操作。由于AES的4个轮操作都是可逆的，因此，解密操作的一轮就是顺序执行逆行移位、逆字节代换、轮密钥加和逆列混合。同加密操作类似，最后一轮不执行逆列混合，在第1轮解密之前，要执行1次密钥加操作。

### 轮函数

#### S盒

![image](https://github.com/1-14/Project9/blob/main/aes/png/4.png)

AES的字节代换其实就是一个简单的查表操作。AES定义了一个S盒和一个逆S盒。加密时，矩阵中的元素按照S盒映射为一个新的字节：把该字节的高4位作为行值，低4位作为列值，取出S盒中对应的行的元素作为输出。解密进行逆操作，按照S盒进行映射。

#### 行移位

行移位是一个简单的左循环移位操作。加密时，如果密钥长度为128比特，状态矩阵的第0行左移0字节，第1行左移1字节，第2行左移2字节，第3行左移3字节，如下图所示：

![image](https://github.com/1-14/Project9/blob/main/aes/png/5.png)

解密时，行移位的逆变换是将状态矩阵中的每一行执行相反的移位操作，例如AES-128中，状态矩阵的第0行右移0字节，第1行右移1字节，第2行右移2字节，第3行右移3字节。

#### 列混合

列混合变换是通过矩阵相乘来实现的，加密时，经行移位后的状态矩阵与固定的矩阵相乘，得到混淆后的状态矩阵，如下图的公式所示：

![image](https://github.com/1-14/Project9/blob/main/aes/png/6.png)

其中，矩阵元素的乘法和加法都是定义在基于GF(2^8)上的二元运算,并不是通常意义上的乘法和加法。

解密时，混淆后的状态矩阵与加密时的逆矩阵相乘

#### 轮密钥加

轮密钥加是将128位轮密钥Ki同状态矩阵中的数据进行逐位异或操作，如下图所示。其中，密钥Ki中每个字W[4i],W[4i+1],W[4i+2],W[4i+3]为32位比特字，包含4个字节，他们的生成算法下面在下面介绍。轮密钥加过程可以看成是字逐位异或的结果，也可以看成字节级别或者位级别的操作。也就是说，可以看成a0 a1 a2 a3 组成的32位字与W[4i]的异或运算。

![image](https://github.com/1-14/Project9/blob/main/aes/png/7.png)

### 密钥扩展

AES首先将初始密钥输入到一个4*4的状态矩阵中，如下图所示。

![image](https://github.com/1-14/Project9/blob/main/aes/png/8.png)

这个4*4矩阵的每一列的4个字节组成一个字，矩阵4列的4个字依次命名为W[0]、W[1]、W[2]和W[3]，它们构成一个以字为单位的数组W。例如，设密钥K为"abcdefghijklmnop",则K0 = ‘a’,K1 = ‘b’, K2 = ‘c’,K3 = ‘d’,W[0] = “abcd”。
接着，对W数组扩充40个新列，构成总共44列的扩展密钥数组。新列以如下的递归方式产生：
1.如果i不是4的倍数，那么第i列由如下等式确定：
W[i]=W[i-4]⨁W[i-1]
2.如果i是4的倍数，那么第i列由如下等式确定：
W[i]=W[i-4]⨁T(W[i-1])
其中，T是一个有点复杂的函数。
函数T由3部分组成：字循环、字节代换和轮常量异或，这3部分的作用分别如下。
a.字循环：将1个字中的4个字节循环左移1个字节。即将输入字[b0, b1, b2, b3]变换成[b1,b2,b3,b0]。
b.字节代换：对字循环的结果使用S盒进行字节代换。
c.轮常量异或：将前两步的结果同轮常量Rcon[j]进行异或，其中j表示轮数。
轮常量Rcon[j]是一个字

### AES解密

AES解密除了上面的流程图中的正常解密步骤，还存在下图的解密步骤。

![image](https://github.com/1-14/Project9/blob/main/aes/png/12.png)

即逆向轮密钥加和逆向列混合可以交换顺序，逆向行移位和逆向S盒可以交换顺序。

### 雪崩效应

AES具有较好的雪崩效应，当明文改变一比特时，随着轮数增加，雪崩效应如下图所示

![image](https://github.com/1-14/Project9/blob/main/aes/png/13.png)

当密钥改变一比特时，随着轮数增加，雪崩效应如下图所示

![image](https://github.com/1-14/Project9/blob/main/aes/png/14.png)

### 工作模式

ECB模式

![image](https://github.com/1-14/Project9/blob/main/aes/png/15.png)

CBC模式

![image](https://github.com/1-14/Project9/blob/main/aes/png/16.png)

CFB模式

![image](https://github.com/1-14/Project9/blob/main/aes/png/17.png)

OFB模式

![image](https://github.com/1-14/Project9/blob/main/aes/png/18.png)

CTR模式

![image](https://github.com/1-14/Project9/blob/main/aes/png/19.png)

## 关键代码展示

可以选择四种不同工作模式的加密函数

```
int AESModeOfOperation::Encrypt(unsigned char* _in, int _length, unsigned char* _out) {
	bool first_round = true;
	int rounds = 0;
	int start = 0;
	int end = 0;
	unsigned char input[16] = { 0 };
	unsigned char output[16] = { 0 };
	unsigned char ciphertext[16] = { 0 };
	unsigned char cipherout[256] = { 0 };
	unsigned char plaintext[16] = { 0 };
	int co_index = 0;
	// 1. get rounds
	if (_length % 16 == 0) {
		rounds = _length / 16;
	}
	else {
		rounds = _length / 16 + 1;
	}
	// 2. for all rounds 
	for (int j = 0; j < rounds; ++j) {
		start = j * 16;
		end = j * 16 + 16;
		if (end > _length) end = _length;	// end of input
		// 3. copyt input to m_plaintext
		memset(plaintext, 0, 16);
		memcpy(plaintext, _in + start, end - start);
		// 4. handle all modes
		if (m_mode == MODE_CFB) {
			if (first_round == true) {
				m_aes->Cipher(m_iv, output);
				first_round = false;
			}
			else {
				m_aes->Cipher(input, output);
			}
			for (int i = 0; i < 16; ++i) {
				if ((end - start) - 1 < i) {
					ciphertext[i] = 0 ^ output[i];
				}
				else {
					ciphertext[i] = plaintext[i] ^ output[i];
				}
			}
			for (int k = 0; k < end - start; ++k) {
				cipherout[co_index++] = ciphertext[k];
			}
			//memset(input,0, 16);
			memcpy(input, ciphertext, 16);
		}
		else if (m_mode == MODE_OFB) {			// MODE_OFB
			if (first_round == true) {
				m_aes->Cipher(m_iv, output); // 
				first_round = false;
			}
			else {
				m_aes->Cipher(input, output);
			}
			// ciphertext = plaintext ^ output
			for (int i = 0; i < 16; ++i) {
				if ((end - start) - 1 < i) {
					ciphertext[i] = 0 ^ output[i];
				}
				else {
					ciphertext[i] = plaintext[i] ^ output[i];
				}
			}
			// 
			for (int k = 0; k < end - start; ++k) {
				cipherout[co_index++] = ciphertext[k];
			}
			//memset(input,0,16);
			memcpy(input, output, 16);
		}
		else if (m_mode == MODE_CBC) {			// MODE_CBC
			printf("-----plaintext------");
			print(plaintext, 16);
			printf("--------------------\n");
			//			printf("-----m_iv-----------\n");
			//			print (m_iv, 16);
			//			printf("--------------------\n");
			for (int i = 0; i < 16; ++i) {
				if (first_round == true) {
					input[i] = plaintext[i] ^ m_iv[i];
				}
				else {
					input[i] = plaintext[i] ^ ciphertext[i];
				}
			}
			first_round = false;
			//			printf("^^^^^^^^^^^^\n");
			//			print(input, 16);
			//			printf("^^^^^^^^^^^^\n");
			m_aes->Cipher(input, ciphertext);
			printf("****ciphertext****");
			print(ciphertext, 16);
			printf("************\n");
			for (int k = 0; k < end - start; ++k) {
				cipherout[co_index++] = ciphertext[k];
			}
			//memcpy(cipherout, ciphertext, 16);
			//co_index = 16;
		}
		else if (m_mode == MODE_ECB) {
			// TODO: 
		}
	}
	memcpy(_out, cipherout, co_index);
	return co_index;
}
```

可以选择四种不同工作模式的解密函数

```
int AESModeOfOperation::Decrypt(unsigned char* _in, int _length, unsigned char* _out) {
	// TODO :
	bool first_round = true;
	int rounds = 0;
	unsigned char ciphertext[16] = { 0 };
	unsigned char input[16] = { 0 };
	unsigned char output[16] = { 0 };
	unsigned char plaintext[16] = { 0 };
	unsigned char plainout[256] = { 0 };
	int po_index = 0;
	if (_length % 16 == 0) {
		rounds = _length / 16;
	}
	else {
		rounds = _length / 16 + 1;
	}

	int start = 0;
	int end = 0;

	for (int j = 0; j < rounds; j++) {
		start = j * 16;
		end = start + 16;
		if (end > _length) {
			end = _length;
		}
		memset(ciphertext, 0, 16);
		memcpy(ciphertext, _in + start, end - start);
		if (m_mode == MODE_CFB) {
			if (first_round == true) {
				m_aes->Cipher(m_iv, output);
				first_round = false;
			}
			else {
				m_aes->Cipher(input, output);
			}
			for (int i = 0; i < 16; i++) {
				if (end - start - 1 < i) {
					plaintext[i] = output[i] ^ 0;
				}
				else {
					plaintext[i] = output[i] ^ ciphertext[i];
				}
			}
			for (int k = 0; k < end - start; ++k) {
				plainout[po_index++] = plaintext[k];
			}
			//memset(input, 0, 16);
			memcpy(input, ciphertext, 16);
		}
		else if (m_mode == MODE_OFB) {
			if (first_round == true) {
				m_aes->Cipher(m_iv, output);
				first_round = false;
			}
			else {
				m_aes->Cipher(input, output);
			}
			for (int i = 0; i < 16; i++) {
				if (end - start - 1 < i) {
					plaintext[i] = 0 ^ ciphertext[i];
					first_round = false;
				}
				else {
					plaintext[i] = output[i] ^ ciphertext[i];
				}
			}
			for (int k = 0; k < end - start; ++k) {
				plainout[po_index++] = plaintext[k];
			}
			memcpy(input, output, 16);
		}
		else if (m_mode == MODE_CBC) {
			printf("------ciphertext------");
			print(ciphertext, 16);
			printf("----------------------\n");
			m_aes->InvCipher(ciphertext, output);
			printf("------output------");
			print(output, 16);
			printf("----------------------\n");
			for (int i = 0; i < 16; ++i) {
				if (first_round == true) {
					plaintext[i] = m_iv[i] ^ output[i];
				}
				else {
					plaintext[i] = input[i] ^ output[i];
				}
			}
			first_round = false;
			for (int k = 0; k < end - start; ++k) {
				plainout[po_index++] = plaintext[k];
			}
			memcpy(input, ciphertext, 16);
		}
		else {
			// TODO
		}
	}
	memcpy(_out, plainout, po_index);
	return po_index;
}
```

## 运行结果展示

### 加密

![image](https://github.com/1-14/Project9/blob/main/aes/png/1.png)

### 解密

![image](https://github.com/1-14/Project9/blob/main/aes/png/2.png)















