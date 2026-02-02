# worflow orchestration



# 이슈 정리

Kestra 연결 실패
1. 원인
    1.1 원인
        Kestra가 DB(Postgres)가 “완전히 준비되기 전에” 먼저 접속을 시도했고,
        그 첫 실패로 Kestra 프로세스가 종료
    1.2 원인
        depends_on:
            kestra_postgres:
                condition: service_started   
2. 해결
    2.1 docker compose 수정
      depends_on:
        kestra_postgres:
            condition: service_healthy
      restart: unless-stopped

    2.2 재시작
        docker restart 02_workfloworchestration-kestra-1

Kestra Python 실행 실패
1. 원인
    1.1 원인
        Kestra의 python.Script Task가 UV 패키지 매니저를 사용하여 Python 런타임을 동적으로 설치하려 했고,
        설치된 Python 실행 파일을 /tmp/kestra-wd/tmp/python/ 경로에서 실행하려다 실패함

    1.2 원인
        Windows 11 + Docker Desktop + WSL2 환경에서
        /tmp 디렉토리가 noexec 옵션으로 마운트되어 있어,
        해당 경로에 설치된 Python 바이너리를 실행할 수 없음

        Permission denied (os error 13)
        Could not find or install Python '3.13' path

        또한 Kestra의 기본 설정은 다음과 같음
        packageManager: UV (기본값)
        UV가 Python을 /tmp 하위에 다운로드 후 실행
        이로 인해 Python 런타임은 존재하지만 실행 권한 문제로 실패

2. 해결
    2.1 Python 패키지 매니저 변경
        UV 사용을 중단하고,
        컨테이너에 이미 설치된 Python을 사용하도록 설정 변경
        packageManager: PIP
        containerImage: python:3.13-slim

        UV를 통한 Python 설치 과정 제거
        /tmp 경로에서 실행하지 않음
        컨테이너 내부의 /usr/local/bin/python 사용

    2.2 결과
        Python Script Task 정상 실행
        Docker Hub API 호출 및 결과 출력 성공
        Windows 환경에서도 동일 Flow 재현 가능