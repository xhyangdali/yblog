---
title: PHP使用3DES算法加密解密字符串
date: 2018-01-22 13:22:14
categories: PHP
tags:
	3DES
permalink: PHP使用3DES算法加密解密字符串
---
3DES（或称为Triple DES）是三重数据加密算法（TDEA，Triple Data Encryption Algorithm）块密码的通称。
它相当于是对每个数据块应用三次DES加密算法。
由于计算机运算能力的增强，原版DES密码的密钥长度变得容易被暴力破解；3DES即是设计用来提供一种相对简单的方法，即通过增加DES的密钥长度来避免类似的攻击，而不是设计一种全新的块密码算法。
<!-- more -->
## 为什么要进行数据加密
数据的安全性越来越得以重视。举个例子说，保存在数据库中的用户密码并不是明文保存的，而是采用md5加密后存储，这样即使数据库被脱库，仍能保证用户密码安全。
但是，md5是不可逆的，开发人员根本就不知道用户的密码到底是什么。
有些时候，我们希望加密后存储的数据是可逆的，比如一些接口密钥，这样即使数据库被脱库，如果没有对应的解密方式，攻击者盗取的密钥也是不能使用的。
## 3DES加密简介
3DES（即Triple DES）是DES向AES过渡的加密算法（1999年，NIST将3-DES指定为过渡的加密标准），加密算法，
其具体实现如下：设Ek()和Dk()代表DES算法的加密和解密过程，K代表DES算法使用的密钥，M代表明文，C代表密文，这样：

3DES加密过程为：C=Ek3(Dk2(Ek1(M)))

3DES解密过程为：M=Dk1(EK2(Dk3(C)))
## 使用PHP实现3DES加密
### 使用PHP实现3DES流程图
->Start
->mcrypt_module_open 打开算法和模式对应的模块
->mcrypt_generic_init 初始化加密所需的缓冲区
->mcrypt_generic 加密数据
->mcrypt_generic_deinit 对加密模块进行清理工作
->mcrypt_module_close 关闭加密模块

要使用以上的函数，在编译PHP的时候必须添加--with-mcrypt选项。
###  PHP实现3DES代码
```
/**
 * 3DES加解密类
 * @Author: yxh
 * @version: v1.0
 * 2018年1月19日
 */
class Encrypt
{
    //加密秘钥，
    private $_key;
    private $_iv;
    public function __construct($key, $iv)
    {
        $this->_key = $key;
        $this->_iv = $iv;
    }

    /**
     * 对字符串进行3DES加密
     * @param string 要加密的字符串
     * @return mixed 加密成功返回加密后的字符串，否则返回false
     */
    public function encrypt3DES($str)
    {
        $td = mcrypt_module_open(MCRYPT_3DES, "", MCRYPT_MODE_CBC, "");
        if ($td === false) {
            return false;
        }
        //检查加密key，iv的长度是否符合算法要求
        $key = $this->fixLen($this->_key, mcrypt_enc_get_key_size($td));
        $iv = $this->fixLen($this->_iv, mcrypt_enc_get_iv_size($td));

        //加密数据长度处理
        $str = $this->strPad($str, mcrypt_enc_get_block_size($td));

        if (mcrypt_generic_init($td, $key, $iv) !== 0) {
            return false;
        }
        $result = mcrypt_generic($td, $str);
        mcrypt_generic_deinit($td);
        mcrypt_module_close($td);
        return $result;
    }

    /**
     * 对加密的字符串进行3DES解密
     * @param string 要解密的字符串
     * @return mixed 加密成功返回加密后的字符串，否则返回false
     */
    public function decrypt3DES($str)
    {
        $td = mcrypt_module_open(MCRYPT_3DES, "", MCRYPT_MODE_CBC, "");
        if ($td === false) {
            return false;
        }

        //检查加密key，iv的长度是否符合算法要求
        $key = $this->fixLen($this->_key, mcrypt_enc_get_key_size($td));
        $iv = $this->fixLen($this->_iv, mcrypt_enc_get_iv_size($td));

        if (mcrypt_generic_init($td, $key, $iv) !== 0) {
            return false;
        }

        $result = mdecrypt_generic($td, $str);
        mcrypt_generic_deinit($td);
        mcrypt_module_close($td);

        return $this->strUnPad($result);
    }

    /**
     * 返回适合算法长度的key，iv字符串
     * @param string $str key或iv的值
     * @param int $td_len 符合条件的key或iv长度
     * @return string 返回处理后的key或iv值
     */
    private function fixLen($str, $td_len)
    {
        $str_len = strlen($str);
        if ($str_len > $td_len) {
            return substr($str, 0, $td_len);
        } else if($str_len < $td_len) {
            return str_pad($str, $td_len, '0');
        }
        return $str;
    }

    /**
     * 返回适合算法的分组大小的字符串长度，末尾使用\0补齐
     * @param string $str 要加密的字符串
     * @param int $td_group_len 符合算法的分组长度
     * @return string 返回处理后字符串
     */
    private function strPad($str, $td_group_len)
    {
        $padding_len = $td_group_len - (strlen($str) % $td_group_len);
        return str_pad($str, strlen($str) + $padding_len, "\0");
    }
    /**
     * 返回适合算法的分组大小的字符串长度，末尾使用\0补齐
     * @param string $str 要加密的字符串
     * @return string 返回处理后字符串
     */
    private function strUnPad($str)
    {
        return rtrim($str);
    }
   /**
    * 取消订单（对已锁定的座位解锁）
    */
   public function cancelorder($oid,$fromstationid,$orderid){
   		$header[] = 'appid:' . config('fjconfig.soapappid');
		$header[] = 'username:' . config("fjconfig.soapusername");
		$header[] = 'password:' . config('fjconfig.soappassword');
		$header[] = 'oid:' . $oid;
		$header[] = 'fromstationid:' . $fromstationid;
		$header[] = 'orderid :' . $orderid;
			
		\think\Log::record ($header, '**************submitdata');
							
		$return=linkInterface("applyorder",$header);
					
		\think\Log::record ($return, '**************cancelorder');
		
		return $return;
   }
}
```
### PHP实现3DES加密-解密调用
```
/*
 *3DES 加密
 *
 * @param type $str
 *
 * */
function encrypt3DES($str) {
    $key   = config("Encypt.key");
    $iv    = config("Encypt.iv");
    $des = new Encrypt($key, $iv);
    $e_str = $des->encrypt3DES($str);
    return urlencode($e_str);
}
/*
 *3DES 解密
 *
 * @param type $str
 *
 * */
function decrypt3DES($str) {
    $str = urldecode($str);
    $key   = config("Encypt.key");
    $iv    = config("Encypt.iv");
    $des = new Encrypt($key, $iv);
    $d_str = $des->decrypt3DES($str);
    return urlencode($d_str);
}
```
## 注意事项
注意，如果要在数据库中保存加密后的数据，建议base64_encode之后再保存，以下是PHP官网上的建议：
如果你在例如 MySQL 这样的数据库中存储数据， 请注意 varchar 类型的字段会在插入数据时自动移除字符串末尾的“空格”。 
由于加密后的数据可能是以空格（ASCII 32）结尾， 这种特性会导致数据损坏。 
请使用 tinyblob/tinytext（或 larger）字段来存储加密数据。