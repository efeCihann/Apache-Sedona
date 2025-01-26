# Sedona SQL
SedonaSQL mekansal sorguların optimize edilmesini sağlayan dağıtık bir mekansal veri işleme motorudur. 

## Geometri Oluşturma Fonksiyonları (Constructors)

Verilen girdi ile geometri nesnesi oluşturulmasını sağlarlar.

En yaygın kullanılan constructor operatörler:

### ST_Point(x,y)

Verilen koordinat değerleri ile point geometrisi oluşturur.

### ST_PolygonFromText(wkt, srid)

WKT formatında verilen metin ile polygon geometrisi oluşturur. Opsiyonel olarak referans değeri de verilebilir (srid)

### ST_LineStringFromText(wkt, srid)

WKT formatında verilen değerler ile LineString geometrisi oluşturur. Opsiyonel olarak referans değeri de verilebilir (srid)

### ST_GeomFromWKT(wkt)

WKT formatındaki herhangi bir geometrik değer içeren metinden geometri oluşturur. 

### ST_GeomFromGeoJSON(geojson)

GeoJSON formatında verilen metin ile geometri oluşturur.

### ST_MPointFromText(wkt,srid)

WKT formatında verilen metin ile MultiPoint geometrisi oluşturur. Opsiyonel olarak referans değeri de verilebilir (srid).

### ST_MLineFromText

WKT formatında verilen metin ile MultiLineString geometrisi oluşturur. Opsiyonel olarak referans değeri de verilebilir (srid).

### ST_MPolyFromText

WKT formatında verilen metin ile MultiPolygon geometrisi oluşturur. Opsiyonel olarak referans değeri de verilebilir (srid).


Bazı örnek kullanımlar: 

Örnek:

```
cokgen = sedona.sql("""
	SELECT   ST_PolygonFromText('-74.0428197, 40.6867969, -74.0421975, 40.6921336, -74.0508020, 40.6912794, -74.0428197, 40.6867969', ',') AS cokgen
""")

cokgen.show(truncate=False)
```
Çıktı:
```
+---------------------------------------------------------------------------------------------------------+
|cokgen                                                                                                   |
+---------------------------------------------------------------------------------------------------------+
|POLYGON ((-74.0428197 40.6867969, -74.0421975 40.6921336, -74.050802 40.6912794, -74.0428197 40.6867969))|
+---------------------------------------------------------------------------------------------------------+
```
Metinsel olarak girdiğimiz değerler ile çokgen oluşturmuş olduk. 

Örnek:
```
cizgiler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")  
	.option("header", "false")
	.load("data/cizgiler.csv")
)

#Çizgiler WKT formatında olduğu için ayırıcı olarak /n kullanılarak her bir LineString bir satır olacak şekilde yüklendi

cizgiler_df.createOrReplaceTempView("cizgi_tablo")

cizgiler = sedona.sql(
	"SELECT ST_GeomFromWKT(cizgi_tablo._c0) AS Cizgiler FROM cizgi_tablo "
)


cizgiler.printSchema()
```

Çıktımız:
```
root
 |-- Cizgiler: geometry (nullable = true)
```
Böylece WKT formatındaki LineString verilerini geometri nesnesi olarak yükledik. 


## Fonksiyonlar

## Geometrik Hesaplama Fonksiyonları

Sedona SQL içerisinde birçok geometrik hesaplamaya olanak tanır. Aşağıda bazı fonksiyonlar ve kullanım örnekleri verilmiştir.

Diğer fonksiyonlar  https://sedona.apache.org/latest/api/sql/Function/ adresindeki liste içerisinden incelenebilir. 

### ST_Distance()

ST_Distance() iki geometri arasındaki mesafeyi hesaplamak için kullanılır. Parametre olarak arasındaki mesafenin ölçülmek istendiği geometriler verilir.

Not: Sedona default olarak EPSG:4326 koordinat sistemini kullanır. Bu nedenle elde ettiğimiz sonuçlar derece cinsindendir. 

Not: İki geometrinin birbirine en yakın noktaları arasındaki mesafe sonuç olarak döner. Eğer bu geometriler iç içeyse sonuç sıfır olacaktır.

