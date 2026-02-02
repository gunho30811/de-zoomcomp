# worflow orchestration



# 이슈 정리
1.Kestra 연결 실패

1.1 원인
    Kestra가 DB(Postgres)가 “완전히 준비되기 전에” 먼저 접속을 시도했고,
    그 첫 실패로 Kestra 프로세스가 종료
1.2 원인
    depends_on:
        kestra_postgres:
            condition: service_started   
1.2 해결
    1.2.1 docker compose 수정
      depends_on:
        kestra_postgres:
            condition: service_healthy
      restart: unless-stopped
      
    1.2.2 재시작
        docker restart 02_workfloworchestration-kestra-1