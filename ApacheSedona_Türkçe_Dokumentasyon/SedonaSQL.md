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

+---------------------------------------------------------------------------------------------------------+
|cokgen                                                                                                   |
+---------------------------------------------------------------------------------------------------------+
|POLYGON ((-74.0428197 40.6867969, -74.0421975 40.6921336, -74.050802 40.6912794, -74.0428197 40.6867969))|
+---------------------------------------------------------------------------------------------------------+

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

root
 |-- Cizgiler: geometry (nullable = true)

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

+------+
|mesafe|
+------+
|   5.0|
+------+


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
+----+
|alan|
+----+
|16.0|
+----+

