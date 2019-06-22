# KOSPI Predictor

## Docker 실행 방법
cd docker
docker build -t scrapper:imx-1 .
./run_container.sh
(실행)
../pybuild.sh
 - Docker 컨테이너 모두 삭제
 docker rm $(docker ps -a -q)
 - Docker 이미지 모두 삭제
 docker rmi $(docker images -q)
 - 특정 이미지만 삭제
 (image 리스트 확인)
 docker images
 (특정 이미지를 삭제)
 docker rmi 이미지id

## 개요
KOSPI 가격을 예측하는 프로그램.


## 전체 구성도
 ### 데이터 수집
 #### 목표
  - 금융 정보의 데이터를 크롤링해서 특정 기간의 데이터(시가, 종가, 거래량)를  시/일 단위로 DB로 저장
 #### 상세 요구 사항
  - 금융정보는 네이버 금융(시간별 데이터), 야후or구글(일봉)을 활용
  - pandas-reader 사용방법 참고(yahoo 일봉 data): https://wikidocs.net/5753
  - naver data 수집부, yahoo data 수집부, Database 재가공/managing를 각각 클래스로 구현
  - naver web crawling reference html 얻기: finance.naver.com접속 -> 종목코드 입력 -> 차트 아래의 '종합정보' 옆의 '시세 차트 클릭 -> '시간별 시세' 표를 우클릭 후, 프레임소스 보기 클릭
  - 읽어올 기간에 대해서는 클래스 사용자가 정할 수 있도록 구현 (data 학습 및 예측부에서 유연하게 변경이 가능하도록)
  - data 학습 및 예측부, 예상 가격 출력부 에서 사용할 데이터를 Database로 저장
  - 종목코드는 초기 기능구현 시에는 고정으로 한 종목만을 사용하고, 추후 확장해나갈 예정
  - 차트의 고점에는 매도를 저점에는 매수를 할 수 있도록 라벨링 기능(매도, 매수, 관망)
    웹에서 얻어온 Data에서 기간동안 고점, 저점, 상향, 하향을 판단,
    DB를 재가공(매도, 매수, 관망의 라벨링)
  - db browser util: https://sqlitebrowser.org/
  
 ### 주식 가격 훈련, 예측
 #### 목표
  - 시가, 종가, 거래량을 토대로 차트를 학습해서 다음 날의 '시' 단위로 가격을 예측
  - 이번 한 '주' 단위까지 예측해서 예상 차트 데이터 생성
  - 예측된 주식 가격값을 토대로 1주 뒤까지의 시간마다의 가격을 DB로 저장 (예상차트출력부에서 활용할 수 있도록)
 #### 상세 요구사항
  - linear regression을 사용해서 가격을 예측, 예측한 데이터를 Numpy array로 저장
  - linear regression을 통해 학습한 데이터를 통해 매수/매도/관망 등의 분류를 할 수 있도록 다른 모델(RNN or CNN ?)의 입력으로 사용
  - 예측데이터(Numpy array), 매수/매도/관망 을 DB로 생성(차트 출력부에서 확용할 수 있도록)
  - 분기별 재무정보(영업이익, 현금 등)를 활용하는 방법 검토 필요(가중치에 상수값으로 ?)
  
  
 ### 차트 예측 viewer
 #### 목표
 이번 한 주의 예상 차트를 시/일 단위로 출력
 #### 상세 요구 사항
 - 이번 한 주의 예상 차트를 시/일 단위로 출력
 - GUI 프레임워크는 맥과 윈도우에서 모두 사용이 가능한 Qt를 이용
 - 실제 현재 주식그래프를 비교 데이터로 출력(추후 서버 연동 시)
 - 서버 연동을 할 경우, 웹에서 출력
  

 ### 챗봇
 - 슬랙 API를 활용해서 종목에 대한 예측정보(시간별 예상 가격)를 텍스트로 답장
