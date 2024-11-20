### Zeus R-BIZ Challenge

## 2023 로봇사업화 아이디어 경진대회 _ 제 9회 R-BIZ Challenge 

## 프로젝트 배경

- 문제 : 아파트 단지 내 지상으로 차량이 이동하게 되면 안전사고가 날 우려해서 긴급 차량을 제외한 모든 차량의 지상 출입을 금지
    - 이에 대한 대안으로 지하를 통한 배송이 제시되었으나 택배 차량 높이 제한으로 출입이 어려운 설정
- 실시간 영상 기반 물류 정렬 적재, 이송ㆍ하차 시스템을 제안

## 실행 계획 및 단계

![image01](https://github.com/user-attachments/assets/5e01b774-33ae-49a3-a3ae-7d85da639e19)

전체적인 실행 계획의 공간은 그림 3과 같이 구성

1. 고정된 Depth Camera를 통해 공간 A내의 물체들의 중심 좌표를 측정
2. 측정된 좌표를 기반으로 그리퍼를 위치하여 End Effect단에 부착된  Depth Camera로 물체의 높이와 중심을 측정
3. 측정된 값을 통해 물체를 파지하고 AMR에 정렬 적재

## 목표

- 카메라 인식을 통해 물체의 좌표와 수를 측정하고 AMR에 정렬·적재
- 이후 목적 장소에 물체를 하차하는 일련의 동작을 구현하는 것이 최종 목표

## 내용

### 실행 과정

1. 진입가능 구역까지 진입 후 정해진 구역에 택배물 하차 [사람]
2. 택배 하차장으로 이동 [자율이동로봇 - AMR]
3. 비전과 그리퍼로 택배 상자를 파지 후, 운송장의 배송 장소를 구분해 AMR 위에 정렬 적재하는 ‘픽업’ 수행 [제우스 로봇]
4. 동, 호수 별 배송장소로 이동 [자율이동로봇 - AMR]
5. 배송장소에 ‘하차’ 후 택배 ‘배송 완료 문자 전송’ [제우스 로봇]

> 자율이동로봇 - AMR은 본 대회에서 안전을 위해 고정된 상태로 진행



### 알고리즘

1. [Top View Camera]
    - 택배 상자 배치 시, 그리퍼 파지를 고려해 박스 사이에 공간 필요
    - 카메라 검출영역을 벗어난 택배 상자 유무 확인
    - 택배 상자의 중심점, 회전각, 기울기, 가로, 세로, 높이 정보 획득
    - 박스의 위치 좌표를 제우스 로봇의 월드 좌표계로 변환
2. [Pick]
    - 제우스 로봇팔이 택배 상단부로 이동
    - ‘Top View Camera’의 정보로, 각 택배 상자 위치로 이동
    - 엔드이펙트(End effector) 단의 depth camera로 marker에서 배송 장소 정보 획득 및 그리퍼와의 거리를 측정해 파지 진행
    - 자율이동로봇 AMR 위 적재 위치로 이동
3. [Stack]
    - marker에서 획득한 배송 장소 별로 상자를 구분해서 그리퍼가 파지 가능하도록 띄어서 정렬 적재 진행
4. [Place]
    - AMR에서 박스 파지 후, 배송 장소 별로 하차 진행
    - 엔드이펙트 단의 카메라로 배송 완료 사진 전송

### 구현 방법

[Depth Camera - Top View]

- 택배 박스 배치 시 카메라 검출 영역 내에 있는지 검증
- YOLO로 택배 박스 객체 인식 (택배 박스 유무 확인, 파지 순서 부여)
- OpenCV로 박스의 중심점, 중심 좌표,  회전각(rz), 가로, 세로 길이 정보 획득
- Realsense depth frame에서 카메라와의 거리 정보 획득 (박스의 높이)
- OpenCV로 획득한 중심 좌표, 제우스 로봇 월드 좌표계로 변환

[Depth Camera - 엔드 이펙트(End effector)]

- Aruco Marker 인식 (운송장 대체, 배송 위치를 파악해 AMR 적재 순서와 하차 위치 결정)
- 제우스 로봇과 연계하면서 박스의 회전각(rx, ry) 파악
- Realsense depth frame에서 박스까지의 거리 정보 획득
(박스를 파지하기 위해 집게 그리퍼가 내려갈 거리 결정)

[Controller & 양방향 Socket 통신 (TCP/IP)]

- 사전에 작성한 프로그램과 전달받은 연산 데이터에 따라 제우스 로봇 동작에 반영
- 오류코드(Exx, Cxx) 발생 시 신속한 대응 필요
- 2대의 depth camera의 측정 값을 controller에 전달
- 제우스 로봇의 현 상황을 파악하여 다음 동작 이전에 depth camera에서 정보를 읽도록 신호 전달
- 집게 그리퍼의 열고 닫는 동작 수행을 위해 포트 신호(I/O 통신) 전달

[제우스 로봇 제로 - 6축 매니퓰레이터(Manipulator)]

- 저장된 위치 좌표에 따라 동작 수행 (동작 중 오류 발생 시 진행이 끝날 때까지 수정 불가)
- 두 지점 사이를 직선에 가깝게 이동하는 pass-through 활용
- 한 점에서 특정 방향으로 평행하게 이동하는 offset 활용
- 집게 그리퍼의 힘 센서로 물체를 적당한 힘으로 파지

## 활용

- 아파트 단지 내 지상, 지하 출입 제한업이 원할하고 효율적인 배송 가능
- 택배 물량 포화 시 택배 기사의 배송 부담 완화
- 24시간 무인 운용 가능

## Notion page
https://www.notion.so/10aa42acb57b803dab36c5a839fd83bb
