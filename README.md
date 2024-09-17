
![](https://img2024.cnblogs.com/blog/1769816/202409/1769816-20240916175324517-1955058201.png)


## 一、说明


随着信息安全的重要性日益凸显，数字证书在各种安全通信场景中扮演着至关重要的角色。国密算法，作为我国自主研发的加密算法标准，其应用也愈发广泛。然而，在Java环境中解析使用国密算法的数字证书时，我们可能会遇到一些挑战。


本文主要分享如何在 `Java` 中解析采用 `SM3WITHSM2` 签发算法的国密数字证书。


 


## 二、问题背景


数字证书通常遵循 `X.509` 格式标准，而在 `Java` 中，我们通常使用 `java.security` 包下的工具来解析这些证书。但是，当证书采用了国密算法，如 `SM3WITHSM2` 时，标准的 `Java` 库可能无法识别这种算法特定的椭圆曲线，因此在解析时会抛出异常。


例如，尝试使用以下代码解析一个采用国密算法的证书时：



```
CertificateFactory cf = CertificateFactory.getInstance("X509");
String filePath ="C:\\Users\\example\\Desktop\\ca.crt";
FileInputStream in =new FileInputStream(filePath);
X509Certificate cer = (X509Certificate) cf.generateCertificate(in);

```

可能会遇到如下错误：



```
java.security.cert.CertificateParsingException: java.io.IOException: Unknown named curve: 1.2.156.10197.1.301

```

这个错误表明 `Java` 标准库无法识别国密算法使用的椭圆曲线。


 


## 三、解决方案


为了解决这个问题，我们需要借助 `BouncyCastle` 这个强大的加密库，它提供了对多种加密算法的支持，包括国密算法。


#### 步骤 1：添加BouncyCastle依赖


首先，需要将 `BouncyCastle` 库添加到项目中，在 `pom.xml` 中添加以下依赖：



```
<dependency>
		<groupId>org.bouncycastlegroupId>
		<artifactId>bcprov-jdk15onartifactId>
		<version>1.62version>
dependency>

```

#### 步骤 2：修改代码以使用BouncyCastle


接下来需要修改代码，以便在解析证书时使用 `BouncyCastle` 提供者：



```
// 引入BC库
Security.addProvider(new BouncyCastleProvider());
// 使用BC解析X.509证书
CertificateFactory cf = CertificateFactory.getInstance("X509", "BC");

```

完整的测试代码如下：



```
import org.bouncycastle.jce.provider.BouncyCastleProvider;  
import java.security.Security;  
import java.security.cert.CertificateFactory;  
import java.security.cert.X509Certificate;  
import java.io.FileInputStream;  
  
public class SMCertificateParser {  
    public static void main(String[] args) {  
        try {  
            // 注册BouncyCastle提供者  
            Security.addProvider(new BouncyCastleProvider());  
              
            // 使用BouncyCastle提供者解析X.509证书  
            CertificateFactory cf = CertificateFactory.getInstance("X509", "BC");  
            String filePath = "C:\\Users\\example\\Desktop\\ca.crt";  
            FileInputStream in = new FileInputStream(filePath);  
            X509Certificate cer = (X509Certificate) cf.generateCertificate(in);  
              
            // 打印证书信息  
            System.out.println("版本号：" + cer.getVersion());  
            System.out.println("序列号：" + cer.getSerialNumber().toString());  
            System.out.println("有效期：from：" + cer.getNotBefore() + "  to: " + cer.getNotAfter());  
            System.out.println("签发算法：" + cer.getSigAlgName());  
            System.out.println("签发算法ID：" + cer.getSigAlgOID());  
              
            in.close();  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
    }  
}

```

执行程序后，输出以下信息：



```
版本号：3
序列号：228766466093659650410797181222534438848
有效期：from：Mon Mar 13 17:31:00 CST 2023  to: Mon Feb 23 17:31:00 CST 2093
签发算法：SM3WITHSM2
签发算法ID：1.2.156.10197.1.501

```

 


## 四、结论


通过引入 `BouncyCastle` 库并修改代码以使用该库，我们现在能够成功解析采用国密 `SM3WITHSM2` 算法的数字证书。这一解决方案不仅限于 `SM3WITHSM2` 还适用于其他国密算法或任何非标准算法，只要 `BouncyCastle` 库支持这些算法。


**扫码关注有惊喜！**


![](https://img2024.cnblogs.com/blog/1769816/202409/1769816-20240916180034863-1490358595.png)


 本博客参考[milou加速器](https://xinminxuehui.org)。转载请注明出处！