Örnek kullanım:

nokta1 = 'POINT (0 0)'
nokta2= 'POINT (3 4)'

```
sonuc = sedona.sql(f"""
	SELECT ST_Distance(ST_GeomFromText('{nokta1}'), ST_GeomFromText('{nokta2}')) AS mesafe
""")


sonuc.show()
```
Çıktımız:
```
+------+
|mesafe|
+------+
|   5.0|
+------+
```

### ST_AREA()

ST_AREA() poligon ve multipoligon türündeki geometrilerin 2 boyutlu alanını hesaplar.


Not: Poligon deliklere sahipse bu deliklerin alanı toplam alandan çıkarılır.

Not: Multipoligonlar için toplam alan döndürülür.

Örnek kullanım:
```
cokgen = 'POLYGON((0 0, 5 0, 5 5, 0 5, 0 0))'

cokgen_alan = sedona.sql(f""" SELECT ST_AREA(ST_GeomFromText('{polygon}')) AS alan""")

cokgen_alan.show()
```
Çıktımız:
```
+----+
|alan|
+----+
|16.0|
+----+
```


## Dönüşüm Fonksiyonları
Sedona SQL’de birçok veri formatı arasında dönüşüm yapılabilir.Aşağıda bazı dönüşüm  fonksiyonları için örnekler verilmiştir.

Diğer fonksiyonlar  https://sedona.apache.org/latest/api/sql/Function/ adresindeki liste içerisinden incelenebilir.  

Örnek:

ST_Point ile oluşturulan bir nokta geometrisini ST_AsText() ile WKT formatına dönüştürelim.
```
wkt_donusum = sedona.sql(
	"""SELECT
	ST_AsText(ST_Point(1, 2)) AS wkt"""
)

wkt_donusum.show(truncate=False)
```
Çıktımız: 
```
+-----------+
|wkt        |
+-----------+
|POINT (1 2)|
+-----------+
```

Örnek:

Çokgen verilerinin olduğu bir .csv dosyasındaki çokgenleri birleştirerek bu birleşik çokgeni ST_AsGeoJSON ile  GeoJSON formatına dönüştürelim.
```
cokgenler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")
	.option("header", "false")
	.load("data/poligonlar.csv")
)

cokgenler_df.createOrReplaceTempView("cokgenler_tablo")



geojson_donusum = sedona.sql(
	"""SELECT
    	ST_AsGeoJSON(ST_Union_Aggr(ST_GeomFromWKT(_c0))) AS geojson
	FROM cokgenler_tablo """
)

geojson_donusum.show(truncate=False)
```
Çıktımız:
```
+--------------------------------------------------------------------------------------------------------------------------------------------------+
|geojson                                                                                                                                           |
+--------------------------------------------------------------------------------------------------------------------------------------------------+
|{"type":"Polygon","coordinates":[[[6.0,4.0],[6.0,1.0],[3.5,1.0],[3.5,2.0],[2.0,2.0],[2.0,4.0],[3.0,4.0],[3.0,5.0],[5.0,5.0],[5.0,4.0],[6.0,4.0]]]}|
+--------------------------------------------------------------------------------------------------------------------------------------------------+
```
Örnek:

Aynı dosyadaki çokgenleri Feature olarak .geojson’a dönüştürmek istersek:
```
geojson2 = sedona.sql(
	"""SELECT
   	ST_AsGeoJSON(ST_GeomFromWKT(_c0), "Feature") AS gejson2	 
  	FROM cokgenler_tablo
 """
)
geojson2 .show(truncate=False)
```
Çıktımız:
```
+------------------------------------------------------------------------------------------------------------------------------------+
|gejson2                                                                                                                             |
+------------------------------------------------------------------------------------------------------------------------------------+
|{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[2.0,2.0],[4.0,2.0],[4.0,4.0],[2.0,4.0],[2.0,2.0]]]},"properties":{}}|
|{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[3.0,3.0],[5.0,3.0],[5.0,5.0],[3.0,5.0],[3.0,3.0]]]},"properties":{}}|
|{"type":"Feature","geometry":{"type":"Polygon","coordinates":[[[3.5,1.0],[6.0,1.0],[6.0,4.0],[3.5,4.0],[3.5,1.0]]]},"properties":{}}|
```
Aynı şekilde FeatureCollection da oluşturulabilirdi. 


