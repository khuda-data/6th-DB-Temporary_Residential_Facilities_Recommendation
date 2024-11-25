# 🏃‍♀️재난 상황 대비 추가 임시주거시설 추천🏃‍♀️‍➡️

## 프로젝트 소개
### 우리는 재난으로부터 안전한가?
- 임시주거시설 중 숙박 시설의 비율이 2.9%에 불과하다
- 지역 규모, 인구 등 여건을 바탕으로지정 가능한 시설을 모색한 경우 없음
- 전체 임시주거시설 중 내진 설계가 적용된 지진겸용 임시주거시설은 약 35.7%에 불과 


## 데이터 수집
| 데이터명                 | 출처                          | 사용목적          | 컬럼명                       |
|:--------:|:---------------:|:-------------:|:--------------------------|
| 건축물대장 (표제부)      | 건축데이터 민간개방 시스템       | 임시주거시설 지정 | - 대지위치<br> - 주용도 코드명(숙박시설)<br> - 내진 설계 적용여부<br> - 면적<br> - 사용승인일(30년 이하) |
| 병원 및 편의시설 위치     | 지방행정 인허가 데이터개방       | 적합성 점수에 반영 | - 개방서비스명<br> - 위도<br> - 경도                                           |
| 지진 정보               | 기상청 날씨누리                | 적합성 점수에 반영 | - 규모<br> - 위도<br> - 경도                                              |
| 산불 정보               | 재난안전데이터 공유 플랫폼        | 적합성 점수에 반영 | - 산불정보아이디<br> - 산불위치 X, Y 좌표                                   |

## 데이터 전처리
|프레임워크 및 알고리즘| 사용목적| 얻게 된 변수|
|:-----:|:------------:|:------------:|
|geocoding|대지위치를 위·경도로 변환 | 고도 데이터|
|QGIS | 위·경도를 고도로 변환 | 고도 데이터|
|OSMnx | 교통상황을 고려하여 거리 계산 | 가까운 병원과의 거리|
|유클리드(haversine)| 직선거리를 구함| 의원수, 편의점수, 산불횟수|
|GEOJSON| 각 시설 위·경도를 통해 소속 행정동을 구함| 숙박시설이 속하는 행정동코드|

## 입력 데이터
1. 고도
2. 수용 인원
3. 700m 반경 의원 수
4. 300m 반경 편의점 수
5. 가장 가까운 병원과의 거리
6. 1km 반경 산불 횟수
7. 건물 연식
8. earthquake_count

## 모델링
### PCA 분석
최적의 주성분 개수를 파악하여 보니 7개였으며, 7가지 주성분으로 데이터의 95%까지 설명이 가능하다. 

PCA 후 데이터 분포의 문제점:
- 데이터의 밀도가 일정하지 않음
- 데이터의 분포가 구형이지 않음
- 데이터 속 이상치가 존재

### HDBSCAN 
HDBSCAN은 밀도 기반 군집화와 계층적 군집화를 결합한 알고리즘이며, 밀도가 높은 데이터 그룹을 찾고 밀도 기반으로 계층적 군집화를 수행하여 안정적인 클러스터를 자동으로 선택한다. 

HDBSCAN을 사용한 클러스터링의 문제점:
- 파라미터를 조정해도 비슷한 결과가 도출됨
- 현재 가지고 있는 데이터 특성 상, 이상치가 많을 수 밖에 없음
→ 데이터를 다른 방식으로 변환할 필요가 있음!

### UMAP을 활용한 차원 축소
UMAP은 고차원 공간의 데이터를 그래프 형태로 표현한 뒤, 그래프를 저차원 공간으로 투영하여 데이터의 구조를 보존하는 방식이다. 이는 국소적 및 전역적 구조를 모두 잘 보존하도록 설계되었다. 

- minPts: 데이터 차원의 두 배 값으로 설정
- min_cluster_size: k-거리 그래프 활용

## 특성 중요도 해석


### 분산 분석 ANOVA



## 적합성 지표 설정
사용한 입력 데이터 7개(1~7)에 대응되는 7개의 가중치를 부여하여 적합성 지표를 정의한다. 여기서 사용된 가중치는 랜덤으로 정해진 값이나 정규분포를 보인다. 

계산한 점수를 바탕으로 각 클러스터별 상위 20% 데이터를 추출한다. 

## p-median
주어진 여러 지점에 대해 p개의 시설을 배치하여 총 이동 비율을 최소화하는 방법을 찾는 문제이기에, 따라 주어진 여러 행정동에 대해 p개의 임시주거시설을 배치하여 총 이동 비용을 최소화한다. 

|    |내용|
|--------|---------------------|
|입력변수| ①인구밀도 중심(동 별 위치)과 ②숙박시설 위치|
|목적함수| 거리/인구밀도|
→ 인구밀도가 높은 행정동과 거리는 최대한 가까운 숙소 선정


## 결과: 임시주거시설 추천


## 결론 및 한계점
### 의의
- 인구 밀도를 고려하여 지진 겸용 임시주거시설의 위치를 선정함
- 대구광역시를 중심으로, 임시주거시설 중 숙박시설이 부족하다는 기존의 문제 해결에 기여함
- 군집화가 어려운 형태의 데이터 분포를 HDBSCAN과 UMAP을 이용하여 군집화에 성공함

### 한계점
- 인구 밀도가 높은 지역 위주로 선정하여, 인구 밀도가 낮은 지역은 소외될 가능성이 존재
- 민간 숙박시설 대상으로 선정하여, 경제성 등 여러 요인들로 인해 실제 지정되기에 어려움이 예상됨
- 가중치 설정에 있어서 전문가의 의견을 수렴한 여러 선행 연구와 달리 임의로 설정한 값을 사용함

## 참고문헌
- 백선경, 조시은, 오민정, 박유나. (2023). 재난 대응을 위한 임시주거시설 관리체계 개선방안. 건축공간연구원.
- Schubert, Erich & Sander, Jörg & Ester, Martin & Kriegel, Hans & Xu, Xiaowei. (2017). DBSCAN revisited, revisited: Why and how you should (still) use DBSCAN. ACM Transactions on Database Systems.


