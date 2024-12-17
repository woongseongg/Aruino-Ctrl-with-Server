# **Arduino-Ctrl-with-Server 프로젝트**

## **1. 프로젝트 개요**
이 프로젝트는 **다중 클라이언트 환경**에서 서버가 센서 데이터를 처리하고 장치(Buzzer, LED, Servo Motor)를 제어하는 **IoT 기반 시스템**입니다.  
클라이언트는 **Arduino**를 사용하여 센서 데이터를 서버로 전송하며, 서버는 수신된 데이터를 분석하고 조건에 따라 장치를 제어합니다.



## **2. 주요 기능**
1. **다중 클라이언트 지원**  
   - `pthread`를 사용하여 여러 클라이언트(Arduino)가 동시에 서버에 연결할 수 있습니다.  

2. **센서 데이터 수신 및 처리**  
   - **CDS 센서**, **초음파 센서**, **RFID 센서** 데이터를 수신합니다.  
   - 수신된 데이터를 기준으로 **임계값**을 검사하고 적절한 제어 명령을 전송합니다.  

3. **장치 제어**  
   - 서버는 조건에 따라 **Buzzer**, **LED**, **Servo Motor** 제어 명령을 클라이언트에 송신합니다.  

4. **에러 로그 기록**  
   - 센서 값이 임계 범위에 도달하면 **로그 파일**에 날짜와 함께 기록됩니다.  
   - 로그 파일은 `error/` 디렉터리에 저장됩니다.



## **3. 사용된 기술**

### **서버**
- **C 언어**: 소켓 통신, 다중 클라이언트 처리, 에러 로깅 구현  
- **POSIX 스레드(pthread)**: 다중 클라이언트 동시 처리를 위해 사용  
- **소켓 프로그래밍**: TCP 소켓을 사용하여 서버-클라이언트 간 통신  
- **파일 I/O**: 에러 로그 파일 저장  

### **클라이언트 (Arduino)**
- **Arduino IDE**: 클라이언트 코드를 작성하고 업로드  
- **센서 제어**: CDS 센서, 초음파 센서, RFID 센서 데이터 읽기  
- **시리얼 통신**: 서버와의 데이터 송수신  



## **4. 프로젝트 파일 구조**
Arduino-Ctrl-with-Server/ 
│ 
├── Makefile                # 서버 빌드 및 실행 자동화 
├── src/ 
│    └── server.c           # 서버 프로그램 메인 소스 코드 
│
├── arduino/
│     ├── client_CDS.ino    # Arduino CDS 센서 클라이언트 코드 
│     ├── client_US.ino     # Arduino 초음파 센서 클라이언트 코드 
│     └── client_RFID.ino   # Arduino RFID 센서 클라이언트 코드 
│
├── error/
│     ├── error_US.txt      # 초음파 센서 에러 로그 파일 (실행 시 생성됨) 
│     └── error_CDS.txt     # CDS 센서 에러 로그 파일 (실행 시 생성됨) 
│
└── README.md               # 프로젝트 설명 파일



## **5. 프로세스 흐름**

### **서버 프로세스**
1. **서버 초기화**  
   - 서버 소켓 생성 → `bind()` → `listen()`을 통해 클라이언트의 연결 대기  
   - 에러 로그 파일을 생성하거나 열기  

2. **클라이언트 연결 처리**  
   - 클라이언트가 연결되면 **accept()**를 통해 소켓을 할당합니다.  
   - 각 클라이언트는 **pthread**를 통해 별도의 스레드에서 처리됩니다.  

3. **데이터 수신 및 분석**  
   - 클라이언트가 전송한 데이터를 읽고, 데이터를 파싱하여 분석합니다.  
   - **센서 값과 플래그**에 따라 조건 검사 (임계값 비교).  

4. **장치 제어 명령 송신**  
   - 조건에 맞는 제어 명령을 클라이언트로 송신 (e.g., `BUZ_ON`, `LED_ON`).  

5. **에러 로그 기록**  
   - 임계값을 초과하는 데이터는 로그 파일에 기록됩니다.  
   - 기록 형식: `에러 번호 시간 센서 값`.  



### **클라이언트 프로세스 (Arduino)**
1. **센서 데이터 측정**  
   - 각 클라이언트(Arduino)는 **CDS**, **초음파**, **RFID** 센서 데이터를 측정합니다.  

2. **서버로 데이터 전송**  
   - 측정된 센서 데이터를 시리얼 통신을 통해 서버에 전송합니다.  

3. **서버 응답 수신**  
   - 서버로부터 제어 명령(`BUZ_ON`, `LED_ON`, `DETECTED`)을 수신합니다.  

4. **장치 제어**  
   - 서버가 보낸 명령에 따라 **Buzzer**, **LED**, **Servo Motor**를 제어합니다.



## **6. 실행 방법**

### **서버 실행**
#### 1) 프로젝트 빌드: 실행 파일 생성
  `make`

#### 2) 서버 실행
  `make run`
  `./svr`

#### 3) 서버 정리(빌드 파일 삭제)
  `make clean`

#### 4) 클라이언트 실행(Arduino)

#### 5) 정상 작동 및 생성된 로그 파일 확인
