# line_follower_by_opencv
Making line following robot (turtlebot-wifi) by using openCV and Tensorflow

●	환경 구성
○	터틀봇
운영체제는 라즈비안. ROS 와 파이썬 2, tensorflow 1.0, openCV 가 설치되어있다.
○	마스터 pc
ROS master 가 설치되어 있다. 이 마스터를 통해 터틀봇과 통신한다. 
○	외부 접속 pc
ssh 통신을 통해서 터틀봇에 접속한다. 수동조종할 때 주로 쓰는 방법이다. 이외에 학습코드가 있어 데이터를 학습시킬 수 있다.


●	프로젝트 구동법
○	먼저 마스터 놋북에서 roscore 를 돌린다.
터미널에서 roscore 를 실행한다.
○	터틀봇 pc 에서 roslaunch 를 돌린다.(HDMI 꽂아서 직접해도 되고, 아무 노트북이나 ssh 연결해서 원격 명령내려도 됨)
roslaunch turtlebot3_bringup turtlebot3_robot.launch 실행
○	터틀봇 pc 에서 collector.py 를 실행한다.(마찬가지로 ssh 원격 명령해도 됨.)그러면 자율주행을 시작하는데, w a d 키를 누르면 수동조종모드로 돌입하여 수동조종 할 수 있다. 단, s 를 눌러줘야 자동모드로 돌아온다. q 를 누르면 주행이 종료되고 저장할 파일이 있다면 파일을 저장한 후 프로그램이 완전히 종료된다.
	
●	소스코드 분석
○	collector.py 프로그램 설명서
이 파일은 걍 총괄 프로그램임.
○	기본적으로 필요한 모듈 다 import 하는데 tensorflow 땜에 import 하는데 좀 오래걸린다.
○	그 다음 초음파 센서 조작하는데 필요한 Pin 셋팅
○	KBHit 클래스 : 터미널 창에서 키보드 입력을 감지하는 클래스
○	kbhit 메서드 : 키보드를 누르면 그 값을 반환, 아무것도 누르지 않으면 False 반환
○	ultrasonic 메서드 : 초음파 센서를 1 회 구동해서 거리 값을 반환
○	getkey 메서드 : 키보드 값 반환하는 메서드인데, 이 메서드는 이제 사용하지 않고(deprecated) 대신 kbhit 으로 대동단결
○	Collector 클래스 : 데이터 수집 / 주행 클래스
○	move_forward 메서드 : 직진 명령을 publish 해주고 [0,1,0] 커맨드를 반환
○	turn_right 메서드 : 우회전 명령을 publish 해주고 [0,0,1] 커맨드를 반환
○	turn_left 메서드 : 좌회전 명령을 publish 해주고 [1,0,0] 커맨드를 반환
○	stop_turtlebot 메서드 : 정지 명령을 publish
○	control 메서드 : getkey() 메서드를 이용해 터틀봇 조종
○	control_use_model 메서드 : CNN 을 통해 나온 결과로 조종
○	collect 메서드 : 데이터 수집하는 메서드. 순수하게 인간이 조종해줌. 이제 필요없음(어느정도 주행을 하므로)
○	drive 메서드 : 자율주행 메서드. main loop 에서 이 메서드만 돌아간다. 주행기능과 동시에 인간이 개입하여 조종하고 데이터 추가기능도 모두 구현됨.
○	make_data : Collector 객체가 가지고 있는 이미지와 그에 대응되는 라벨 데이터를 h5py 형식의 파일로 저장하는 메서드
○	add_data : 기존의 파일에 데이터를 추가하는 메서드. 여기서 안쓰이고 이 기능은 학습을 수행하는 노트북에서 일괄적으로 처리
○	read_data : 파일이 잘 저장되었나 확인하려는 디버그 메서드. 안쓰임.
○	main 함수: Collector 객체 생성 이제 collect 메서드는 사용할 일이 없고, drive 메서드만 돌린다. 주행이 끝내면 데이터를 만들고 프로그램이 종료된다.

테크니컬 리포트

●	개요
○	환경
하드웨어는 터틀봇 와플파이(라즈베리파이3B + 라이다 센서 + 파이카메라 + 서보모터 2), 운영체제는 라즈비안을 활용하였다. 학습환경은 빈 교실에 검은색 테이프를 붙여 트랙을 구성하였고 테스트환경도 동일하다.

