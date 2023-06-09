fscrypt
=========

目录
----

-  `文件加密 <#文件加密>`__
-  `fscrypt <#fscrypt>`__
-  `支持的加密算法 <#支持的加密算法>`__
-  `使用fscrypt加密文件 <#使用fscrypt加密文件>`__

   -  `前置条件 <#前置条件>`__
   -  `step1 给文件系统enable encrypt
      属性 <#step1--给文件系统enable--encrypt-属性>`__
   -  `step2 fscrypt 初始化 <#step2-fscrypt-初始化>`__
   -  `step3 加密目录 <#step3-加密目录>`__
   -  `step4 lock/unlock 目录 <#step4--lockunlock--目录>`__

-  `PAM 模块 <#PAM-模块>`__

文件加密
--------

通常我们会以文件作为数据载体，使用磁盘，USB 闪存，SD卡等存储介质进行数据存储，即便数据已经离线存储，
仍然不能保证该存储介质不会丢失，如果丢失那么对于我们来说有可能是灾难性的事件。因此对这些离线存储的重
要数据文件进行加密是非常有必要的。

fscrypt
-------

内核中的 fscrypt
是一个库，文件系统可以使用它以支持文件和目录的透明加密。

与 dm-crypt 不同，fscrypt 在文件系统级别而不是块设备级别运行。
这允许它使用不同的密钥加密不同的文件，并在同一文件系统上拥有未加密的文件。
这对于多用户系统非常有用，在该系统中，每个用户的静态数据都需要与其他用户进行加密隔离。
除了文件名，fscrypt 不加密文件系统的元数据。

与作为栈式文件系统的 eCryptfs 不同，fscrypt
是直接集成到支持的文件系统中，目前支持 fscrypt
的文件系统是\ **ext4、F2FS 和 UBIFS**

fscrypt
允许读取和写入加密文件，而无需在页面缓存中同时缓存解密和加密页面，从而将使用的内存几乎减半并使其与未加密文件保持一致。
同样，需要一半的 dentry 和 inode。 

eCryptfs 还将加密文件名限制为 143 字节，从而导致应用程序兼容性问题；
fscrypt 允许完整的 255 个字节 (NAME_MAX)长度的文件名。 最后，与 eCryptfs
不同，fscrypt API 可以由非特权用户使用，而无需依赖其它任何组件。

fscrypt 不支持就地加密文件。 相反，它支持将空目录标记为已加密。
然后，在用户空间提供密钥后，在该目录树中创建的所有常规文件、目录和符号链接都将被透明地加密。

支持的加密算法
----------------

fscrypt 允许为文件内容指定一种加密模式，为文件名指定一种加密模式。
不同的目录树允许使用不同的加密方式。 目前支持以下几种加密方式对：

AES-256-XTS for contents and AES-256-CTS-CBC for filenames

AES-128-CBC for contents and AES-128-CTS-CBC for filenames

Adiantum for both contents and filenames

AES-256-XTS for contents and AES-256-HCTR2 for filenames (v2 policies
only)

SM4-XTS for contents and SM4-CTS-CBC for filenames (v2 policies only)

If unsure, you should use the (AES-256-XTS, AES-256-CTS-CBC) pair.

AES-128-CBC 仅为具有不支持 XTS 模式的加速器的低功耗嵌入式设备使用。
要使用 AES-128-CBC，必须启用 CONFIG_CRYPTO_ESSIV 和
CONFIG_CRYPTO_SHA256（或其他 SHA-256 实现）以便使用 ESSIV。

Adiantum 是一种基于流密码的模式，即使在没有专用加密指令的 CPU 上也很快。
与 XTS 不同，它也是真正的宽块模式。
它还可以消除派生每个文件加密密钥的需要。 要使用 Adiantum，必须启用
CONFIG_CRYPTO_ADIANTUM。 此外，应启用 ChaCha 和 NHPoly1305
的快速实现，例如 ARM 架构上的 CONFIG_CRYPTO_CHACHA20_NEON 和
CONFIG_CRYPTO_NHPOLY1305_NEON。

AES-256-HCTR2 是另一种真正的宽块加密模式，旨在用于具有专用加密指令的
CPU。 AES-256-HCTR2 具有明文中的位翻转会更改整个密文的属性。
由于初始化向量在目录中重复使用，因此此属性使其成为文件名加密的理想选择。
要使用 AES-256-HCTR2，必须启用 CONFIG_CRYPTO_HCTR2。 此外，应启用 XCTR
和 POLYVAL 的快速实现，例如 用于 ARM64 的 CRYPTO_POLYVAL_ARM64_CE 和
CRYPTO_AES_ARM64_CE_BLK。

最后是 SM4 算法，目前仅在 fscrypt v2 策略中启用。 
::

   fscrypt v2 版本只在5.4 以上内核支持

https://www.kernel.org/doc/html/latest/filesystems/fscrypt.html#encryption-modes-and-usage

使用fscrypt加密文件
-----------------------------

前置条件
----------------

内核配置

-  CONFIG_EXT4_FS_ENCRYPTION=y
-  CONFIG_F2FS_FS_ENCRYPTION=y
-  CONFIG_FS_ENCRYPTION=m

文件系统 

-  ext4

step1 给文件系统enable encrypt 属性
-----------------------------------