## Geometrik Özellikler
Sedona SQL ile geometrilerin birçok özelliği incelenebilir. Geometrik özelliklerin belirlenmesi ve incelenmesi için kullanılan bazı fonksiyonlara ve örnek kullanımlarına aşağıda yer verilmiştir. 

Diğer fonksiyonlar  https://sedona.apache.org/latest/api/sql/Function/ adresindeki liste içerisinden incelenebilir.  

Örnek

Elimizde bulunan çokgen verilerinin:
ST_GeometryType() ile hangi geometri olduğunu, 
ST_Centroid() ile ağırlık merkezini,
ST_Boundary() ile sınırlarını ,
ST_IsSimple() ile geometrinin basit yapıda olup olmadığını 
inceleyelim.
```
cokgenler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")
	.option("header", "false")
	.load("data/poligonlar.csv")
)

cokgenler_df.createOrReplaceTempView("cokgenler_tablo")

geo_ozellik = sedona.sql(
	"""SELECT
   	ST_GeometryType(ST_GeomFromWKT(_c0)) AS Geometri,
   	ST_Centroid(ST_GeomFromWKT(_c0)) AS Merkez,
   	ST_Boundary(ST_GeomFromWKT(_c0)) AS Sınırlar,
   	ST_IsSimple(ST_GeomFromWKT(_c0)) AS Basitlik
  	 
  	FROM cokgenler_tablo
 """
)
geo_ozellik .show(truncate=False)
```
Çıktı:
```
+----------+----------------+------------------------------------------+--------+
|Geometri  |Merkez          |Sınırlar                                  |Basitlik|
+----------+----------------+------------------------------------------+--------+
|ST_Polygon|POINT (3 3)     |LINESTRING (2 2, 4 2, 4 4, 2 4, 2 2)      |true    |
|ST_Polygon|POINT (4 4)     |LINESTRING (3 3, 5 3, 5 5, 3 5, 3 3)      |true    |
|ST_Polygon|POINT (4.75 2.5)|LINESTRING (3.5 1, 6 1, 6 4, 3.5 4, 3.5 1)|true    |
+----------+----------------+------------------------------------------+--------+
```


## Bütünsel Fonksiyonlar

ST_Envelope_Aggr()

Bir geometri kolonundaki tüm geometrileri kapsayan en küçük dikdörtgeni döndürür.

Örnek:
```
kapsayan_pencere = sedona.sql(
	"""SELECT
	ST_Envelope_Aggr(ST_GeomFromWKT(cizgi_tablo._c0)) AS kapsam
	FROM cizgi_tablo """
)
```
Çıktı:
```
+-----------------------------------------------------------+
|kapsam                                                     |
+-----------------------------------------------------------+
|POLYGON ((1.1 100, 1.1 109.9, 9.8 109.9, 9.8 100, 1.1 100))|
+-----------------------------------------------------------+
```
ST_Intersection_Aggr()

Bir geometri kolonundaki tüm geometrilerin kesişimini döndürür.

ST_Union_Aggr()

Bir geometri kolonundaki tüm geometrilerin birleşimini tek bir geometri olarak döndürür.
 
Örnek:
```
cokgenler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")
	.option("header", "false")
	.load("data/poligonlar.csv")
)

cokgenler_df.createOrReplaceTempView("cokgenler_tablo")

cokgenler = sedona.sql(
	"""SELECT
   	ST_Union_Aggr(ST_GeomFromWKT(cokgenler_tablo._c0)) AS CokgenBirlesim,   
   	ST_Intersection_Aggr(ST_GeomFromWKT(cokgenler_tablo._c0)) AS CokgenKesisim
  	FROM cokgenler_tablo
 """
)
cokgenler.show(truncate=False)
```
Çıktımız:
```
+---------------------------------------------------------------------+-----------------------------------------+
|CokgenBirlesim                                                       |CokgenKesisim                            |
+---------------------------------------------------------------------+-----------------------------------------+
|POLYGON ((6 4, 6 1, 3.5 1, 3.5 2, 2 2, 2 4, 3 4, 3 5, 5 5, 5 4, 6 4))|POLYGON ((3.5 4, 4 4, 4 3, 3.5 3, 3.5 4))|
+---------------------------------------------------------------------+-----------------------------------------+
```
Birden fazla çokgenin olduğu bir veriden tüm çokgenlerin birleşimi olan tek bir çokgen elde etmiş olduk. Aynı zamanda bu çokgenlerin kesişimini döndürdük. 


