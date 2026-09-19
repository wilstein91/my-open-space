# 인수인계

## 지금 상태
- `index.html`, `scene.sog`, `README.md`, `shots/viewer.png` 모두 이 폴더에 있고 로컬 git 저장소로 커밋까지 끝남 (origin: `https://github.com/wilstein91/my-open-space.git`, 브랜치 `main`)
- **push는 아직 안 했음** — 본인이 나중에 직접 `git push -u origin main` 실행하기로 함 (이 환경에 GitHub 인증 정보가 없어서 대신 못 했음)
- README는 "폰에서 첫 화면까지" 한 항목만 빈칸. Pages 켠 뒤 실제 주소로 로딩 시간 재서 채우면 끝

## 찍은 대상
`My Room.mp4` — 취미방 겸 드레스룸. 원래 교안은 "밖에서 고른 공개 가능한 공간"을 요구했지만(자기 방을 올리면 링크 보낼 때 망설여진다는 게 취지), 본인이 "공개돼도 불이익 없는 공간"이라 판단해 방으로 진행하기로 결정함. README "어디를 골랐나"에 그 근거를 적어 둠. 채점 기준(1단계: 고른 근거가 README에 있는가)에서 이 판단이 받아들여질지는 미지수 — 걸리면 밖에서 새로 찍어 2단계부터 다시 돌리면 됨 (아래 재현 방법 참고).

## 파이프라인 파라미터 (재현/재실행용)
- 작업 폴더: `/mnt/c/aiffel_work/gsplat_project/work/` (이 폴더는 git에 안 잡힘, 제출 폴더 밖)
- 파이썬 venv: `work/.venv` (uv로 만듦, python 3.11 + torch 2.14+cu130 + map-anything editable install)
- 영상 24장 추출: `work/frames/`
- 복원: `work/reconstruct.py` — 모델 `facebook/map-anything-apache`, `amp_dtype="bf16"` (RTX 3070은 compute capability 8.6이라 bf16 가능, T4라면 fp16으로 바꿔야 함)
- 점 → 가우시안 변환: `work/make_splat.py <K> <출력.ply> <입력.npz>` — 지금 쓴 K=1.4, 반지름 0.5~99.5 백분위 밖 점은 버림, y/z 부호 반전 적용됨
- 점 개수 줄이기(선택): `work/downsample.py <목표개수>` — 렌더링이 버벅일 때만 씀. 지금 제출본은 원본 그대로(330만 개)이고, 안 쓴 이유는 브라우저 하드웨어 가속이 꺼져 있던 게 진짜 원인이었기 때문 (`chrome://settings/system`에서 그래픽 가속 켜서 해결)
- 형식 변환: `npx -y @playcanvas/splat-transform@latest -w <in> <out>` — `.ply`/`.compressed.ply`/`.sog`/`.spz` 넷 다 알갱이 3,295,198개로 동일함을 확인함

## 측정값 (README에 이미 반영됨)
- 카메라 퍼짐 / 장면 대각선 = 0.34 (기준선 0.1)
- .ply 224,073,881B(68.00B/알갱이) · .compressed.ply 53,650,613B(16.28B) · .sog 15,116,871B(4.59B) · .spz 20,830,149B(6.32B)

## index.html 변경점
원본 room-3d-demo 템플릿에 드래그앤드롭을 추가함. `loadFromBuffer(buf, displayName)` 함수 하나로 첫 로딩(fetch)과 드롭 로딩을 합쳤고, 확장자→Spark 파일타입 대응표(`TYPE_OF`)는 그대로 재사용. 파일을 새로 드롭하면 기존 mesh를 지우고(`scene.remove` + `dispose`) 새로 얹은 뒤 다시 `fit()` 호출함.

## 다음에 할 일
1. `git push -u origin main`
2. GitHub 저장소 Settings → Pages → Source를 main 브랜치 루트로 설정
3. `https://wilstein91.github.io/my-open-space/scene.sog` 를 새 탭에서 열어 파일이 바로 내려오는지 확인 (안 되면 Pages가 아직 안 켜졌거나 캐시 문제)
4. 본인 폰 / 다른 사람 폰에서 로딩 시간 재서 README "폰에서 첫 화면까지" 채우기
