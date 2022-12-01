# **STOCKER "가치투자를 기반한 비지도학습 포트폴리오 구성"**

🏆 2022 UBION Project

![Untitled](https://user-images.githubusercontent.com/88031549/205055658-286ea206-1f43-4979-babb-0c4d73f14bac.jpeg)

![Untitled 1](https://user-images.githubusercontent.com/88031549/205056311-73bfbb0d-aadf-4760-b1d8-5d7fb6d4b4a5.jpeg)

![Untitled 2](https://user-images.githubusercontent.com/88031549/205056394-b5e4df7d-428e-4274-bae6-4c3c57f5b985.jpeg)

![Untitled 3](https://user-images.githubusercontent.com/88031549/205056428-0120c124-357f-4308-bdcd-46245d93858b.jpeg)

![Untitled 4](https://user-images.githubusercontent.com/88031549/205056477-adfddfbd-c29e-42bf-b522-730b7c8c4069.jpeg)

![Untitled 5](https://user-images.githubusercontent.com/88031549/205056509-3f95be7f-7c1e-4675-9a14-66149a9d2994.jpeg)

# **프로젝트 소개**

본 프로젝트는 KOSPI&KOSDAQ 기업을 대상으로 총 10가지의 피쳐(펀더멘탈 지표, 위험 대비 성과 지표, 팩터 지표)를 수집하여, 비지도학습(Agglomerative Clustering)을 수행한 뒤 투자유니버스와 기업을 선정해 여러 성격의 투자 포트폴리오를 제공한다. 이를 통해 개개인의 잘못된 투자성향이 반영될 수 있는 초개인화의 한계점을 보완하고 고객이 선호하는 특정 거래소 상장 주식, 섹터 등을 반영하여 수익성을 높일 수 있는 포트폴리오 제공이 가능하다.

[ 배경 ]

현재 많은 금융상품들이 넘쳐나고 있고, 일반 투자자들이 많은 선택지 속에서 자신이 바라는 종목을 찾는 과정은 귀찮고 복잡하다. 이러한 상황에 맞게 최근 개인화를 고도화한 초개인화 서비스들이 대거 등장하고 있는데 그 중 “다이렉트 인덱싱”은 자산운용업계의 떠오르는 화두이다. 그러나 이는 지나친 맞춤형 상품이다보니 개개인의 잘못된 투자성향이 반영될 가능성이 있어 수익률에 한계를 줄 수 있다.

 그렇기에 본 프로젝트는 초개인화의 한계점을 보완하여 개인의 선호를 반영할 수 있는 매력적인 상품을 개발하고자 월가의 영웅들의 가치투자 지표들을 활용한 비지도학습을 통해 다양한 성격의 포트폴리오를 구성하고자한다.

---

# **구조**

## 0**. 데이터 수집**

- 출처 : KRX & TS200
- 수집기간 : 20.04.01 ~ 21.03.31
- 대상종목 : KOSPI & KOSDAQ(최종 746개)
- 제거 종목 : 관리 / 거래정지 / 우선주/ 금융업 / 보험업 종목
- 선정 피쳐
    1. 펀더멘탈 지표
        1. NCAV
        2. PEGR
        3. PSR
        4. 유동비율
        5. 부채비율
    2. 위험 대비 성과 지표
        1. 샤프지수
        2. 일별수익률 평균
    3. 팩터 지표
        1. PBR
        2. 시가총액

## 1. Preprocessing

- 결측치&Inf 제거
- 이상치 제거 : 이상치 존재 시 Scaling, Clustering에 영향을 미치기에 제거가 필요하지만,

                   데이터 하나하나가 잠재적 투자기업이에 Boxplot에서 직관적으로 보이는 기업을 제거

## 2. PCA

- Standard Scaling
- 누적 설명력 80%정도의 주성분 갯수만큼 차원축소 진행 → 5차원 축소

## 3. Clustering

- Clustering Method(Agglomerative Clustering)
    - hyperparameter로 ward연결을 선택하여 크기가 비교적 비슷한 클러를 만듦으로서 기업수가 적어 투자 후보에서 제외되는 군집이 없도록 방지한다.
- Cluster 갯수 선정
    - K-Ellbow Method : 데이터와 군집 중심간의 거리 & 연산 시간을 고려해 클러스터 수 k 선정
    - Silhouette Score : 개체가 다른 클러스에 비해 자신의 클러스터와 얼마나 유사한 지 측정
    - 나쁜 특성을 가진 군집을 선별해내야하기에 silhouette score를 참고하여 클러스터 수를 11개로 선정

## 4. 투자유니버스 선정

- 클러스터 별 종목 갯수 확인
- WICS 업중 중분류에 따라 클러스터 업종특성 확인
- 2,3차원 scatter
- 클러스터 별 각 피쳐의 중간값 확인
- 최종 선정 군집(특징)
    - 0군집(NCAV/유동비율)
    - 3군집(유동비율/샤프지수)
    - 10군집(KOSPI/변동성/PER)

## 5. 포트폴리오 구성

- 각 클러스터 마다 투자를 진행할 10개 종목을 선별하기 위해 수익률에 높은 영향을 미치는 피쳐 2개 선정
    - 각 피쳐에 rank를 매기고 선택된 3개 컬럼의 rank을 합산하여 total rank 생성
    - total rank 기준 상위 10개의 기업들의 수익률을 평균내어 컬럼 3개 조합마다의 수익률을 구함
    - 평균 수익률 기준 상위 10개 컬럼조합에서 빈도수를 기준으로 메인 컬럼 2개를 선정(”일일변동평균-샤프지수”)
- 메인 컬럼 각각에 rank를 매기고 합산하여 total rank 기준 상위 10개 기업을 선별
- 클러스터 3개에 위 프로세스를 진행하여 총 3개의 포트폴리오 구성

## 6. 성과측정

- 백테스팅 기간 : 21.04.01 ~ 22.03.31
- 벤치마크 대비 성과(수익률, MDD, SHAPR, SORTINO)

![Untitled](https://user-images.githubusercontent.com/88031549/205056688-65bfeddd-4ccb-4a5b-b9b9-becb4593ef15.png)

(*아래 ppt예시)

![Untitled 6](https://user-images.githubusercontent.com/88031549/205056568-4346b3a2-d21d-4b55-8652-294c06ce48e1.jpeg)

![Untitled 7](https://user-images.githubusercontent.com/88031549/205056626-2a7ec4a1-5e6a-48f7-a63d-282c469802c5.jpeg)

![Untitled 8](https://user-images.githubusercontent.com/88031549/205056661-c7141e95-f81d-487e-a2c0-25b3de9a12d1.jpeg)