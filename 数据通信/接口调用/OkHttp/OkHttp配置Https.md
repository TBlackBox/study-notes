# Https配置

对于https,需要配置sslSocketFactory，一般如需忽略所有证书的话，可以这样配置：

```java
OkHttpClient client = new OkHttpClient.Builder()
        // 设置https配置，此处忽略了所有证书
        .sslSocketFactory(createEasySSLContext().getSocketFactory(), new EasyX509TrustManager(null))  
        .build();  
```
## SSLContext方法
```java
private static SSLContext createEasySSLContext() throws IOException {
    try {
        SSLContext context = SSLContext.getInstance("TLS");
        context.init(null, null, null);
        return context;
    } catch (Exception e) {
        throw new IOException(e.getMessage());
    }
}
```
## EasyX509TrustManager类
```java
public class EasyX509TrustManager implements X509TrustManager {

    private X509TrustManager standardTrustManager = null;

    /**
     * Constructor for EasyX509TrustManager.
     */
    public EasyX509TrustManager(KeyStore keystore) throws NoSuchAlgorithmException,
            KeyStoreException {
        super();
        TrustManagerFactory factory = TrustManagerFactory.getInstance(TrustManagerFactory
                .getDefaultAlgorithm());
        factory.init(keystore);
        TrustManager[] trustmanagers = factory.getTrustManagers();
        if (trustmanagers.length == 0) {
            throw new NoSuchAlgorithmException("no trust manager found");
        }
        this.standardTrustManager = (X509TrustManager)trustmanagers[0];
    }

    /**
     * @see X509TrustManager#checkClientTrusted(X509Certificate[],
     *      String authType)
     */
    public void checkClientTrusted(X509Certificate[] certificates, String authType)
            throws CertificateException {
        standardTrustManager.checkClientTrusted(certificates, authType);
    }

    /**
     * @see X509TrustManager#checkServerTrusted(X509Certificate[],
     *      String authType)
     */
    public void checkServerTrusted(X509Certificate[] certificates, String authType)
            throws CertificateException {
        if ((certificates != null) && (certificates.length == 1)) {
            certificates[0].checkValidity();
        } else {
            standardTrustManager.checkServerTrusted(certificates, authType);
        }
    }

    /**
     * @see X509TrustManager#getAcceptedIssuers()
     */
    public X509Certificate[] getAcceptedIssuers() {
        return this.standardTrustManager.getAcceptedIssuers();
    }
}
```

此外还可以使用以下代码忽略本地校验url正确性：

```java
.hostnameVerifier(new HostnameVerifier() {
    @Override
    public boolean verify(String hostname, SSLSession session) {
        return true;
    }
})  // 验证服务器的证书域名。在https握手期间，如果 URL 的主机名和服务器的标识主机名不匹配，则验证机制可以回调此接口的实现程序来确
```

