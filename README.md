# 전국 행정구역별 지도 시각화 (Folium, 일부-경기도/인천)

-   Python **지도 시각화** Library folium을 사용해 행정구역별 시각화를 진행해보겠습니다.
-   필요 Site : [SGIS](https://sgis.kostat.go.kr/view/index)(통계지리정보서비스), [QGIS](https://qgis.org/ko/site/), [지도 셰이퍼](https://mapshaper.org/)

 기존의 folium을 이용한 행정구역별 시각화 자료는 굉장히 많지만 다들 전국 단위이거나 서울특별시만을 기준으로 하고 있어 일부 지역만 표시하고 싶은 니즈를 충족시키기 위해 작성하게 되었습니다. 꽤나 번거롭고 여러 가지 절차를 거쳐야 하지만 한 번 방법을 익혀둔다면 두고두고 도움되실 거라 믿어 의심치 않습니다 👊

---

## folium 설치

 우선 간단하게 pip install을 이용하여 folium을 설치해줍니다.

```
!pip install folium
```

---

## Data 획득

### 1\. 경기도, 인천의 행정구역별(이번에는 읍면동을 기준) 시각화를 진행할 예정이니, 행정구역별 경계 위경도가 필요합니다.

정말 감사하게도 [South Korea](https://github.com/southkorea) Github 링크에서 서울특별시와 전국에 대한 행정구역별 경계 좌표를 구할 수 있으나 가장 최신 자료가 2018년이고 일부 자료는 따로 추출하기 어렵다는 아쉬움이 남습니다. 😿

### 2\. 우리는 직접 데이터를 요청해볼 것입니다. [SGIS](https://sgis.kostat.go.kr/view/index)(통계지리정보서비스) 사이트에 접속해 자료제공 > 자료 신청 탭에서 원하는 자료를 신청합니다.

1.  그중 우리는 센서스용 행정구역경계(읍면동) 데이터를 신청할 것입니다. (당일 처리)
2.  신청자료 다운로드 탭을 들어가 다운로드를 진행하면 dbf, prj, shp, shx 총 4개의 파일이 다운로드됩니다.

<img width="800" alt="image" src="https://user-images.githubusercontent.com/54921730/160831666-a0555aa7-d853-47d2-b402-5f87d4566fea.png">

### 3\. 우리는 그중 shp 파일을 사용할 것인데, 그것을 위해 [QGIS](https://qgis.org/ko/site/)라는 프로그램이 필요합니다.

1.  자유 오픈 소스 지리정보 시스템으로 지리 데이터를 다루면 꼭 만나게 되는 프로그램이니 친해져 봅시다. 🐳

2.  **왜 이번 작업에서 우리는 QGIS를 다뤄야 할까요?** **SGIS에서 제공해주는 데이터는 좌표계(CRS)는 EPSG:5179 - Korea 2000 기준인데, folium은 기본 좌표계인 EPSG:4326에 맞춰서 제공하기 때문입니다.**

3.  다운이 모두 완료되면, 레이어 > 레이어 추가 > 벡터 레이어 추가에서 shp 파일 경로를 입력 후 인코딩으로는 EUC-KR을 선택해 추가해줍니다.

    <img width="600" alt="image" src="https://user-images.githubusercontent.com/54921730/160831773-ec1ec1f1-47b6-4747-a5a9-1099ce6648c7.png">

4.  이제 우리가 생각하는 지도의 형상을 띄고 있습니다. 레이어를 선택 후 Export > 객체를 다른 이름으로 저장을 선택해줍니다.

<img width="600" alt="image" src="https://user-images.githubusercontent.com/54921730/160831944-5abbc51c-3451-46b7-b392-43cccd6cb07a.png">

5. 마지막으로 위에서 언급했듯이 좌표계를 기본 좌표계: EPSG:4326 - WGS 84로 변경하여 저장해줍니다. 거의 다 됐어요!

<img width="600" alt="image" src="https://user-images.githubusercontent.com/54921730/160832196-79db4fa8-756e-44fe-ad6b-0f5e8822f5c5.png">

### 4\. 이제 EPSG:4326 좌표계를 가진 shp 파일을 python에서 사용하기 위해 json형식으로 변경해줍니다.

1.  이 과정에서 [지도 셰이퍼](https://mapshaper.org/)라는 사이트를 이용해줍니다. shp 파일을 업로드해준 뒤 Export > GeoJSON > Export를 하면 이제 데이터는 준비가 완료되었습니다.

<img width="800" alt="image" src="https://user-images.githubusercontent.com/54921730/160832284-e3bb6301-86cb-4252-ab0f-49d9c2d7c0ec.png">

---

## Code 실행 후 결과 도출

1. Folium 외에도 Github에 올라가 있는 데이터를 추출한 뒤 json 파일을 다루기 위해 requests와 json을 import 해줍니다.

   ```
   import folium
   import requests
   import json
   ```

2. 각각의 위경도 데이터를 추출해줍니다.

   ```
   kkd_content = requests.get('https://raw.githubusercontent.com/herjh0405/map_data/main/kkd_geo.json').content
   kkd_geo = json.loads(kkd_content)
   
   ic_content = requests.get('https://raw.githubusercontent.com/herjh0405/map_data/main/ic_geo.json').content
   ic_geo = json.loads(ic_content)
   ```

3. 마지막으로 folium을 호출한 뒤 GeoJson을 사용하여 맵에 정보를 추가해주면 저희가 원하는 결과가 도출되는 것을 확인할 수 있습니다.!! 🌟🌟🌟

   ```
   m = folium.Map(
       location=[37.559819, 126.963895],
       zoom_start=11, 
   )
   
   folium.GeoJson(
       kkd_geo,
       name='경기도 읍면동'
   ).add_to(m)
   
   folium.GeoJson(
       ic_geo,
       name='인천광역시 읍면동'
   ).add_to(m)
   
   m
   ```

<img width="800" alt="image" src="https://user-images.githubusercontent.com/54921730/160832409-4a704524-388c-4e8d-b1de-47a571976f87.png">

이제 지도 시각화를 진행했으니 Marker와 Tooltip을 이용해 더 많은 정보를 기록하며 잘 활용하시길 바라겠습니다. 감사합니다 :)