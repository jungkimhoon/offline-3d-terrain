
# 절차
1. Get DEM.tif file (Mapbox에 사용될 terrain-rgb 파일을 만들기 위해 해당 파일이 필요하다.)
2. Build Dockerfile & Run 
3. Inspect & Resetting DEM.tif
4. Convert DEM.tif to rgb data (RGBify)
5. Slice the file
6. Terrain Service

# Open DEM(Digital Elevation Model) 사이트
Cesium Terrain 참고자료 참고
# 래스터파일 형식 변환 방법
Cesium Terrain 참고자료 참고

# Build Dockerfile & Run
DEM.tif 데이터를 Cesium-Terrain-Builder를 통해 Cesium에 사용할 Terrain 파일로 변환한다.
```console
// Dockerfile 경로에서 빌드
// TODO: 도커파일 업로드 예정
$ docker build . -t rio

// 빌드 후 run
$ docker run --rm -it -v /mnt/d/junghoonkim/workspace/mapbox:/opt/dem rio bash
$ cd /opt/dem
```


# Inspect & Resetting DEM.tif
아래 명령어로 tif를 inspect 할 수 있다.
``` console
$ rio info --indent 2 DEM.tif
```

inspect 결과
``` json
{
  "bounds": [
    -64424.0,
    36953.0,
    656746.0,
    1181753.0
  ],
  "colorinterp": [
    "gray"
  ],
  "count": 1,
  "crs": "PROJCS[\"Transverse Mercator\",GEOGCS[\"GRS80 ELLIPSOID\",DATUM[\"GRS80 ELLIPSOID\",SPHEROID[\"GRS 1980\",6378137,298.257222101004],TOWGS84[0,0,0,0,0,0,0]],PRIMEM[\"Greenwich\",0],UNIT[\"degree\",0.0174532925199433,AUTHORITY[\"EPSG\",\"9122\"]]],PROJECTION[\"Transverse_Mercator\"],PARAMETER[\"latitude_of_origin\",38.0000000000001],PARAMETER[\"central_meridian\",127],PARAMETER[\"scale_factor\",1],PARAMETER[\"false_easting\",200000],PARAMETER[\"false_northing\",600000],UNIT[\"metre\",1,AUTHORITY[\"EPSG\",\"9001\"]],AXIS[\"Easting\",EAST],AXIS[\"Northing\",NORTH]]",
  "descriptions": [
    "Layer_1"
  ],
  "driver": "GTiff",
  "dtype": "float32",
  "height": 12720,
  "indexes": [
    1
  ],
  "interleave": "band",
  "lnglat": [
    128.09598592026137,
    38.079152696603614
  ],
  "mask_flags": [
    [
      "nodata"
    ]
  ],
  "nodata": -9999.0,
  "res": [
    90.0,
    90.0
  ],
  "shape": [
    12720,
    8013
  ],
  "tiled": false,
  "transform": [
    90.0,
    0.0,
    -64424.0,
    0.0,
    -90.0,
    1181753.0,
    0.0,
    0.0,
    1.0
  ],
  "units": [
    null
  ],
  "width": 8013
}
```

inspect 데이터 중에 "crs", "nodata"를 확인해야한다.  
- crs: epsg:3857 변환 권장 (변환안해도 구동 가능했다..)  
- nodata: 값이 음수일 경우 수정필요 (Terrain RGB가 음수를 표현 못함)

아래 gdalwarp 명령어를 통해 Reprojection, nodata를 수정한다.

``` console
  $ gdalwarp \
    -t_srs EPSG:3857 \
    -dstnodata None \
    -novshiftgrid \
    -co TILED=YES \
    -co COMPRESS=DEFLATE \
    -co BIGTIFF=IF_NEEDED \
    -r lanczos \
    DEM.tif \
    DEM_RESETTING.tif \
```

위 명령어 수행 후 inspect 결과