○	주행
이 프로젝트의 기본기능인 주행은 하드웨어로 제공된 라즈베리파이 카메라만을 활용한다. 활용한 소프트웨어로는 python2, openCV 파이썬 라이브러리, tensorflow 기계학습 라이브러리가 있다. openCV 는 영상획득과 전처리 역할을 하고, tensorflow 는 영상을 학습시키고 터틀봇 조종에 필요한 결론을 도출하는 역할을 한다.

○	장애물 회피
이 프로젝트의 핵심 요구사항인 회피 및 우회로 탐색 기능은 하드웨어로 제공된 라이다 센서만을 활용한다. ROS 에서 제공되는 라이다 관련 라이브러리를 활용해 알고리즘을 작성함

●	기능 명세
○	주행 부문
카메라만을 이용해 검은 선을 따라갈 수 있다. 현재 직진, 좌회전, 우회전, 후진 기능 있음

○	우회로 탐색 부문
경로에 장애물이 있을 때 장애물의 크기에 관계없이 회피한 후 다시 라인에 복귀할 수 있다. 장애물은 4 각형 모양으로 한정하고 있다.

○	기타
빛의 세기, 라인의 굵기 등 다양한 새로운 환경에 대응할 수 있게 새로이 학습하는 모듈 포함하고 있다. 비상정지 기능은 초음파 센서로 구현했으나 라이다와 작동범위가 겹쳐 일단 제외

●	주행
○	모델 개념
지도학습 기반의 Imitation Learning 모델이다.
○	프로세스
영상 획득 - 영상 전처리 - CNN 모델에 입력 - 출력으로 나온 커맨드 시행

○	영상획득
파이썬 2 환경에서 openCV 와 파이 카메라를 이용하여 영상을 획득한다. 해상도는 480 x 640 이고 30 fps 속도로 받아들인다. 기존의 카메라의 각도가 너무 높은 탓에 하단의 라인을 잘 보지 못했기 때문에 카메라를 약간 기울여 고정했다.

○	영상 전처리
획득된 영상을 openCV 의 전처리 기능을 이용해 전처리한다. 이미지 처리에서 주로 하는 처리과정인 gray scale 화 하였다. 색깔 정보는 이미지 처리에 방해가 되기때문에 이미지는 1 채널로 변환했다. 또한 영상에서 노이즈를 제거하기 위해 blurring 작업을 거쳤다. 라즈베리파이의 컴퓨팅 파워를 고려해 가우시안 blurring 이 아닌 median 필터를 적용했다. 

