## 开发准备

### SDK 获取
对象存储服务的 XML Android SDK 的下载github地址:[XML Android SDK](https://github.com/tencentyun/qcloud-sdk-android/releases).

更多示例可参考Demo:[XML Android SDK Demo](https://github.com/tencentyun/qcloud-sdk-android-samples).

### 开发准备

1. SDK 支持 Android 2.2 及以上版本的手机系统；
2. 手机必须要有网络（GPRS、3G或 WIFI 网络等）；
3. 手机可以没有存储空间，但会使部分功能无法正常工作；
4. 从控制台获取APPID、SecretID、SecretKey。

### SDK 配置

配置工程导入下列 jar 包：

- cos-xml-android-sdk-1.2.jar

- qcloud-core-1.2.jar
 
- okhttp-3.8.1.jar

- okio-1.13.0.jar
 
- xstream-1.4.7.jar

- fastjson-1.1.62.android.jar

SDK 需要网络、存储等相关的一些访问权限，需要在 AndroidManifest.xml 中增加如下权限声明：
```html
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
```


## 快速入门 

### 初始化 CosXmlService 和 CosXmlServiceConfig
进行操作之前需要实例化 CosXmlService 和 CosXmlServiceConfig.

````java
String appid = "对象存储 的服务APPID";
String region = "存储桶 所在的地域"; 

String secretId = "云 API 密钥 secretId";
String secretKey ="云 API 密钥 secretKey";
long keyDuration = 600; //secretKey的有效时间,单位秒

//创建 CosXmlServiceConfig 对象，根据需要修改默认的配置参数
CosXmlServiceConfig serviceConfig = new CosXmlServiceConfig.Builder()
       .setAppidAndRegion(appid, region)
       .setDebuggable(true)
       .setConnectionTimeout(45000)
       .setSocketTimeout(30000)
       .build();

//创建获取签名类
LocalCredentialProvider localCredentialProvider = new LocalCredentialProvider(secretId, secretKey, keyDuration);

//创建 CosXmlService 对象，实现对象存储服务各项操作.
Context context = getApplicationContext()； //应用的上下文

CosXmlService cosXmlService = new CosXmlService(context,cosXmlServiceConfig, localCredentialProvider);
````

### 简单上传文件

````java
String bucket = "存储桶名称";
String cosPath = "远端路径，即存储到cos上的绝对路径"; //格式如 cosPath = "/test.txt";
String srcPath = "本地文件的绝对路径"; // 如 srcPath = Environment.getExternalStorageDirectory().getPath() + "/test.txt";
long signDuration = 600; //签名的有效期，单位为秒

PutObjectRequest putObjectRequest = new PutObjectRequest(bucket, cosPath, srcPath);

putObjectRequest.setSign(signDuration,null,null);

//设置进度显示
putObjectRequest.setProgressListener(new QCloudProgressListener() {
    @Override
    public void onProgress(long progress, long max) {
        float result = (float) (progress * 100.0/max);
        Log.w("TEST","progress =" + (long)result + "%");
    }
});

//使用同步方法上传
try {
    PutObjectResult putObjectResult = cosXmlService.putObject(putObjectRequest);

		Log.w("TEST","success: " + putObjectResult.accessUrl);
    
  } catch (CosXmlClientException e) {

	   //抛出异常
       Log.w("TEST","CosXmlClientException =" + e.toString());
  } catch (CosXmlServiceException e) {

	   //抛出异常
       Log.w("TEST","CosXmlServiceException =" + e.toString());
}

//使用异步回调上传：sdk为对象存储各项服务提供异步回调操作方法
/**

cosXmlService.putObjectAsync(putObjectRequest, new CosXmlResultListener() {
     @Override
     public void onSuccess(CosXmlRequest request, CosXmlResult result) {
      	 Log.w("TEST","success =" + result.accessUrl);
     }

     @Override
     public void onFail(CosXmlRequest cosXmlRequest, CosXmlClientException clientException, CosXmlServiceException serviceException)  {
        
		String errorMsg = clientException != null ? clientException.toString() : serviceException.toString();
		Log.w("TEST",errorMsg);
    }
});

*/

````

### 分片上传文件
````java
//分片上传一般需要经历：初始化分片上传->分块上传->完成等3个阶段.

String bucket = "存储桶名称";
String cosPath = "远端路径，即存储到cos上的绝对路径";


//第一步，初始化分片上传，获取 uploadId

String uploadId = null;

InitMultipartUploadRequest initMultipartUploadRequest =  new InitMultipartUploadRequest(bucket, cosPath);

initMultipartUploadRequest.setSign(600,null,null);
try {
	InitMultipartUploadResult initMultipartUploadResult =
                  cosXmlService.initMultipartUpload(initMultipartUploadRequest);

	//若初始化成功，则获取uploadId;
   Log.w("TEST","success");
   uploadId = initMultipartUploadResult.initMultipartUpload.uploadId;

} catch (CosXmlClientException e) {

	   //抛出异常
       Log.w("TEST","CosXmlClientException =" + e.toString());
  } catch (CosXmlServiceException e) {

	   //抛出异常
       Log.w("TEST","CosXmlServiceException =" + e.toString());
}

//第二步，分片上传，需要参数 uploadId 和分片号 partNumber; 并获取对应的 eTag # 此处只演示只有一个分片的文件例子 #.

String srcPath = "本地文件的绝对路径";
int partNumber = 1; //上传分片编码，从1开始； 此处演示上传第一个分片

String eTag = null;

UploadPartRequest uploadPartRequest = new UploadPartRequest(bucket, cosPath, partNumber,
srcPath, uploadId);

uploadPartRequest.setSign(600,null,null);

uploadPartRequest.setProgressListener(new QCloudProgressListener() {
	 @Override
	 public void onProgress(long progress, long max) {
	     float result = (float) (progress * 100.0/max);
	     Log.w("TEST","progress =" + (long)result + "%");
	   }
	});

try {
	UploadPartResult uploadPartResult = cosXmlService.uploadPart(uploadPartRequest); 
	
	Log.w("TEST","success");	
	eTag = uploadPartResult.getETag(); // 获取分片文件的 eTag

  } catch (CosXmlClientException e) {

	   //抛出异常
       Log.w("TEST","CosXmlClientException =" + e.toString());
  } catch (CosXmlServiceException e) {

	   //抛出异常
       Log.w("TEST","CosXmlServiceException =" + e.toString());
}

//第三步，当确定所有分片全部上传完成之后，调用CompleteMultiUploadRequest完成分片上传结束.
//需要参数 uploadId， partNumber和对应每块分片文件的eTag值


CompleteMultiUploadRequest completeMultiUploadRequest = new CompleteMultiUploadRequest(bucket, cosPath, uploadId, null);

completeMultiUploadRequest.setPartNumberAndEtag(partNumber, eTag); //此处只演示一个分片的例子

completeMultiUploadRequest.setSign(600,null,null);
try {
	CompleteMultiUploadResult completeMultiUploadResult =
	                    cosXmlService.completeMultiUpload(completeMultiUploadRequest);

	Log.w("TEST","success: " + completeMultiUploadResult.accessUrl );

  } catch (CosXmlClientException e) {

	   //抛出异常
       Log.w("TEST","CosXmlClientException =" + e.toString());
  } catch (CosXmlServiceException e) {

	   //抛出异常
       Log.w("TEST","CosXmlServiceException =" + e.toString());
}
````
 
### 下载文件
````java

String bucket = "存储桶名称";
String cosPath = "远端路径，即存储到cos上的绝对路径";
String savePath = "下载到本地的路径";

GetObjectRequest getObjectRequest = GetObjectRequest(bucket, cosPath, savePath);

getObjectRequest.setSign(signDuration,null,null);
getObjectRequest.setProgressListener(new QCloudProgressListener() {
    @Override
    public void onProgress(long progress, long max) {
        float result = (float) (progress * 100.0/max);
        Log.w("TEST","progress =" + (long)result + "%");
    }
});

//使用同步方法下载
try {
   GetObjectResult getObjectResult =cosXmlService.getObject(getObjectRequest);
	
	Log.w("TEST","success： " + getObjectResult.xCOSStorageClass);
       
  } catch (CosXmlClientException e) {

      Log.w("TEST","CosXmlClientException =" + e.toString());

   } catch (CosXmlServiceException e) {

       Log.w("TEST","CosXmlServiceException =" + e.toString());
}


//**使用异步回调请求**
/**

cosXmlService.getObjectAsync(getObjectRequest, new CosXmlResultListener() {
    @Override
    public void onSuccess(CosXmlRequest cosXmlRequest, CosXmlResult cosXmlResult) {

		Log.w("TEST","success");
     }

     @Override
     public void onFail(CosXmlRequest cosXmlRequest, CosXmlClientException clientException, CosXmlServiceException serviceException)  {
        
		String errorMsg = clientException != null ? clientException.toString() : serviceException.toString();
		Log.w("TEST",errorMsg);
    }
});

*/
````


## 生成签名

签名具体的生成和使用请参照[签名流程](https://www.qcloud.com/document/product/436/7778).

SDK中已提供了签名获取类，用户只需要继承 BasicLifecycleCredentialProvider 类，并重写 fetchNewCredentials() 方法.

#### 示例
````java
public class LocalCredentialProvider extends BasicLifecycleCredentialProvider{
    private String secretKey;
    private long keyDuration;
    private String secretId;

     public LocalCredentialProvider(String secretId, String secretKey, long keyDuration) {
        this.secretId = secretId;
        this.secretKey = secretKey;
        this.keyDuration = keyDuration;
     }

     @Override
     public QCloudLifecycleCredentials fetchNewCredentials() throws CosXmlClientException {
         long current = System.currentTimeMillis() / 1000L;
         long expired = current + duration;
         String keyTime = current+";"+expired;
         return new BasicQCloudCredentials(secretId, secretKeyToSignKey(secretKey, keyTime), keyTime);
     }

     private String secretKeyToSignKey(String secretKey, String keyTime) {
         String signKey = null;
         try {
              if (secretKey == null) {
                   throw new IllegalArgumentException("secretKey is null");
              }
              if (keyTime == null) {
                    throw new IllegalArgumentException("qKeyTime is null");
              }
         } catch (IllegalArgumentException e) {
                e.printStackTrace();
         }
         try {
             byte[] byteKey = secretKey.getBytes("utf-8");
             SecretKey hmacKey = new SecretKeySpec(byteKey, "HmacSHA1");
             Mac mac = Mac.getInstance("HmacSHA1");
             mac.init(hmacKey);
             signKey = StringUtils.toHexString(mac.doFinal(keyTime.getBytes("utf-8")));
        } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
        } catch (NoSuchAlgorithmException e) {
                e.printStackTrace();
        } catch (InvalidKeyException e) {
                e.printStackTrace();
        }
      return signKey;
    }
}
````