``` json
{
  "blockxsize": 256,
  "blockysize": 256,
  "bounds": [
    13775464.221044756,
    3872718.6514187697,
    14762115.108592,
    5348392.225641946
  ],
  "colorinterp": [
    "gray"
  ],
  "compress": "deflate",
  "count": 1,
  "crs": "EPSG:3857",
  "descriptions": [
    "Layer_1"
  ],
  "driver": "GTiff",
  "dtype": "float32",
  "height": 12861,
  "indexes": [
    1
  ],
  "interleave": "band",
  "lnglat": [
    128.1787184179301,
    38.22002976386787
  ],
  "mask_flags": [
    [
      "all_valid"
    ]
  ],
  "nodata": null,
  "res": [
    114.7401892716878,
    114.7401892716878
  ],
  "shape": [
    12861,
    8599
  ],
  "tiled": true,
  "transform": [
    114.7401892716878,
    0.0,
    13775464.221044756,
    0.0,
    -114.7401892716878,
    5348392.225641946,
    0.0,
    0.0,
    1.0
  ],
  "units": [
    null
  ],
  "width": 8599
}
```
crs, nodata 부분이 수정됨.

# RGBify
grayscale의 DEM.tif 파일을 RGB data파일로 변환한다.
``` console
height = -10000 + ((R * 256 * 256 + G * 256 + B) * 0.1)
```
위 공식을 참조하여 아래처럼 변환을 진행
```
rio rgbify -b -10000 -i 0.1 DEM_RESETTING.tif DEM_RGB.tif
```

결과로 높이 정보가 포함된 RGB tif파일로 변환된다.  


# Slice the file
얻어진 DEM_RGB.tif 파일을 mapbox의 terrain으로 사용하기 위해 xyz타일로 변환하여 사용한다. (QGIS 이용)

1. DEM_RGB 파일 QGIS 새 프로젝트에 드래그 앤 드랍  
2. 공간 처리 툴박스 > 래스터 도구 > XYZ 타일 생성 (디렉터리)  
3. XYZ 타일 생성 팝업에서 다음과 같이 설정  
4. 실행



# Terrain Service
얻어진 xyz타일은 서버를 만들어서 서비스 테스트를 진행함. (따로 Server를 지원해주는 자원은 없어보인다.)
## Server
``` java
  @GetMapping("/terrain/{z}/{x}/{y}")
	public ResponseEntity<Object> getMapboxTerrain(@PathVariable("z") String z,
												   @PathVariable("x") String x,
												   @PathVariable("y") String y) throws IOException {		
		String path = Paths.get(filePath, String.format("%s/%s/%s.png",z,x,y)).toAbsolutePath().toString();

		try {
			Path filePath = Paths.get(path);
			Resource resource = new InputStreamResource(Files.newInputStream(filePath));

			File file = new File(path);

			HttpHeaders headers = new HttpHeaders();
			headers.setContentType(MediaType.IMAGE_PNG);
      headers.setContentDisposition(ContentDisposition.builder("attachment")
                                                      .filename(file.getName()).build()));

			return new ResponseEntity<Object>(resource, headers, HttpStatus.OK);
		} catch(Exception e) {
			return new ResponseEntity<Object>(null, HttpStatus.CONFLICT);
		}
	}
```

## Client
``` javascript
this.map = new mapboxgl.Map({
    container: this.$refs['map'],
    center: center,
    zoom: zoom,
    pitch: pitch,
    bearing: bearing,
    antialias: true,            
    attributionControl: false, 
    style: {
        version: 8,
        sources: {
            'dem': {
                'type': 'raster-dem',
                'tiles': ['http://localhost:9999/api/terrain/{z}/{x}/{y}'],
                'tileSize': 256,
                'maxzoom': 15                        
                
            },
            'osm-tiles': {
                'type': 'raster',
                'tiles': [`${OFFLINE_OSM_TILE}/tile/{z}/{x}/{y}.png`],
                'tileSize': 256,                    
            }
        },
        layers: [{
            id: 'simple-tiles',
            type: 'raster',
            source: 'osm-tiles',
            minzoom: 0,
            maxzoom: 22
        }],   
    },                    
});
```