○	CNN 모델에 입력
tensorflow 로 제작한 CNN 모델에 전처리된 영상데이터를 주입한다. layer 는 2 층으로 설계했는데 라즈베리파이의 연산력의 제한때문에 더 늘리지는 못했다. 출력으로는 직진([0100]), 좌회전([1000]), 우회전([0010]), 라인없음([0001) 의 4 개의 클래스에 대한 확률값이 나온다. 

○	커맨드 시행
CNN 모델에서 나온 출력에서 가장 큰 확률값을 가지는 클래스를 액션으로 선택한다. 직진, 좌회전, 우회전의 경우에는 해당하는 커맨드를 ROS node 를 통해 터틀봇에 보낸다. ‘라인없음’ 결과가 나오면 잠시 시야를 잃었다고 판단하여 터틀봇에 후진 명령을 내린다.
○	


●	우회
○	개요
일반적인 상황의 범위를 잡기 어려워 어느정도 제한된 상황에서의 rule based 알고리즘을 설계했다. 라이다 센서가 핵심이 되는데 전방을 기준으로 반시계방향으로 해상도 1 도의 360 도 정보를 획득할 수 있다. 라이다 값이 1 회 갱신되는데 걸리는 시간은 약 0.23 초가 걸리고 그 값은 파이썬 환경에서 360 개의 tuple 형태로 받을 수 있다.

○	라이다 센서 구동
라이다 센서의 값을 받으려면 0.23 초 가량이 소요된다. 확인해본 결과 이 시간만큼 프로그램이 정지된다는 것을 발견하여 주행시 큰 문제가 될 수 있다고 판단했다. 멀티스레딩을 이용해 별도의 서브스레드에서 라이다 센서값을 받아내게 했다. 이로써 주행에 사용되는 메인스레드와 별도로 라이다 값을 받아 주행에 지연시간을 없애는 방식으로 구동된다.


○	프로세스
전방 라이다 확인 - 물체감지 시 우회모드로 전환후 직각 회전 - 측면 라이다 확인하며 직진 - 물체감지 시 또 회전하여 ‘ㄷ’ 자 형태의 경로를 따라간 후 주행 모드로 전환한다.

○	주행모드 → 우회모드
주행을 하는 도중 전방에 물체가 감지되면 우회모드로 전환된다. 0 도 기준 +- 10 도 전방의 라이다 센서값을 읽어서 그 중 최소값이 특정치(35 cm) 미만이면 앞에 장애물이 있다고 판단한다. 터틀봇을 직각으로 꺾은 후 우회경로가 기존 경로에 크게 벗어나지 않도록 자신의 위치를 조금 조절하는 과정을 거친다.

○	우회 경로
직각으로 꺾으면 측면이 장애물을 마주하는데, 이 때부터는 측면 라이다 값을 이용해 우회로를 따라간다. 90 도 기준 +- 5 도 측면에 해당하는 라이다 값 중 최소값이 특정치 미만이면 계속 장애물이 옆에 있다 판단하여 쭉 가고 그렇지 않으면 장애물의 영역을 벗어났다고 판단하여 회전하고 이 과정을 ‘ㄷ’ 자 형태의 우회로를 따라갈 때까지 반복한다.

○	우회모드 → 주행모드
우회경로를 모두 따라갔으면 주행모드로 전환하여 다시 라인을 따라 주행을 하게 한다.

●	학습
○	개요
주행을 위해 최초 학습 그리고 데이터를 추가하여 추가 학습을 시키는 모듈이다. 이는 환경이 바뀌어도 처음부터 학습을 실시할 수 있게 모듈로 분리했다. 학습데이터는 터틀봇을 직접 키보드로 조종하며 이미지와 라벨을 함께 저장해 생산한다.

○	프로세스
데이터 수집 - 학습 - 데이터 보강 - 학습

○	데이터 수집
로컬 컴퓨터를 이용해 터틀봇을 직접 제어하며 카메라의 영상데이터와 해당 라벨데이터를 함께 저장한다. 개인 노트북에서 ssh 연결로 라즈베리파이에 접속하여 키보드로 조종할 수 있다. 저장된 데이터는 ‘h5py’ 모듈을 이용해 터틀봇 라즈베리파이에 저장한다.

○	학습
위 과정을 통해 저장된 학습용 데이터를 로컬에 가져와 openCV 를 이용해 데이터를 프레임 단위로 확인할 수 있다. 만약 학습하기에 좋지 않은 데이터가 있다면 편집할 수 있다. 최종적으로 편집한 데이터를 CNN 모델에 주입해 학습을 한다. 자유롭게 셋팅을 하여 여러번 학습을 할 수 있다. 학습결과를 그래프 등으로 시각화하여 평가를 할 수 있는 기능도 있다. 학습이 완료되면 CNN 의 weight 파일이 만들어지는데, 그 파일을 다시 터틀봇에 전달하여 주행을 시킬 수 있다.

○	데이터 보강
처음 학습을 시킨 결과로 주행을 하면 성능이 안 좋을 수도 있다. 충분한 학습 데이터가 확보되지 않았기 때문이다. 그럴 땐 주행을 하며 잘못된 행동을 할 때에만 수동으로 조종을 하고 그 때의 데이터를 저장한다. 

○	재학습
추가로 수집된 학습 데이터를 노트북으로 가져와 학습을 시킬 수 있는데 기존의 데이터와 합칠 수도 있고 별도로 학습시킬 수도 있다.

●	프로그램 구조
○	collector.py
메인 스크립트 파일. 터틀봇 조종, 자율주행, 데이터셋 저장 등의 기능이 포함. 터미널에서 이 스크립트만 실행시키면 구동된다.
○	agent.py
CNN 모델만을 저장하는 파일. 모델 수정역시 여기서 한다.
