
# 절차
1. Get DEM.tif file (Cesium에 사용될 quantized-mesh terrain 데이터를 만들기 위해 해당 파일이 필요하다.)
2. Cesium-Terrain-Builder로 .tif -> .terrain 변환
3. Cesium-Terrain-Server로 .terrain 서비스
4. Cesium에서 사용

# Open DEM(Digital Elevation Model) 사이트
실제 테스트해본 사이트만 기술함.

1. 국가공간정보포털 (http://data.nsdi.go.kr/dataset/20001)
  - img 형식 DEM파일 (90M)  
  - .img -> .tif 파일형식 변환 필요
  - 사이트 가입 필요

2. OpenTopography (https://portal.opentopography.org/datasets)
  - tif 형식 DEM파일
  - 지형정보 데이터 제약 없이 사용가능 (일부 high resolution 데이터 등 제약있음)
  - 사이트 가입 필요

# 래스터파일 형식 변환 방법
국가공간정보포털과 같은 경우 DEM파일을 img형식으로 제공한다.  
Cesium-Terrain-Builder의 제약조건에 따라 .tif 파일로 컨버팅해야한다.
1. QGIS의 새 프로젝트에 래스터파일 드래그 앤 드랍  
2. 래스터 > 변환 > 변환 (포맷변경)  
3. 좌표계 - 기본 좌표계 설정후 .tif 형식 파일로 저장  

# Cesium-Terrain-Builder
DEM.tif 데이터를 Cesium-Terrain-Builder를 통해 Cesium에 사용할 Terrain 파일로 변환한다.
```console
// 변환을 원하는 tif 경로에 볼륨을 위치시킨다.
$ docker run -it --name ctb -v /mnt/d/junghoonkim/data/docker/terrain:/data tumgis/ctb-quantized-mesh

$ mkdir -p terrain

$ ctb-tile -f Mesh -C -N -o terrain korea.tif (로컬, 388MB기준 약 20분)
$ ctb-tile -f Mesh -C -N -l -o terrain korea.tif
```


# Cesium-Terrain-Server
 변환된 terrain 파일을 적절한 경로에 위치시키고 볼륨을 연결하여 run한다.
``` console
$ docker run -p 11800:8000 -v \ /home/inno/3d_map_offline_volume/data/korea/tilesets/terrain:/data/tilesets/terrain geodata/cesium-terrain-server
```
동작이 정상적으로 됐다면  
http://127.0.0.1:11800/ --- was open 확인가능  
http://127.0.0.1:11800/tilesets/tiles/9/873/362.terrain?v=1.1.0  --- terrain 확인가능


# Cesium에서 사용
아래처럼 Cesium-Terrain-Server에 terrain tile을 요청하여 동작한다.
``` javascript
scene.terrainProvider = new Cesium.CesiumTerrainProvider({           
    url : `http://127.0.0.1:11800/tilesets/tiles`,
});
```
# 결과
![image](https://user-images.githubusercontent.com/59942147/182511367-31e9ec13-661d-4bd2-8ae1-d79543a48671.png)


# 참고
## DEM.tif
DEM.tif 파일은 GeoTIFF 형식으로 지형 및 고도 정보의 메타데이터 등 아래와 같은 정보로 저장된다.
- horizontal and vertical datums 
- spatial extent, i.e. the area that the dataset covers 
- the coordinate reference system (CRS) used to store the data
- spatial resolution, measured in the number of independent pixel values per unit length
- the number of layers in the .tif file
- ellipsoids and geoids - estimated models of the Earth’s shape
- mathematical rules for map projection to transform data for a three-dimensional space into a two-dimensional display

## Cesium-Terrain-Builder
https://github.com/tum-gis/cesium-terrain-builder-docker  
https://hub.docker.com/r/tumgis/ctb-quantized-mesh  


## Cesium-Terrain-Server 참고  
https://github.com/geo-data/cesium-terrain-server  
https://hub.docker.com/r/geodata/cesium-terrain-server
