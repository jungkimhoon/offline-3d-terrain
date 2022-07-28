
# Open DEM 사이트
1. http://data.nsdi.go.kr/dataset/20001
  -  img 형식 DEM파일
  - QGIS를 통해 래스터 병합 저장 img -> tif

2. https://portal.opentopography.org/datasets
  - tif 형식 DEM파일
  - 지형정보 데이터 제약 없이 오픈됨 supported by national science foundation
  - 사이트 가입 필요


# 절차
1. Get DEM.tif (Digital Elevation Model)
2. Cesium-Terrain-Builder로 .tif -> .terrain 변환
3. Cesium-Terrain-Server로 .terrain 서비스



# Cesium-Terrain-Builder
docker run -it --name ctb -v /mnt/d/junghoonkim/data/docker/terrain:/data tumgis/ctb-quantized-mesh

mkdir -p terrain

ctb-tile -f Mesh -C -N -o terrain korea.tif (로컬, 388MB기준 약 20분)
ctb-tile -f Mesh -C -N -l -o terrain korea.tif

# Cesium-Terrain-Server
docker run -p 8000:8000 -v /mnt/d/junghoonkim/data/docker/tilesets/terrain:/data/tilesets/terrain geodata/cesium-terrain-server