##	Predicates 

Sedona SQL içerisinde coğrafi ilişkileri incelemek ve belirlemek için için birçok fonksiyon bulunur.Aşağıda bazı temel fonksiyonlar ve örnek kullanımları verilmiştir.

Diğer fonksiyonlar  https://sedona.apache.org/latest/api/sql/Function/ adresindeki liste içerisinden incelenebilir. 

Örnekler:
```
cokgen = 'POLYGON((1 1, 4 1, 4 4, 1 4, 1 1))'

cizgi = 'LINESTRING(2 0, 5 5)'

nokta = 'POINT(3 3)'
```
Oluşturduğumuz geometrilerin görüntüsü aşağıdaki gibi olmaktadır:

![Apache_Sedona_Predicates_Example_1](./gorseller/ApacheSedona_Predicates_1.png)


Temel topolojik operasyonlar bu örnek geometriler üzerinden anlatılacak.

ST_Contains( )

Bir geometri diğerini tamamen içeriyorsa “True” döner.

Not: Eğer geom_Y ‘nin geom_X dışında kalan kısımları varsa veya geom_Y geom_X’in sınırında yer alıyorsa  ST_Contains(geom_X, geom_Y )  “False” döner.

Örnek:
```
cont_cokgen_cizgi = sedona.sql(f"""SELECT ST_Contains(ST_GeomFromText("{cokgen}") ,ST_GeomFromText("{cizgi}")) AS Dahil"""  )

cont_cokgen_cizgi.show()
```
Çıktı: 
```
+-----+
|Dahil|
+-----+
|false|
+-----+
```
Görselde görüldüğü gibi çizgi geometrimizin çokgen geometrisinin dışında kalan kısımları olduğu için sonucumuz False döndü.

Nokta geometrimiz için de incelersek:
```
cont_cokgen_nokta = sedona.sql(f"""SELECT ST_Contains(ST_GeomFromText("{cokgen}") ,ST_GeomFromText("{nokta}")) AS Dahil"""  )

cont_cokgen_nokta.show()
```
Çıktı:
```
+-----+
|Dahil|
+-----+
| true|
+-----+
```
Beklendiği gibi çokgen geometrimiz nokta geometrimizin tamamını içerdiği için True döndü.

ST_Intersects( )

Bİr geometrinin başka bir geometri ile kesişim durumunu kontrol etmek için kullanılır.

Örnek:
```
int_cokgen_cizgi = sedona.sql(f"""SELECT ST_Intersects(ST_GeomFromText("{cokgen}"),
                 	ST_GeomFromText('{cizgi}')) AS kesisim""")
int_cokgen_cizgi.show()
```
Çıktı: 
```
+-------+
|kesisim|
+-------+
|   true|
+-------+
```
Şekil x.x ‘de görüldüğü gibi çokgen geometrimiz ile çizgi geometrimiz kesiştiği için True döndü.

Nokta ile çizgi için incelersek:
```
int_nokta_cizgi = sedona.sql(f"""SELECT ST_Intersects(ST_GeomFromText("{nokta1}"),
                 	ST_GeomFromText('{line1}')) AS kesisim""")
int_nokta_cizgi.show()
```

Çıktı:
```
+-------+
|kesisim|
+-------+
|  false|
+-------+
```
Beklendiği gibi çizgi geometrimiz ile nokta geometrimiz arasında herhangi bir temas olmadığı için False döndü.


ST_Touches

İki geometrinin yalnızca sınırlarının temas edip etmediğini kontrol eder.