对于ext4，要在其上使用加密的文件系统必须启用加密功能标志。要启用它，请运行：

   > tune2fs -O encrypt /dev/DEVICE

如果是新建的一个文件系统， 可以使用 mkfs.ext4 -O encrypt

step2 fscrypt 初始化
--------------------

   > fscrypt setup 

这个命令会创建文件 /etc/fscrypt.conf 还有一个目录/.fscrypt 

如果要加密的文件不是根文件系统， 运行：

   > mount /dev/DEVICE /home/uos/mnt

..

   > fscrypt setup /home/uos/mnt

这个命令会创建文件 /home/uos/mnt/.fscrypt

/etc/fscrypt.conf 配置文件内容如下：

.. code:: json

   {
     "source": "custom_passphrase", //默认保护原  pam_passphrase、custom_passphrase、raw_key
     "hash_costs": {  // 描述密封hash的难度
       "time": "52",
       "memory": "131072",
       "parallelism": "32"
     },
     "options": { // 加密选项
       "padding": "32", //加密文件的填充字节数 32、16、8、4
       "contents": "AES_256_XTS",  //加密文件内容的算法
       "filenames": "AES_256_CTS", // 加密文件目录名字的算法
       "policy_version": "2"  // 加密策略的版本， 只有5.4 以上的内核才支持2
     },
     "use_fs_keyring_for_v1_policies": false,  //此设置的目的是允许人们利用与较旧内核兼容的加密目录上Linux V5.4的一些改进
     "allow_cross_user_metadata": false  //指定FSCRYPT是否允许阅读其他非根本用户的保护者和策略，例如fscrypt加密作为选项提供
   }

step3 加密目录
--------------

.. code:: bash

   >  mkdir   /home/uos/mnt/dir
   >  fscrypt encrypt  /home/uos/mnt/dir
   >  sudo fscrypt encrypt /home/uos/mnt/dir  --user=uos
   The following protector sources are available:
   1 - Your login passphrase (pam_passphrase)  # 使用用户登录的口令，需要配置相应的pam 模块
   2 - A custom passphrase (custom_passphrase) # 使用用户自定义的口令
   3 - A raw 256-bit key (raw_key)          # 其他256位的key
   Enter the source number for the new protector [2 - custom_passphrase]: 2
   Enter a name for the new protector: fish
   Enter custom passphrase for protector "fish": 
   Confirm passphrase: 
   "/home/uos/test/fscrypt_mnt/dir" is now encrypted, unlocked, and ready for use.

**Tip : 被加密的目录必须为空, 如果要加密已有内容的目录， 需要先移出去**

.. code:: bash

   > mkdir new_dir
   > fscrypt encrypt new_dir
   > cp -a -T old_dir new_dir
   > find old_dir -type f -print0 | xargs -0 shred -n1 --remove=unlink
   > rm -rf old_dir

step4 lock/unlock 目录
----------------------

准备工作：

.. code:: bash

   > cd  /home/uos/mnt/dir
   > mkdir aaa
   > echo "hello world" > aaa/a.txt
   > cd ..
   > tree .
   .
   └── dir
       └── aaa
           └── a.txt

   2 directories, 1 file

lock

.. code:: bash

   > sudo fscrypt lock ./dir/  --user=uos
   Encrypted data removed from filesystem cache.
   "./dir/" is now locked.

   > tree .
   .
   └── dir
       └── FlXWBy9XVFYB3+EXd9tsBX7kNGpj4Vz0RGhmw4QfJ4K
           └── tCJi0VMPGkfLM6iPnqbi9YAYnErm2nPMvJLMrYtwxwB

   2 directories, 1 file


   > cd dir/FlXWBy9XVFYB3+EXd9tsBX7kNGpj4Vz0RGhmw4QfJ4K/
   > cat tCJi0VMPGkfLM6iPnqbi9YAYnErm2nPMvJLMrYtwxwB 
   cat: tCJi0VMPGkfLM6iPnqbi9YAYnErm2nPMvJLMrYtwxwB: 需要的关键字不存在

   > cd -

status 

.. code:: bash

   > fscrypt status dir
   "dir" is encrypted with fscrypt.

   Policy:   e7f93b6219638134
   Options:  padding:32 contents:AES_256_XTS filenames:AES_256_CTS policy_version:1 
   Unlocked: No

   Protected with 1 protector:
   PROTECTOR         LINKED  DESCRIPTION
   575642ae2964c6d4  No      custom protector "fish"

unlock

.. code:: bash

   > sudo fscrypt unlock  dir --user=uos
   Enter custom passphrase for protector "fish": 
   "dir" is now unlocked and ready for use.
   > tree .
   .
   └── dir
       └── aaa
           └── a.txt

   2 directories, 1 file

   > cat  dir/aaa/a.txt 
   hello world

PAM 模块
--------

要在登录时自动解锁受登录密码短语保护的目录，并使受登录密码保护的目录与登录密码短语的更改保持同步，请调整系统PAM配置以启用PAM_fscrypt。

.. code:: bash

    /etc/pam.d/login
    
    auth  option  pam_fscrypt.so

代码地址

https://github.com/google/fscrypt

https://www.kernel.org/doc/html/latest/filesystems/fscrypt.html

https://openanolis.github.io/whitebook-shangmi/kernel_fscrypt.html
