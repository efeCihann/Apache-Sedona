# Python ile Apache-Sedona Dökümantasyonu (TR)

## 1.Giriş 

### Apache Sedona 

Apache Sedona mekansal verilerin işlenmesi için optimize edilmiş bir küme hesaplama sistemidir. Büyük ölçekli mekansal verilerin verimli bir şekilde yüklenmesini, işlenmesini ve analiz edilmesini sağlar. Apache Flink ve Snowflake gibi birden fazla dağıtılmış hesaplama motorunu destekleyerek paralel veri işleme ortamlarında en kapsamlı mekansal SQL desteğini sunar.

### Bulut Sağlayıcıları

Apache Sedona; AWS, Databricks, Azure ve Google Cloud gibi farklı bulut sağlayıcıları ile çalışabilir. 

### Veri Formatları

Apache Sedona; GeoJSON, WKT/WKB, GeoParquet, Shapefile gibi birçok farklı vektör veri formatlarının yanında GeoTIFF gibi raster veri formatlarını da destekler.

## 2.Kurulum 


### 2.1 Kütüphane

**Sedona’yı PySpark ile birlikte indirmek için:**

 ` pip install apache-sedona[spark]`

Komutu ile PySparkı da dahil edererek Apache Sedona indirilebilir. 

**Eğer halihazırda PySpark yüklü ise:**

`pip install apache-sedona`

İle apache-sedona kütüphanesini indirebilirsiniz.Ancak bu durumda Spark sürümünüz ile Python sürümünüzün, Java sürümünüzün ve Scala sürümünüzün uyumlu olduğuna dikkat edilmesi gerekir.

### 2.2 Yapılandırma ve Entegrasyon

Yapılandırmaya başlamadan önce sürümlerin uyumluluğundan emin olunmalıdır. Sürümlerin uyumluluğu aşağıda yer alan tablolardan kontrol edilebilir.

Sedona’nın Spark ile çalışabilmesi için doğru JAR dosyalarının kullanılması gerekir.

**Yapılandırma için gerekli JAR dosyaları:**

**sedona-spark-shaded** veya **sedona-spark :** Sedona’nın Scala ve Spark sürümüne uygun jar dosyasıdır.  

Eğer Sedona AWS EMR, Databricks gibi bulut platformlarda kullanılacaksa **shaded jar,** yerel ortamlarda çalıştırılackasa **unshaded jar** kullanılmalıdır.

**geotools-wrapper:**  Coğrafi verileri işlemek için gereken jar dosyasıdır. 

Maven Central Koordinatlarını kullanarak gerekli jar dosyalarını otomatik olarak indirebiliriz. Jar Dosyaları Maven Central üzerinden manuel olarak da indirilip  SPARK\_HOME klasörüne koyulabilir. 

**Maven Central Koordinatları ile jra dosyalarını indirmek için:**

Belirtildiği gibi iki jar dosyası gereklidir:

**org.apache.sedona:sedona-spark-3.3\_2.12:1.7.0**  : Spark 3.3 ve Scala 2.12 ve Sedona 1.7.0 ile uyumlu **sedona-spark jar dosyası.**

**org.datasyslab:geotools-wrapper:1.7.0-28.5**  :  Sedona 1.7.0 sürümü ve geotools-wrapper kütüphanesinin 28.5 sürümü ile uyumlu jar dosyasıdır.


Sedona ile geotools-wrapper uygun sürümleri için:

 **[**https://central.sonatype.com/artifact/org.datasyslab/geotools-wrapper/versions**](https://central.sonatype.com/artifact/org.datasyslab/geotools-wrapper/versions)**

**spark.jars.packages** :  Belirtilen bağımlılıkları (JAR dosyalarını) Maven Central’dan otomatik olarak indirip yükler.



 ****

**Gerekli bağımlılıkları yüklemek için yapılandırma ayarımız:**

    ```   config('spark.jars.packages',
           'org.apache.sedona:sedona-spark-3.3_2.12:1.7.0,'
           'org.datasyslab:geotools-wrapper:1.7.0-28.5')"```


 

Spark bağımlılıkları indirmek için Maven Central deposunu kullanır ancak bazı bağımlılıklar başka Maven depolarında bulunabilir. Örneğin bazı sürümler için gerekli GeoTools Wrapper bağımlılıkları **Unidata Maven** ‘da bulunur. Bu yüzden ek bir Maven deposu olarak Unidata Maven’ı belirtmemiz gerekir.

**Bağımlılıkların indirilebileceği ek bir Maven deposu tanıtmak için yapılandırma ayarımız:**

``config('spark.jars.repositories', 'https\://artifacts.unidata.ucar.edu/repository/unidata-all')``

 **Yapılandırmamız ileSpark oturumumuzu oluşturursak:**

```
from sedona.spark import *
config = SedonaContext.builder(). \
    config('spark.jars.packages',
           'org.apache.sedona:sedona-spark-3.3_2.12:1.7.0,'
           'org.datasyslab:geotools-wrapper:1.7.0-28.5'). \
    config('spark.jars.repositories', 'https://artifacts.unidata.ucar.edu/repository/unidata-all'). \
	getOrCreate()
 ```


 

**Apache Sedona’yı mevcut spark oturumuna entegre etmek için:**

`sedona = SedonaContext.create(config)`

 