Örnek:
```
touch_cokgen_cizgi = sedona.sql(f"""SELECT ST_Touches(ST_GeomFromText("{cokgen}") ,ST_GeomFromText("{cizgi}")) AS Dahil"""  )

touch_cokgen_cizgi.show()
```
Çıktı:
```
+-----+
|Dahil|
+-----+
|false|
+-----+
```
Çizgi geometrimiz her ne kadar çokgen ile kesişse ve içinden geçse de, ST_Touches() fonksiyonunun true dönmesi için sadece sınır teması gereklidir. Bu yüzden False çıktısı aldık.

Nokta için incelersek: 
```
touch_cokgen_nokta = sedona.sql(f"""SELECT ST_Touches(ST_GeomFromText("{cokgen}") ,ST_GeomFromText("{nokta}")) AS Dokunuyor"""  )

touch_cokgen_nokta.show()
```
Çıktı:
```
+---------+
|Dokunuyor|
+---------+
|    false|
+---------+
```
Nokta geometrimiz de çokgen geometrisinin içinde kaldığı ve sınır temasında bulunmadığı için beklendiği gibi çıktımız False oldu.

Çokgenimizin köşe sınırında yeni bir nokta tanımlayıp deneyelim:
```
nokta2 = 'POINT(4 4)'
touch_cokgen_nokta2 = sedona.sql(f"""SELECT ST_Touches(ST_GeomFromText("{cokgen}") ,ST_GeomFromText("{nokta2}")) AS Dokunuyor"""  )

touch_cokgen_nokta2.show()
```
Çıktı:
```
+---------+
|Dokunuyor|
+---------+
|    true|
+---------+
```
Bu sefer geometrilerimiz sadece sınır temasında bulunduğu için çıktımız true oldu.

##	SedonaSQL Veri Görselleştirme

### SedonaPyDeck
SedonaPyDeck, Jupyter Notebook ve JupyterLab gibi ortamlarda mekansal verilerin etkileşimli bir şekilde görselleştirilmesi için etkili bir araçtır.  
PyDeck mekansal verilerin coğrafi olarak görselleştirmesi için kullanılan güçlü bir kütüphanedir. SedonaPyDeck ile bu kütüphane Sedona ile entegre bir şekilde kullanılabilir. 

Aşağıda bazı SedonaPyDeck fonksiyonları ve kullanım örnekleri verilmiştir. 
```
create_geometry_map()
```
Geometrilerin harita üzerinde görselleştirilmesini sağlar. 

Örnek: 
Ankara’da bir bölgeyi temsil eden çokgeni harita üzerinde görselleştirelim:
```
 ankara_ornek = sedona.sql(
	"""SELECT
  	 
   	ST_GeomFromWKT('POLYGON((
32.8540 39.9200,
32.8600 39.9220,
32.8580 39.9260,
32.8520 39.9250,
32.8500 39.9210,
32.8540 39.9200
))') AS cokgen
  	 
	"""
)
 ankara_ornek .show(truncate=False)
 
         	 
r = SedonaPyDeck.create_geometry_map(df= ankara_ornek, fill_color="[85, 183, 177, 255]", line_color="[85, 183, 177, 255]",stroked=True)
r.show()
```
Çıktı:

![Apache_Sedona_PyDeck_Example_1](./gorseller/SedonaPyDeck_1.png)

```
create_choropleth_map()
```
Belirli sütundaki değerlere göre renkli haritalar oluşturmak için kullanılır.

Örnek:
Ülkelerin 2005 yılındaki nüfus verilerini içeren FeatureCollection yapısındaki bir geojson dosyasından nüfus değerlerine göre renklendirilmiş bir harita oluşturalım.
```
from pyspark.sql.functions import explode, col, expr

#GeoJSON ‘dan verilerimizi yükleyelim:
ulke_nufus = (
	sedona.read.format("geojson")
	.option("multiLine", "true")
    
	.load("data/world-population.geo.json")
 
)

#FeatureCollections’dan Feature’ları ayırdık
ulke_nufus = ulke_nufus.selectExpr("explode(features) as features")

#Geometri ve nüfus verilerimiz
ulke_nufus  = ulke_nufus.select(
	col("features.geometry").alias("geometry"),
	col("features.properties.POP2005").alias("POP2005")   
)


r= SedonaPyDeck.create_choropleth_map(df=ulke_nufus,   fill_color='[0, 0, 255, POP2005 * 0.000001]')

r.show()
```

