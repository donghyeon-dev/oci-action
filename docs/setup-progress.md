# OCI A1 GitHub Actions Setup Progress

이 파일은 GitHub Actions가 주기적으로 OCI A1 인스턴스 생성을 시도하기까지 필요한 작업을 추적하는 체크리스트입니다.

민감 정보는 절대 이 파일에 적지 마세요. OCID, private key, Telegram token, chat ID는 GitHub Secrets 또는 개인 보관용 메모에만 저장합니다.

## 1. OCI API Key 준비

- [ ] OCI Console 접속
- [ ] 우측 상단 프로필 -> My profile 이동
- [ ] Resources -> API keys -> Add API key
- [ ] Generate API key pair 선택
- [ ] private key `.pem` 파일 다운로드
- [ ] API key 추가 완료
- [ ] Configuration File Preview에서 아래 값 확보
- [ ] `user` OCID 확보
- [ ] `fingerprint` 확보
- [ ] `tenancy` OCID 확보
- [ ] `region` 확인, 예: `ap-chuncheon-1`
- [ ] `.pem` 파일 전체 내용을 개인 보관용 메모에 복사

## 2. OCI 리소스 값 준비

- [ ] 사용할 compartment OCID 확인
- [ ] root compartment를 쓸 경우 tenancy OCID를 `OCI_COMPARTMENT_ID`로 사용하기로 결정
- [ ] VCN/Subnet 준비 또는 기존 subnet 확인
- [ ] `OCI_SUBNET_ID` 확보
- [ ] Ubuntu 22.04 ARM 이미지 OCID 확인
- [ ] `OCI_IMAGE_ID` 확보
- [ ] Availability Domain 확인
- [ ] `OCI_AVAILABILITY_DOMAIN` 확보, 예: `nHas:AP-CHUNCHEON-1-AD-1`
- [ ] SSH public key 준비
- [ ] `OCI_SSH_PUBLIC_KEY` 확보

## 3. Telegram 알림 준비

- [ ] BotFather에서 새 Telegram bot 생성
- [ ] `TELEGRAM_BOT_TOKEN` 확보
- [ ] 봇에게 메시지 한 번 보내기
- [ ] chat ID 확인
- [ ] `TELEGRAM_CHAT_ID` 확보

## 4. GitHub Repository Secrets 등록

Repository -> Settings -> Secrets and variables -> Actions -> New repository secret에서 등록합니다.

- [ ] `OCI_CLI_USER`
- [ ] `OCI_CLI_TENANCY`
- [ ] `OCI_CLI_FINGERPRINT`
- [ ] `OCI_CLI_KEY_CONTENT`
- [ ] `OCI_CLI_REGION`
- [ ] `OCI_COMPARTMENT_ID`
- [ ] `OCI_SUBNET_ID`
- [ ] `OCI_IMAGE_ID`
- [ ] `OCI_AVAILABILITY_DOMAIN`
- [ ] `OCI_SSH_PUBLIC_KEY`
- [ ] `TELEGRAM_BOT_TOKEN`
- [ ] `TELEGRAM_CHAT_ID`

자세한 설명은 `docs/secrets.md`를 참고합니다.

## 5. 첫 수동 실행

- [ ] GitHub repository Actions 탭 이동
- [ ] `Launch OCI A1 Instance` workflow 선택
- [ ] Run workflow로 수동 실행
- [ ] `Check existing A1 instance` step 성공 확인
- [ ] `Try launch A1 instance` step 실행 확인
- [ ] 결과가 capacity 부족인지, 성공인지, 인증 오류인지 확인
- [ ] Telegram 알림 동작 확인

첫 실행 결과:

```text
아직 실행 전
```

## 6. 주기 실행 확인

- [ ] 첫 수동 실행이 인증 문제 없이 끝남
- [ ] workflow가 disabled 상태가 아닌지 확인
- [ ] 15분 후 schedule 실행이 자동으로 생성되는지 확인
- [ ] 실패가 capacity 부족 계열이면 그대로 유지
- [ ] `NotAuthenticated`, `InvalidPrivateKey`, `NotFound` 계열이면 Secret 또는 OCI 리소스 값 수정

## 7. 성공 후 마무리

- [ ] Telegram 성공 알림 수신
- [ ] OCI Console에서 `dh-server-a1` RUNNING 확인
- [ ] Public IP 확인
- [ ] workflow가 자동 disabled 되었는지 확인
- [ ] 자동 disabled가 안 되었으면 Actions 탭에서 수동 disable
- [ ] SSH 접속 테스트

```powershell
ssh -i $HOME\.ssh\oci_a1 ubuntu@<public-ip>
```

## Troubleshooting Log

문제가 생기면 날짜, GitHub Actions 실행 URL, 에러 요약만 적습니다. Secret 값은 적지 않습니다.

```text
2026-04-29: setup-progress.md 생성
```

