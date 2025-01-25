# Temel Kavramlar

## Veri Türleri
Apache sedona mekansel verileri ifade edebilmek için çeşitli geometrik veri türlerini destekler. 
Sedona’nın içerdiği veri türleri:

Point
LineString
Polygon
MultiPoint
MultiLineString
MultiPolygon
GeometryCollection

## Veri Yapısı
SpatialRDD, Sedona’nın büyük ölçekli mekansal verilerle çalışmak için tasarladığı bir veri yapısıdır.

## Mekansal Verileri İşlemek

Sedona’da SpatialRDD oluşturmak için iki yol mevcuttur. Hem bir veri kaynağından yükleyerek hem de mevcut bir RDD ‘den SpatialRDD oluşturulabilir.

### SpatialRDD’leri Veri Kaynağından Yüklemek

Not: Dosyanın yapısına dikkat edilmelidir, eğer dosya içindeki veri uygun yapıda değilse uygun hale getirilmelidir. Örneğin bir CSV dosyasındaki Nokta geometrileri WKT yapısında ise hata verecektir. Bu kısımda alınan hataların çoğu dosyalardaki verilerin Sedona’nın okuyabileceği formata uymamasından kaynaklıdır.
```
point_rdd = PointRDD(
	sc,
	"data/points.csv",	#Dosya Yolu
	1,  			# X koordinat sütunu
	FileDataSplitter.TSV,  #Verilerin Ayrıştırılacağı Format
	False,  		#Başlıklar dahil edilecek mi  
	10			#RDD kaç parçaya bölünecek
```

sc ile SparkContext,	
"data/points.csv" ile dosya yolu,
1 ile koordinatların başlangıç sütunu (burada bu parametreye 1 verilmesi 2.kolondan itibaren verilerin x-y kordinatları içerdiğini belirtir)  , 
FileDataSplitter.TSV verilerin hangi formatta ayrıştırılacağı,
False başlık satırının dahil edilip edilmeyeceği, 
10 RDD’nin kaç parçaya bölüneceğini ifade eder.

Not: Başlangıç koordinat kolonu sadece PointRDD için kullanılır.  

Diğer geometrilerden örnekler:
 ```
polygon_rdd = PolygonRDD(
	sc,
	"data/polygons.csv",    #Dosya Yolu
	FileDataSplitter.CSV,   #Verilerin Ayrıştırılacağı Format
	False,   		#Başlıklar dahil edilecek mi  	
	11			#RDD kaç parçaya bölünecek
)

```
```
linestring_rdd = LineStringRDD(
	sc, "data/primaryroads-linestring.csv", FileDataSplitter.CSV, True
) 
```
##	Genel SpatialRDD Oluşturmak

PointRDD, PolygonRDD, LineStringRDD gibi özel SpatialRDD’ler haricinde içerisinde farklı türlerde geometriler içeren dosyalar için genel SpatialRDD oluşturulmalıdır.

###  WKT/WBT ‘den yüklemek

`wkt_rdd= WktReader.readToGeometryRDD(sc,"data/points_wkt.WKT", 0, True, False) `

veya .csv / .tsv uzantısıyla:
```
wkt_rdd= WktReader.readToGeometryRDD(
sc,
"data/points_csv.csv",  #Dosya Yolu	
0, 			  #Geometri başlangıç sütunu
True, 			  #Sütunların ayırıcı ile ayrılıp ayrılmadığı	
False			  #Geometrilere ek olarak diğer sütunların da taşınıp taşınmacağı	

)
```

sc ile SparkContext,	
"data/points_wkt.WKT" ile dosya yolu,
0 ile koordinat başlangıç sütunu,
True ile sütunların bir ayrıcı ile ayrıldığını 
Örneğin virgül ile ayrılmış bir wkt yapısı: (1,POINT (30 10),ExampleData 
    				        2,POINT (40 40),AnotherData)
False ile geometri verisinin yanında diğer verilerin de SpatialRDD’ye dahil edilip edilmeyeceği

belirtildi. 

###  Shapefile ‘dan yüklemek

`shp_rdd = ShapefileReader.readToGeometryRDD(sc, shape_file_yolu)`

ile ShapeFİle dosyalarından SpatialRDD oluşturulabilir. Burada farklı ve önemli olan dosya yolu için doğrudan .shp uzantılı dosyanın değil bu dosyanın bulunduğu klasörün dosya yolunu belirtmektir. .shp, .shx, .dbf gibi diğer ek dosyalar da bu klasörde olmalıdır.

###  GeoJSON’dan yüklemek

`
geojson_rdd = GeoJsonReader.readToGeometryRDD(sc, geo_json_file_location)
`
ile GeoJson dosyalarından SpatialRDD oluşturulabilir.

Not: GeoJSON ‘ı bu şekilde SpatialRDD olarak okunması için Sedona’nın desteğinde geliştirilmesi gereken konular var. (Örneğin GeoJSON Feature yerine FeatureCollection yapısında ise verinin işlenmesinde mevcut sedona sürümünde sorun çıkıyor.)  Orijinal dökümantasyonunda da bu yöntem yerine Sedona SQL ve DataFrame API kullanılarak GeoJSON’ın okunması tavsiye ediliyor.

###  Sedona SQL ile yüklemek 

Veriyi SedonaSQL’e DataFrame olarak yükledikten sonra geometri tipinde bir kolonu bulunan yeni bir DataFrame oluştururuz. Ardından Adapter ile bu Dataframe’i SpatialRDD’ye çevirebiliriz.

#Veriyi yükleyelim
`
df = sedona.read.format("csv").option("header", "false").load("data/points_csv.csv")
`
#Sedona SQL için geçici tablo olşuturalım
`
df.createOrReplaceTempView("inputtable")
`


#WKT yapısında mekansal veriler içeren _c0 kolonundan geometri tipinde bir sütün içeren yeni bir DataFrame oluşturduk
`
spatial_df = sedona.sql("""
	SELECT ST_GeomFromWKT(_c0) AS geometry FROM inputtable
""")
`
Yeni dataframe’imizi kontrol edelim

`
spatial_df.printSchema()
`
root
 |-- geometry: geometry (nullable = true)

Artık elimizde geometri tipinde kolonu bulunan bir DataFrame var. Bunu RDD’ye çevirebiliriz.

Bu Dataframe’i Adapter ile SpatialRDD’ye çevirdik. 
 spatial_rdd = Adapter.toSpatialRdd(spatial_df, "geometry")