Çıktı:

![Apache_Sedona_PyDeck_Example_2](./gorseller/SedonaPyDeck_2.png)



```
create_scatterplot_map()
```
Nokta geometrilerinin harita üzerinde görselleştirilmesi için kullanılır.

Örnek:

Çokgen verilerimizin ağırlık merkezlerini görselleştirelim
```
cokgenler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")
	.option("header", "false")
	.load("data/poligonlar.csv")
)

cokgenler_df.createOrReplaceTempView("cokgenler_tablo")

 

geo_merkez = sedona.sql(
	"""SELECT
  	 
   	ST_Centroid (ST_GeomFromWKT(_c0)) AS Merkez,
   	ST_GeomFromWKT(_c0) AS Geometri
  	 

  	FROM cokgenler_tablo
 """
)
geo_merkez.show(truncate=False)

 
       	 
r = SedonaPyDeck.create_scatterplot_map(df=geo_merkez.select("Merkez"), fill_color="[85, 183, 177, 255]",  radius_scale=100000)
 

r.show()
```
Çıktı:

![Apache_Sedona_PyDeck_Example_3](./gorseller/SedonaPyDeck_3.png)

```
create_heatmap()
```
Mekansal verilerin yoğunluk dağılımını göstermek için kullanılır.




### SedonaKepler
SedonaKepler KeplerGL üzerine inşa edilmiş mekansal verilerin hızlı ve etkileşimli bir şekilde görselleşmesini sağlayan bir modüldür. Jupyter Notebook veya JupyterLab ortamlarında kullanılabilir.

```
create_map() 
```
Harita oluşturmak için kullanılır.

Parametre olarak:
Görselleştirilecek DataFrame
DataFrame’i temsil edecek isim
Harita için yapılandırma ayarları

create_map(df: SedonaDataFrame=None, name: str='unnamed', config: dict=None)

```
add_df()
```
Harita üzerine yeni DataFrame’ler eklemek için kullanılır. Birden fazla mekansal veri bu sayede aynı harita üzerinde görselleştirilebilir.

Parametre olarak:
Eklemenin yapılacağı harita
Eklenecek DataFrame
DataFrame’i temsil edecek isim

add_df(map, df: SedonaDataFrame, name: str='unnamed')

Not: Şu an için birden fazla veriyi aynı haritada görselleştirmek SedonaPyDeck içerisinde mümkün değildir ancak PyDeck kütüphanesi kullanılarak farklı layerlar ile yapılabilir.

Örnek: 

Elimizdeki çokgen verilerindeki hem çokgenleri hem çokgenlerin merkezlerini çizdirelim.

 ```
cokgenler_df = (
	sedona.read.format("csv")
	.option("delimiter", "/n")
	.option("header", "false")
	.load("data/poligonlar.csv")
)

cokgenler_df.createOrReplaceTempView("cokgenler_tablo")

 

geo_merkez = sedona.sql(
	"""SELECT
  	 
   	ST_Centroid (ST_GeomFromWKT(_c0)) AS Merkez,
   	ST_GeomFromWKT(_c0) AS Geometri
  	 

  	FROM cokgenler_tablo
 """
)
geo_merkez.show(truncate=False)
##############################################


from sedona.maps.SedonaKepler import SedonaKepler
map = SedonaKepler.create_map(df=geo_merkez.select("Merkez"), name="A")
SedonaKepler.add_df(map, geo_merkez.select("Geometri"), name="B")

map.save_to_html(file_name="ornek_map1.html")#HTML olarak kaydettik.
```

Çıktımız:

![Apache_Sedona_Kepler_Example_1](./gorseller/SedonaKepler_1.png)

Turuncu renktekiler B ismi ile belirttiğimiz geo_merkez’in Geometri kolonu ve mavi noktalar ise A ismi ile belirttiğimiz geo_merkez’in Merkez kolonudur.



